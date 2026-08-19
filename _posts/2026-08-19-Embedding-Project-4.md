---
title: pgvector, DB한테 계산시키기
date: 2026-08-19 12:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, pgvector, postgresql, study]
---

## 지난 시간

캐시로 파싱을 없애서 28.5ms를 줄였습니다.<br>
근데 데이터가 커지면? 벡터 하나가 대략 24KB니까 3만 건이면 700MB예요. 많이 크죠.

그리고 생각해보면, 유사도 계산을 자바가 하고 있으니까 벡터를 자바가 들고 있어야 했던 겁니다.<br>
계산을 다른 데서 하면 안 들고 있어도 되겠죠?

## pgvector가 뭐냐면

PostgreSQL에 붙이는 확장입니다.<br>
설치하면 vector라는 컬럼 타입이 생기고, 벡터끼리 비교하는 연산자가 딸려옵니다.

저번에 짠 코사인 유사도 처럼 생고생 없이, 이걸로 대체가 가능합니다.

```sql
embedding <=> '[0.1, 0.2, ...]'
```

<=>가 코사인 거리를 계산합니다. 유사도가 아니라 거리라서 작을수록 비슷해요.<br>
유사도로 바꾸려면 1 - (a <=> b) 하면 됩니다.

## 준비

저는 H2를 쓰고 있어서 DB부터 갈아야 했습니다.<br>
도커로 postgres를 띄우는데, pgvector 이미지를 쓰면 확장이 이미 들어 있어서 편합니다.

띄우고 나서 확장을 켭니다.

```sql
CREATE EXTENSION vector;
```

## 컬럼을 벡터로

엔티티에서 임베딩 필드는

```java
@Column(length = 30000)
private String embedding;
```
여기서 이렇게 바뀝니다.

```java
@JdbcTypeCode(SqlTypes.VECTOR)
@Array(length = 1024)
private float[] embedding;
```

hibernate-vector라는 의존성을 추가해야 가능합니다. 없으면 하이버네이트가 vector 타입을 몰라서 못 만들어요.<br>
@Array(length = 1024)의 1024가 vector(1024)의 괄호 안 숫자가 됩니다.

## 도미노

타입 하나 바꿨더니 이것저것 줄줄이 딸려왔습니다. 좋은 뜻으로요.

저번 코드는 벡터를 문자열로 바꾸고 저장하고, 다시 꺼내서 문자열을 벡터로 바꾸고 했는데,
컬럼이 진짜 벡터가 됐으니 문자열로 바꿔 넣을 이유가 이제는 없어졌죠. 거꾸로도 마찬가지겠죠?<br>
그럼 EmbeddingConverter.toText()가 필요 없습니다. toVector()도 필요 없습니다.
자바에서 코사인 유사도 계산할때 쓰던 List<Double>도 필요 없어졌어요.<br>

그래서 Voyage에서 받는 것부터 아예 float[]로 바꿨습니다. 이제는 벡터가 float니깐요.

```
전: JSON → List<Double> → String → varchar 컬럼
후: JSON → float[] → vector 컬럼
```

중간 단계가 통째로 없어졌습니다. 훨씬 간단해졌습니다.

## 검색 쿼리

이게 핵심인데,

```java
@Query(value = """
        SELECT title,
               summary,
               url,
               published_at AS "publishedAt",
               source,
               1 - (embedding <=> CAST(:queryVector AS vector)) AS "score"
        FROM news
        WHERE 1 - (embedding <=> CAST(:queryVector AS vector)) >= :minScore
        ORDER BY "score" DESC
        LIMIT :limit
        """, nativeQuery = true)
List<NewsSearchProjection> searchByVector(@Param("queryVector") float[] queryVector,
                                          @Param("minScore") double minScore,
                                          @Param("limit") int limit);
```

전에 자바로 하던 게 뭐였는지 하나씩 대보면 이렇습니다.

- `cosineSimilarity(...)` → `1 - (embedding <=> ...)`
- `if(score >= MIN_SEARCH_SCORE)` → `WHERE`
- `results.sort(...reversed())` → `ORDER BY ... DESC`
- `subList(0, 10)` → `LIMIT`

네 개가 전부 SQL 다섯 줄로 옮겨갔습니다.

그래서 search()에 남은 게 이것뿐이에요.

```java
public List<NewsSearchResult> search(String query) {

    float[] queryToVector = embeddingClient.embedQuery(query);
    List<NewsSearchProjection> newsList =
            newsRepository.searchByVector(queryToVector, MIN_SEARCH_SCORE, MAX_SEARCH_SIZE);

    List<NewsSearchResult> results = new ArrayList<>();

    for(NewsSearchProjection news : newsList) {
        results.add(new NewsSearchResult(
                news.getTitle(), news.getSummary(), news.getUrl(),
                news.getPublishedAt(), news.getSource(), news.getScore()));
    }

    return results;
}
```

계산도 필터도 정렬도 없습니다. 받아서 옮겨 담기만 하면 끝입니다.

이제는 300건 전체를 불러와서 290개를 버리고 10개만 표시해주는게 아니라, 그냥 딱 상위 검색 결과 10개만 가져옵니다. 훨씬 효율적이겠죠?

## 지운 파일들

- `EmbeddingSimilarity` — 1편에서 직접 짠 코사인 유사도
- `EmbeddingConverter` — 문자열 변환
- `NewsCache`, `CachedNews` — 지난 편에서 만든 캐시

테스트 두 개도 마찬가지로 지웠습니다.

## 근데 왜 안빨라짐?

안빨라져요.

전건 스캔이든 pgvector든 300건에서는 몇 ms 차이고, 응답 시간은 여전히 200ms 근처예요.<br>
지난 편에서 봤듯이 그 200ms는 검색어를 벡터로 바꾸는 API 왕복이라 DB랑 상관이 없습니다.

애초에 300건은 어떻게해도 빨라서, 더 빠르게 하기가 힘듭니다.

## 배운 점

- 타입 하나 바꿨더니 파일 네 개가 지워졌습니다. List<Double>이나 문자열 변환이 전부 자바한테 시켜서 필요했었는데, 계산 위치를 DB로 옮기니 자바에서 쓰던것들이 필요 없어지더라고요.
- 인터페이스는 한 곳만 어긋나도 쓰는 쪽이 전부 깨집니다. EmbeddingClient 두 줄 때문에 세 파일에 빨간 줄이 떴는데, 고칠 곳은 한 군데였어요. 에러가 흩어져 있으면 공통 원인부터 찾는 습관을 가지면 좋을거 같습니다.

## 다음 시간

300건은 너무 적으니, 데이터를 늘려서 한번 계산해볼까 합니다.
