
---
title: 임베딩 그리고 배치
date: 2026-08-13 02:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, 임베딩, 배치, study]
---

## 지난 시간에

코사인 유사도를 만들었으니, 이제 기사를 DB에 넣어야죠.<br>
더미 데이터 300건을 임베딩으로 바꿔서 저장하는 코드를 짰습니다.

```java
for (NewsArticle article : articles) {

    News news = new News(
            article.title(),
            article.content(),
            article.summary(),
            article.url(),
            article.source(),
            article.publishedAt()
    );

    List<Double> vector = embeddingClient.embedDocument(news.getSummary() + "\n" + news.getTitle());
    news.setEmbedding(EmbeddingConverter.toText(vector));
    newsRepository.save(news);
}
```

기사 하나 꺼내서, API에 보내고, 받은 벡터를 저장하고. 300번 반복입니다.

돌려봤더니 58.6초 나왔습니다.

## 300건 넣는데 1분?

근데 이상하죠. 300건이면 많은 것도 아니잖아요.<br>
그래서 어디서 시간을 쓰는지 보니깐,<br>
API 요청 한 번에 평균 200ms 정도 걸리더라고요.

200ms면 빠른데? 싶으실 텐데, 곱해보면 이렇게 됩니다.

```
300번 × 200ms = 60,000ms = 60초
```

실제로 걸린 58.6초랑 비슷하죠.<br>
느린 코드가 있는 게 아니라, 요청 보내고 받고 넣고. 이거를 300번 하니깐 느린거더라고요.

## AI야, Voyage API 공식 문서 좀 찾아보거라.

AI에게 Voyage API 문서 찾아보라고 시켰더니, input 파라미터가 배열을 받더라고요.<br>
즉, 한 번에 하나씩만 보낼 수 있는 게 아니라 여러건이 가능합니다. 저는 하나씩 보내고 있었고요.

한 번에 128개까지 됩니다.<br>
그러면 300건은 세 번이면 끝나죠.

## 코드

128개씩 잘라서 보내봅시다.

```java
public List<List<Double>> embedDocuments(List<String> texts){

    List<List<Double>> batchList = new ArrayList<>();

    //voyage4lite batch 요청 최대 크기 128
    for(int i = 0; i < texts.size(); i += 128){
        int end = Math.min(texts.size(), i + 128);
        List<String> chunk = texts.subList(i,end);
        batchList.addAll(embedBatch(chunk));
    }

    return batchList;
}
```

i를 1씩이 아니라 128씩 올립니다.<br>
subList로 그만큼 잘라내서 embedBatch에 넘기고요.

Math.min이 있는 이유는 마지막 조각 때문입니다.<br>
300건이면 128, 128, 44로 잘리는데, 마지막에 i + 128을 그대로 쓰면 인덱스 범위 밖이니깐요.

실제로 요청을 보내는 쪽은 이렇습니다.

```java
private List<List<Double>> embedBatch(List<String> chunk) {

    EmbeddingRequest embeddingRequest = new EmbeddingRequest(chunk, "voyage-4-lite", "document");
    List<List<Double>> result = new ArrayList<>();

    EmbeddingResponse response = restClient.post()
            .uri("https://api.voyageai.com/v1/embeddings")
            .header("Authorization", "Bearer " + apiKey)
            .body(embeddingRequest)
            .retrieve()
            .body(EmbeddingResponse.class);

    if(response == null || chunk.size() != response.data().size()){
        throw new EmbeddingApiException("데이터 매칭 실패", null);
    }

    for (EmbeddingData data : response.data()){
        result.add(data.embedding());
    }

    return result;
}
```

전에는 문자열 하나를 보냈는데 이제 덩어리를 통째로 보냅니다.<br>
받는 쪽도 벡터 하나가 아니라 여러 개를 꺼내야 해서 반복문이 붙었고요.

## 개수 확인

128개를 보냈으면 128개가 와야 하잖아요.<br>
안 그러면 기사랑 벡터가 어긋나서, 1번에 2번 벡터가 들어가거나 이상한 벡터값이 들어갈 수 있습니다.

심지어 이건 에러도 안 나서 그냥 검색 결과만 이상해져요.<br>
그래서 개수가 다르면 예외를 던지게 해뒀습니다.

## 결과

| | 요청 횟수 | 소요 시간 |
|---|---|---|
| 개선 전 | 300회 | 58.6초 |
| 개선 후 | 3회 | 3.5초 |

같은 데이터로 1회 측정입니다. 네트워크 상황에 따라 달라질 수 있고요.

## 배운 점

- 느리면 어디가 느린지 재보고 시작하기. 저는 반복문에 찐빠가 있는 줄 알았는데, 아니더라고요. API 쓸때는 AI 시켜서 공식문서 정독시키는것도 나쁘지 않은거 같습니다.

## 다음 시간

적재는 사실 한 번만 하면 끝입니다. 58초든 3.5초든 최초 1회니까요.

근데 검색은 할 때마다 300건을 전부 훑습니다.

```java
List<News> newsList = newsRepository.findAll();

for(News news : newsList) {
    // 300번 코사인 유사도 계산
}
```

기사가 3만 건이면 검색하다가 속 터지겠죠?<br>
다음 시간에는 이걸 방지할, 캐시에 대해서 알아 봅시다.