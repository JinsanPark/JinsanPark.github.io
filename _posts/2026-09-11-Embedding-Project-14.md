---
title: 동시 요청에 API를 1번만
date: 2026-09-11 15:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---

## 지난 시간

[지난시간](/posts/Embedding-Project-13/)에 캐시 두 단에서 전부 미스가 나면, API를 두번 호출하는 거 해결한다고 했었죠.<br>
그러면 한번 해결 해 봅시다.

## 나도 똥싸개 빨리 좀

요청이 세가지가 들어왔습니다.<br>
하나는 소변이고, 두개는 큰거네요.

근데 화장실에 소변칸 하나, 변기칸 하나가 있다고 해봅시다.<br>
저희 코드는 줄을 안서요. 그냥 들어가요. <br>
되게 비효율적이죠?

그렇다면, 볼일끼리 서로 줄을 다르게 세우면 될거 아니에요.<br>
큰거는 이쪽줄, 작은거는 저쪽줄로 해서.

이렇게 서로 락을 쪼개는게 바로 키별 락이에요.

## 락을 어디에 보관하나?

`Map<String, Object> locks;`
HashMap을 쓸 수도 있는데, 저는 못씁니다.<br>
일단 두 스레드 동시에 put 요청이 들어오면 망가질 수 있기 때문 입니다.<br>
ConcurrentHashMap을 쓰면 됩니다.

`Object lock = locks.computeIfAbsent(norm, k -> new Object());`
은 없으면 만들고, 있든 없든 그 값을 돌려주는 문법인데, 이걸 ConcurrentHashMap이 통째로 처리해줘요.<br>
즉, 두 스레드가 동시에 불려도, 둘 다 같은 객체를 받습니다. 하나는 만들어서, 하나는 남이 만든 걸요.

## 어디서부터 어디까지 잠궈요?

락으로 감쌀 구간은 L1 미스 이후 ~ API 호출 ~ 캐시에 넣기

`스레드 A: L1 미스 → 락 잡음 → API 185ms → L1에 put, DB에 save → 락 놓음`
`스레드 B: L1 미스 → 락 대기 ............................. → 블록 진입`

인데, 스레드 B는 A가 끝날 때 까지 대기합니다. 끝까지 대기할 필요가 있을까요? L1 꺼내쓰면 안됨? 같은 검색어인데 말이죠.

맞긴한데, 대기는 해야합니다.<br>
A끝나기 전에는 L1에 값이 없거든요.

double-checked locking이란걸 할건데, 두번 체크 합니다.

락밖 : 이미 있으면 락을 아예 안 잡고 빠져나가도록
```java
            normToVector = lruCached.get(norm);
            if (normToVector != null) {
                path = "L1_hit";
                return normToVector;
            }
```

지난 시간에 이미 했던거죠.

락안 : 기다리는 동안 남이 채워놨는지 보려고 기존 코드를 synchronized (lock) {}으로 감싸줄겁니다.

```java
synchronized (lock) {
                normToVector = lruCached.get(norm);
                if (normToVector != null) {
                    path = "L1_hit_by_lock_wait";
                    return normToVector;
                }

                // 기존 코드...
                Optional<QueryVectorCache> cached = queryVectorCacheRepository.findByNormalizedQueryAndModel(norm, voyageModel);
                if (cached.isPresent()) {
                    path = "DB_hit";
                
```

왜 normToVector = lruCached.get(norm) 을 한번 또 해요? 라고 질문 하실 수 있는데, 다른 곳이 끝났으면, 대기 중이던게, L1에 값이 있나 확인하고 그거만 꺼내 가려고 그런거에요.

<details markdown="1">
<summary><b>코오드 (클릭하여 펼치기)</b></summary>

```java

private final ConcurrentHashMap<String, Object> locks = new ConcurrentHashMap<>();

public float[] getVector(String query) {

        long startGetVector = System.nanoTime();
        double ms;
        String norm = normalizeQuery(query);
        float[] normToVector;
        String path = null;
        double apiTime = 0;
        double saveTime = 0;

        try {
            normToVector = lruCached.get(norm);
            if (normToVector != null) {
                path = "L1_hit";
                return normToVector;
            }

            Object lock = locks.computeIfAbsent(norm, k -> new Object());

            synchronized (lock) {
                normToVector = lruCached.get(norm);
                if (normToVector != null) {
                    path = "L1_hit_by_lock_wait";
                    return normToVector;
                }

                Optional<QueryVectorCache> cached = queryVectorCacheRepository.findByNormalizedQueryAndModel(norm, voyageModel);
                if (cached.isPresent()) {
                    path = "DB_hit";
                    QueryVectorCache cache = cached.get();
                    lruCached.put(norm, cache.getEmbedding());
                    return cache.getEmbedding();
                } else {
                    path = "API_miss";
                    long apiStart = System.nanoTime();
                    normToVector = embeddingClient.embedQuery(norm);
                    apiTime = (System.nanoTime() - apiStart) / 1_000_000.0;
                    QueryVectorCache cache = new QueryVectorCache(norm, voyageModel, normToVector, LocalDateTime.now());
                    lruCached.put(norm, cache.getEmbedding());
                    long saveStart = System.nanoTime();
                    try {
                        queryVectorCacheRepository.save(cache);
                    } catch (DataIntegrityViolationException e) {
                        path = "API_miss_dup";
                    }
                    saveTime = (System.nanoTime() - saveStart) / 1_000_000.0;
                }
            }

        } finally {
            ms = (System.nanoTime() - startGetVector) / 1_000_000.0;
            log.debug("path={} term={} save_ms={} api_ms={} total_ms={}", path, norm, saveTime, apiTime, ms);
        }

        return normToVector;

    }
```
</details>

이렇게 했고요.

## 잘 됨

```text
2026-09-11T14:34:24.704+09:00 DEBUG 10376 --- [newsFinder] [nio-8080-exec-8] o.j.n.queryVector.QueryVectorService     : path=L1_hit_by_lock_wait term=안녕못해요 save_ms=0.0 api_ms=0.0 total_ms=221.0197
2026-09-11T14:34:24.704+09:00 DEBUG 10376 --- [newsFinder] [nio-8080-exec-4] o.j.n.queryVector.QueryVectorService     : path=API_miss term=안녕못해요 save_ms=33.4589 api_ms=181.4465 total_ms=228.5786
```

잘 되는거 같죠.<br>
저번 시간이랑 비교해 볼까요?

```text
//이번 시간
path=L1_hit_by_lock_wait api_ms=0.0
path=API_miss api_ms=181.4465

//지난 시간
path=API_miss     ... api_ms=188.4605
path=API_miss_dup ... api_ms=181.4975
```
제가 의도한 대로, API 호출이 1번 들어가는거 같네요. `api_ms=0.0`,`api_ms=181.4465` 인걸 보니, API 요청을 1번만 넣은거 같네요.

근데,

`total_ms=228.5786, total_ms=221.0197`

 api호출 안한쪽이 빠르지는 않습니다. 한 요청의 API 호출이 끝나야 돌려주니깐요.

## 근데 문제가 아직 있음

ConcurrentHashMap이 계속 커져요. 그도 그럴게, 검색어가 계속 쌓이는데, 따로 치우는 거는 없죠.

```java
synchronized (lock) {
    ...
}
locks.remove(norm);
```

A가 remove한 직후에 같은 검색어로 C가 들어왔다고 합시다. C의 computeIfAbsent가 뭘 하게 될까요?<br> 그리고 그때 B가 아직 이전 락 안에 있다면 뭐가 문제될까요?

바로, 같은 검색어인데, 줄이 2개가 생겨버립니다.<br>
저희가 한게 같은 검색어는 검색어끼리 하나로 묶는거였는데, 그게 안되는거죠.

그래서 일단 놔두도록 했습니다.<br>
아마 10만 검색어여도 몇 mb정도니깐, 이정도는 아직은 감안 가능하다고 생각됩니다. 재보지는 않았어요.

추후에, 참조 카운트나 WeakReference 개념을 제가 배우고 나서 수정을 해보도록 합시다.

## 다음 시간에는

근데 아깝지 않나요? 한쪽이 L1에 값을 넣을때, 그걸 가져오면, 다른 쪽이 훨씬 빠르게 결과를 볼 수 있을거 같은데 말이죠.

그러면 다음시간에는, 진짜로 대기중인 사람이 더 빠르게 받아볼 수 있도록 해봅시다.