---
title: containsKey를 꼭 만져야 동시성을 해결하나?
date: 2026-09-09 16:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---

## 지난 시간

[저번 시간](/posts/Embedding-Project-12/)에 containsKey 어쩌구 하면서 null값 받을 수도 있다고 그랬죠.<br>
그러면 오늘 이어서 알아봅시다.

## 안전?

```java
if (lruCached.containsKey(norm)) {
    normToVector = lruCached.get(norm);
    return normToVector;
}
```

위 코드에서 containsKey에다가 synchronized를 오버라이드를 했다고 가정해봅시다.<br>
lruCached에 put/get에도 오버라이드 되어있고요.<br>
그럼 동시 요청시 안전할까요?

답은 안전하지 않습니다. 왜냐면, synchronized는 잠그는 범위가 달라요.<br>
락의 단위가 메서드 도는 동안만 이고, 메서드가 끝나면 바로 풀어버려요.

## 그래서 무슨 일이 발생하냐면

| 순서 | 스레드 A (금리 검색) | 스레드 B (경제 검색) |
|---|---|---|
| 1 | `containsKey("금리")` → true | |
| 2 | | `put("경제", ...)` → 꽉 차서 `금리` 축출 |
| 3 | `get("금리")` → **null** | |
| 4 | null을 반환. 로그엔 `path=L1_hit` | |

참고로, lruCache에는 synchronized가 안 붙습니다. 필드에는 못 붙여요. 오직 메서드나 블록에만 붙힐 수 있습니다.

## 해결책

이걸 해결하려면 2가지 방법이 있습니다.

1번 : 락 범위를 크게 잡기
2번 : 호출 횟수 줄이기

저는 2번을 사용하려고 합니다.<br>
1번 같은 경우에는 락의 범위가 넓어질 수록 다른 사람이 대기해야 하는 시간이 길어질 수도 있으니깐요. <br> 저번 시간에 말한거 처럼 API를 대기 하는 시간까지 포함될 수 있거든요.

그러면, 호출 횟수를 줄여버리면 되겠죠?

```java
if (lruCached.containsKey(norm)) { // 1번
    normToVector = lruCached.get(norm); // 2번
    return normToVector;
}
```

위에 코드 보시면, lruCached 2번 호출이 됩니다.
여기서 그러면 따로 값을 1번만 받아 버리면 되는거 아닐까요?

```java
normToVector = lruCached.get(norm);

if(normToVector != null){
	path = "L1_hit";
	return normToVector;
}
```

이런식으로요.<br>
위 처럼 구현 하면 1번만 호출하고도 값을 가져올 수 있겠죠.

테스트 돌려보니깐 무사히 통과가 되는걸 알 수 있습니다.

## 문제중 하나만

지금 문제중 하나만 확인하고 넘어 갑시다.<br>
확인한 순간에 참이라고 그 다음 순간에도 참일까요? 대부분 제가 지금껏 진행해온 문제 풀이나 코드에서는 맞았습니다.<br>
근데, 동시에 들어오는 순간 이게 달라지더라고요.<br>
그래서 아까 처럼 동시에 못하도록 잠구거나, 메서드 사이사이를 없애서 확인과 행동을 하나로 합쳐야합니다.

그래서 처음 보는 같은 검색어가 동시에 들어온다면 어떻게 될까요?

두 검색어 다 L1에서 미스, DB에서도 미스가 나서, else로 빠지고 그리고는 API 호출 2번 + DB에 저장 2번 일어나겠죠.<br>
여기서 문제는 put 2번은 어차피 덮어 쓰고, API 호출은 2번 되서 낭비인데, <br>
진짜진짜 문제는 DB에 중복 저장이 문제입니다.

```java
@Table(uniqueConstraints = @UniqueConstraint(
    columnNames = {"normalized_query", "model"},
    name = "uk_query_vector_cache_query_model"))
```

(normalized_query, model)에 유니크 제약이 걸려 있어서, 늦게 도착한 save()가 이걸 위반해버리는 바람에 사용자는 500에러 봅니다.<br>
검색어 멀쩡하고 벡터값도 받았는데 말이죠.

```text
"1231415123"로 동시에 2개 요청을 보낸 결과, 한 쪽은 성공했지만 다른 한 쪽은 500 에러가 발생했습니다.

테스트 결과:

- 요청 1: ✅ 정상 응답 (검색 결과 HTML 반환)
- 요청 2: ❌ HTTP 500 Internal Server Error 발생
  - 응답 내용: {"timestamp":"2026-09-09T05:12:25.187Z","status":500,"error":"Internal Server Error","path":"/search"}
```

에이전트 통해서 2개 동시 요청 보내니 이런 결과가 나옵니다.

`DataIntegrityViolationException: could not execute statement [ERROR: duplicate key value violates unique constraint "uk_query_vector_cache_query_model"`

로그에 오류도 보이고요.

유니크 제약이 없었다면 어떻게 됬을까요? <br>
중복 행 두 개가 조용히 들어가고, 그러면 다음 조회에서 findByNormalizedQueryAndModel이, [저번에](/posts/Embedding-Project-6/) 한번 말했다 싶히, Optional 하나를 기대하는데 2건이 나와서 터지겠죠? 지금보다 훨씬 찾기 어려운 자리에서 터질겁니다.<br>

몰래 실패하는거보다 티나게 실패하는게 고치기 쉽고, 그렇게 만들어야 하는거 같습니다. 생각보다 어렵지만요.

그러면 일단 문제 발생한 부분 응급처치를 해봅시다.

## 응급처치

```java
else {
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

```

기존 코드에서,

```java
else {
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
                    
                }
                saveTime = (System.nanoTime() - saveStart) / 1_000_000.0;
            }
```

이렇게 바꿨습니다.

왜 else안에 있는거 다 안감싸요? 라고 물어보실 수 있겠죠.<br>
catch를 쓴다는 건 "이 예외가 나면 무시해도 된다"고 보증하는 것과 같습니다. <br>

즉, 유니크 제약을 위반한 경우에만 유효하다는 증표를 쥐여주는거죠. <br>
범위를 넓히면 어떨까요? 다른 이유로 같은 타입 예외가 나도 똑같이 무시하겠죠.<br>
아까 이야기한 조용한 실패 혹은 어디서 실패했는지 찾기 힘들어지겠죠.

## 왜 catch 안채워요?

솔직히 말하면 채울게 없습니다.<br>
에러 나면 저장을 안하면 되거든요.

그런데, 안채우는거보다 채우는게 낫습니다.<br>
왜냐하면 현재의 저는 매우 확신에 차있지만, 미래의 저는 바보기 때문에, 저는 미래의 저에게 친절해야할 필요가 있습니다.

"엥? 이게 왜 비어있지? 코드 짜다가 말았나? 아니면 일부러 비워뒀나?" 라고 할 수 있기 때문에, 채우는게 낫습니다.

```java
                try {
                    queryVectorCacheRepository.save(cache);
                } catch (DataIntegrityViolationException e) {
                    path = "API_miss_dup";
                }
```

이렇게요. 나중에 로그 찍어볼때, 어디서 에러 났는지 알 수 있겠죠 이러면.<br>
그러면 테스트를 한번 해 봅시다.

## 다시 테스트

```text
"나는바보입니다"로 동시에 2개 요청을 보낸 결과입니다.

테스트 결과:

- 요청 URL: http://localhost:8080/search?query=%EB%82%98%EB%8A%94%EB%B0%94%EB%B3%B4%EC%9E%85%EB%8B%A4
- 요청 1: ✅ 성공 (검색 결과 없음 페이지 반환)
- 요청 2: ✅ 성공 (검색 결과 없음 페이지 반환)
```

```text
2026-09-09T14:45:39.447+09:00 DEBUG 10344 --- [newsFinder] [nio-8080-exec-3] o.j.n.queryVector.QueryVectorService     : path=API_miss term=나는바보입다 save_ms=24.476 api_ms=188.4605 total_ms=225.5737
2026-09-09T14:45:39.447+09:00  WARN 10344 --- [newsFinder] [nio-8080-exec-4] org.hibernate.orm.jdbc.error             : ERROR: duplicate key value violates unique constraint "uk_query_vector_cache_query_model"
  Detail: Key (normalized_query, model)=(나는바보입다, voyage-4-lite) already exists.
2026-09-09T14:45:39.460+09:00 DEBUG 10344 --- [newsFinder] [nio-8080-exec-4] o.j.n.queryVector.QueryVectorService     : path=API_miss_dup term=나는바보입다 save_ms=37.3564 api_ms=181.4975 total_ms=229.5606
```

`나는바보입니다`로 시켰는데 `나는바보입다`로 들어간건, 에이전트가 바보라서 그렇고요.<br>
어차피 같은 검색어 동시에 들어갔으니깐, 별로 상관이 없겠네요.<br>

api_ms=188.4605<br>
api_ms=181.4975

API도 2번 씩 호출 합니다. 낭비죠. 


## 다음 시간

이번 시간에는 containsKey를 만지지 않고, 호출 횟수를 줄여서 L1 캐시 부분을 해결했고, L1 + DB 미스일때 동시에 오면 고장 나는 부분도 임시로 땜빵을 해뒀습니다.

근데 진짜 말 그대로 임시 입니다.<br>
위에 로그를 보시면 API를 2번 호출하죠. <br>

[다음시간](/posts/Embedding-Project-14/)에는아무리 Voyage가 싸다고 하지만, 초절정인기웹사이트가 되버리면 그 API 비용도 아깝잖아요. 그것도 한번 절약을 해 봅시다.