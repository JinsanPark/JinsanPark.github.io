---
title: 캐시로 검색 성능 향상시키기?
date: 2026-08-14 00:30:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, 캐시, 성능측정, study]
---

## 지난 시간에

배치로 묶어서 적재 시간을 58.6초에서 3.5초로 줄였습니다.<br>
근데 적재는 최초 1회잖아요. 한 번 넣고 나면 다시 할 일이 없습니다. 적어도 적재를 자주 한다면, 성능의 차이를 느낄 수 있겠네요.

그러면, 검색쪽을 한번 봅시다.

```java
public List<NewsSearchResult> search(String query) {

    List<Double> queryToVector = embeddingClient.embedQuery(query);
    List<News> newsList = newsRepository.findAll();

    for(News news : newsList) {
        List<Double> newsVector = EmbeddingConverter.toVector(news.getEmbedding());
        double score = EmbeddingSimilarity.cosineSimilarity(queryToVector, newsVector);
        ...
    }
}
```

findAll()로 300건을 전부 꺼냅니다. 그리고 하나씩 유사도를 계산해요.<br>
검색할 때마다요.

## 파싱 챗바퀴

여기서 눈여겨볼 건 findAll()보다는 다른 코드인데,

```java
List<Double> newsVector = EmbeddingConverter.toVector(news.getEmbedding());
```

DB에 임베딩을 문자열로 저장해뒀거든요. 0.123,0.456,... 이런 식으로요.<br>
그래서 꺼낼 때마다 쉼표로 잘라서 숫자로 바꿔야 합니다.

벡터 하나에 숫자가 1024개고, 기사가 300건이니까

```
300 × 1024 = 307,200번
```

검색 한 번에 parseDouble을 30만 번씩 부르고 있는거죠.

## 어차피 기사는 잘 안바뀜

근데 생각해보면 기사들이 바뀌는 경우가 많지는 않잖아요?<br>
앱 켜져 있는 동안 계속 똑같습니다.

그럼 켤 때 한 번만 파싱해서 들고 있으면 되겠죠.

먼저 담아둘 곳부터 만들었습니다.

```java
public record CachedNews(String title, String summary, String url, String source,
                         LocalDateTime publishedAt, List<Double> vector) {
}
```

News 엔티티랑 거의 같은데 두 가지가 다릅니다.<br>
content가 없어요. 기사당 만 자가 넘고, 요약으로 어느정도 주제를 잡을 수 있어서 빼버렸고요.<br>
그리고 벡터가 String이 아니라 List<Double>입니다. 이게 핵심이에요. 이미 파싱된 상태로 들고 있는 겁니다.

## 코드

```java
@Component
public class NewsCache {

    private final NewsRepository newsRepository;
    private List<CachedNews> cachedNewsList;

    public NewsCache(NewsRepository newsRepository) {
        this.newsRepository = newsRepository;
    }

    @PostConstruct
    public void loadCache() {

        List<News> newsList = newsRepository.findAll();
        this.cachedNewsList = new ArrayList<>();

        for (News news : newsList) {
            cachedNewsList.add(new CachedNews(
                    news.getTitle(), news.getSummary(), news.getUrl(), news.getSource(),
                    news.getPublishedAt(), EmbeddingConverter.toVector(news.getEmbedding())));
        }
    }

    public List<CachedNews> getCachedNewsList() {
        return cachedNewsList;
    }
}
```

`@PostConstruct`가 붙은 메서드는 스프링이 이 객체를 다 만들고 나서 자동으로 한 번 불러줍니다.

## return이 없는데 값이 있다

저는 처음에 이렇게 짰어요.

```java
@PostConstruct
public List<Double> vector() {
    newsRepository.findAll();
}
```

@PostConstruct가 뭔가를 만들어서 돌려주는 거라고 생각했거든요.<br>
근데 이 메서드는 제가 부르는 게 아니라 스프링이 부릅니다. 스프링이 그 반환값을 받아서 쓸 데가 없어요. 그냥 버립니다.

그러니까 돌려주는 게 아니라 어딘가에 담아둬야 하는 거였습니다.<br>
지금까지 짠 코드는 전부 제가 직접 부르는 것들이었어서, return이 없는데 값이 어딘가에 있고, 그걸 참조할 수 있다는게 좀 낯설었어요.

## 검색

이제 DB 대신 캐시를 보게 바꿉니다.

```java
List<CachedNews> newsList = newsCache.getCachedNewsList();

for(CachedNews news : newsList) {
    double score = EmbeddingSimilarity.cosineSimilarity(queryToVector, news.vector());
    ...
}
```

toVector 줄이 통째로 사라졌습니다.<br>
news.vector()가 이미 List<Double>이니까 바꿀 게 없거든요. 30만 번짜리 파싱이 여기서 없어집니다.

## 측정

지난번엔 1회 측정이었는데 이번엔 좀 제대로 했습니다.<br>
ai 에이전트 갈궈서 검색어 50개를 각각 5회씩, 중앙값으로요.

| | 중앙값 | p95 |
|---|---|---|
| 캐시 없음 | 225.3 ms | 273.1 ms |
| 캐시 있음 | 196.8 ms | 233.8 ms |

28.5ms 줄었습니다. 12% 정도네요.

와 12% 감소! 인데, 막상 구체화된 수치 보면 28.5ms입니다.
얼마 차이가 없죠. 하지만 50개 다 빨라졌죠?

## 남은 딜레이의 정체?

196.8ms 저번 시간에 본 것 같지 않습니까?

Voyage에 보내고 받는 시간이 200ms 정도였잖아요, 검색어도 마찬가지에요.<br>
검색어를 벡터로 바꿔서 DB와 유사도를 검색해야하기 때문에 API응답시간 만큼 느려지는거죠.


```
캐시 없음: 225ms = API 왕복 200 + 조회·파싱 25
캐시 있음: 197ms = API 왕복 200 + 거의 0
```

캐시를 넣으니까 응답 시간이 그냥 API 왕복 시간이 됐습니다.<br>

## 배운 점

- 25ms 가량 밖에 못줄인다고 쓸모 없어 보이지만, 파싱은 건수에 비례하니까, 300건에 25ms면 3만 건이면 2.5초입니다. 데이터셋이 커지니깐 꽤 할만한 딜 같아 보이지 않나요? 근데 더 커지면 메모리가 터지는데, 이건 다른 방법을 써야합니다. pgvector + 인덱스 같은거요.


## 다음 시간

저는 학습자라서 그냥 공부용으로 캐시를 만들어 본거라, 여러분은 이렇게 직접 캐시짜면서 고생 하지 마시고, pgvector 같은거 쓰시면 됩니다.
다음 시간에는 pgvector에 대해서 살짝 알아 봅시다.
