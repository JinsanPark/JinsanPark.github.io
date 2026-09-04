---
title: L1 캐시 측정과 조건 기준점 최대한 세우기
date: 2026-09-04 09:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---

## 지난 시간

지난 시간에 L1 캐시를 만들어 봤습니다.<br>
측정을 한다고 했으니깐, 측정을 해야겠죠.

그런데, 제가 측정을 매번 진행했는데, 그때 마다 매번 기준이 바뀌고, 저도 매번 까먹고해서, 
매번 이게 차이가 좀 크게 나고, 고개가 갸우뚱한 상황이 좀 나왔습니다.<br>
 심지어는 측정을 하고 지워버린 다음, 제 머리에서도 지워져서, 이게 재현이 안되더라고요.

이번에는 최대한 다음 측정에서도 비슷하게 유지를 할 수 있게, 기준점을 세웠습니다.

## 기준은?

저는 달리기의 시간을 재는데, 날마다 이제 시작하는 위치, 중간에 기록 하는 위치, 그리고 결승점이 매 달리기 마다 바뀌고 있어서, 이거를 이제 고정을 하려고 했습니다.<br>
 적어도 어디서 쟀는지, 기록은 남겨둬야할거 같았거든요.

그래서 이번 기준은 getVector로 잡았습니다.

## 어디어디?

우리가 재볼거는 L1 히트, DB히트, DB미스 입니다.<br>
L1 미스는 어딨냐고요? 어차피 미스나면 DB쪽으로 가니깐 필요 없죠.


```java
//L1 히트
if (lruCached.containsKey(norm)) {
    normToVector = lruCached.get(norm);
    return normToVector;
}
```
```java
//DB 히트
if (cached.isPresent()){
    QueryVectorCache cache = cached.get();
    lruCached.put(norm,cache.getEmbedding());
    return cache.getEmbedding();
}
```
```java
//DB 미스
else {
            normToVector = embeddingClient.embedQuery(norm);
            QueryVectorCache cache = new QueryVectorCache(norm, voyageModel, normToVector, LocalDateTime.now());
            lruCached.put(norm,cache.getEmbedding());
            queryVectorCacheRepository.save(cache);
        }
```

이 코드들에 시작과 끝에다가 측정을 하면 좋을거 같더라고요.<br>
또, getVector의 시작과 끝도 좋겠죠.

| 경로 | 필요한 상태 | 만드는 방법 |
|---|---|---|
| L1 히트 | L1에 값이 있어야 함 | 앱 재시작 + 워밍업 한 바퀴 |
| DB 히트 | L1은 비고 DB에는 있어야 함 | 앱 재시작 (DB는 그대로) |
| API 미스 | 둘 다 비어야 함 | 앱 재시작 + DB 캐시 비움 |


앱 재시작은 L1만 비웁니다. DB는 앱이 꺼져도 남아 있으니 따로 비워야 하고요.<br>
L1 캐시 사이즈 100으로 증설했고,<br>
spring.jpa.show-sql=false로 바꿔서 로그랑 안섞이게 만들었습니다.



try - finally도 사용해 줬고요.<br>
왜냐면, 마지막에 값을 출력하고 나가야 하는데, 중간에 return있어서 끝나버리니깐요.<br>
매리턴 전마다 출력해도 되긴하는데, 너무 귀찮잖아요.

측정은 순차적으로 200개 단어를 AI에게 입력하라고 시켰고, L1만 크기가 100이라, 1부터 200까지 돌리면 1 ~ 100까지 가득 찼다가 101 ~ 200까지 다시 기존거 밀어내고 다시 채워버리니깐, 101 ~ 200번 까지 단어 검색을 웜업으로 1번, 측정으로 2번 반복시켰습니다.

측정 돌리다 보니깐, 저번에 DB에 저장하는 시간이 좀 의아하게 나와서, 그 부분도 다시 추가해서 재봤습니다.

```java
    public float[] getVector(String query) {

        long startGetVector = System.nanoTime();
        double ms;
        String norm = normalizeQuery(query);
        float[] normToVector;
        String path = null;
        double apiTime = 0;
        double saveTime = 0;

        try {

            if (lruCached.containsKey(norm)) {
                path = "L1_hit";
                normToVector = lruCached.get(norm);
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
                queryVectorCacheRepository.save(cache);
                saveTime = (System.nanoTime() - saveStart) / 1_000_000.0;
            }

        } finally {
            ms = (System.nanoTime() - startGetVector) / 1_000_000.0;
            log.debug("path={} term={} save_ms={} api_ms={} total_ms={}", path, norm, saveTime , apiTime, ms);
        }

        return normToVector;

    }
```

## 빠르긴한데...?

| 경로 | 응답 시간 |
|---|---|
| L1 히트 | 0.0098 ms |
| DB 히트 | 15.2 ms |
| API 미스 | 216.5 ms |

꽤 빨라진거 같은데, 막상 DB 히트랑 비교해서는 차이도 별로 없죠.<br>
DB에서 히트나면 200ms를 줄였는데 반해,<br>
L1은 숫자가 저렇게 찍혀있어서 빨라 보이는거지 15ms 안팍이라서 체감을 거의 없을거 같습니다.

허나, 이제 지금 db는 와이파이 하나를 건너서, 바로 옆에 있는 랩탑을 사용중이기에, DB와의 거리가 좀 더 멀어지거나 하면, 아마 체감이 좀 되지 않을까 하는 행복한 상상을 해봅니다.

## DB 저장 시간은?

저번에 [측정](/posts/Embedding-Project-6/)한 결과랑 차이가 좀 있네요.<br>
저번 측정 결과는 서로 다른 시점에다가 재현이 불가능해 믿을게 못되서 이번 측정을 기준으로 삼을까 합니다.

| 구간 | 지난 측정 | 이번 측정 |
|---|---|---|
| 캐시 저장 | 77.7 ms | 23.5 ms |
| 정규화 + 캐시 조회 | 12.1 ms | 6.9 ms |
| 손익분기 히트율 | 32.7% | 14.6% |

다행이도 손익분기가 좀 많이 줄어서, 캐시 히트가 덜 나도, 이득보는 상황이 좀 더 낭낭해졌습니다.

## 다음 시간

다음 시간에는 그러면 이제 동시에 요청이 들어오면 박살나는 L1 캐시를 한번 안고장나게 만져봅시다.