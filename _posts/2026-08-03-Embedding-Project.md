---
title: 코사인 유사도와 벡터 검색
date: 2026-08-03 18:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, 임베딩, 코사인유사도, study]
math: true
---

## 임베딩 활용한 검색기 만드는 중

임베딩으로 검색하는 걸 만들어 보고 있습니다.<br>
예를 들어 "코스피 하락"이라고 검색했는데, 그 단어가 없지만, 연관성이 깊은 단어를 찾아주는거요. 예를들어, "레버리지 규제", "반도체 하락 마감" 이런것들이요.

이게 왜 필요하냐면,<br>
그냥 검색은 글자가 똑같아야 찾아주거든요.<br>
"코스피 하락"로 검색하면 "사이드 카 이틀 연속 발동", "반도체 레버리지 규제 서둘러"같은 기사는 안 나와요. 글자가 다르니까요.<br>
근데 딱 보기에도 요즘 같은 시국에, 예시로 준 검색어와 내용이 딱 연관이 있어 보이죠?

그래서 임베딩이라는 걸 씁니다.<br>
문장을 집어넣으면 벡터라는 숫자 뭉치로 바꿔주는 API가 있어요. 여러 API가 있는데, 저는 Voyage AI 썼습니다. 무료 토큰을 많이 줘서 쌀먹하기 좋습니다.<br>
비슷한 뜻이면 비슷한 숫자가 나와요.

"비슷한 숫자"인지를 컴퓨터가 어떻게 알까요? 벡터 하나에 숫자가 1024개씩(더 많이, 더 적게도 가능합니다) 들어 있는데, 그걸 서로 비교하는게 코사인 유사도예요.

## 길이보다는 방향

저랑 제 친구가 각자 손가락으로 어딘가를 가리킨다고 쳐봐요.
그리고 둘 다 편의점을 가리켰어요. 그럼 같은 생각인 거죠.<br>
저는 편의점을 가리키는데 친구는 정반대 뒤쪽을 가리켰어요. 완전 다른 생각이고요.

근데, 여기서 생각해볼게, 방향을 가리키는데, 길이가 중요할까요?
팔 긴 사람이 가리키든 짧은 사람이 가리키든,<br>
편의점 쪽을 가리켰으면 둘 다 편의점 얘기하는 거잖아요?

이게 핵심이에요. 방향만 보고 길이는 안 봐요.

왜 이게 중요하냐면요,<br>
기사 하나는 3줄인데 다른 하나는 30줄이에요. 근데 내용은 똑같아요.<br>
여기서 길이까지 따지면 이 둘이 다르다고 나옵니다.<br>
근데 우리가 궁금한 건 "무슨 얘기냐"지, "얼마나 길게 썼냐"가 아니잖아요?

## 수식은?

$$ \cos\theta = \frac{A \cdot B}{|A||B|} $$

무슨 소린지 모르겠죠? 저도 몰라요.
같이 알아 봅시다.

|A| 이게 A라는 손가락의 팔 길이예요.<br>
분모에 팔 길이 두 개를 곱해서 나누고 있죠?<br>
아까 "길이는 상관없다"고 했잖아요. 그걸 수식으로 쓰면 이렇게 됩니다.<br>
나눠서 지워버리는 거예요.

위에 있는 $A \cdot B$ 는 내적이라고 하는데,<br>
그냥 두 손가락이 같은 쪽을 보고 있으면 커지고, 반대쪽을 보면 마이너스가 되는 숫자라고 생각하면 돼요.

그래서 결과는 -1에서 1 사이로 나와요.

- 1이면 같은 방향
- 0이면 아무 상관 없음
- -1이면 정반대


라이브러리 쓰면 되는거 아닌가요? 라고 하실 수 있는데, 맞습니다. 저는 학습자라서, 그냥 어떻게 굴러가는지는 최소한 알아야할거 같아서 코드로 직접 구현해 봤습니다.

## 코드

수식 그대로 옮기면 됩니다.


```java
public class EmbeddingSimilarity {
    public static double cosineSimilarity(List<Double> vector1, List<Double> vector2){
        double innerProduct = 0;
        double vector1Mag = 0;
        double vector2Mag = 0;

        for(int i = 0; i < vector1.size(); i++){
            innerProduct += vector1.get(i) * vector2.get(i);
            vector1Mag += Math.pow(vector1.get(i),2);
            vector2Mag += Math.pow(vector2.get(i),2);
        }

        vector1Mag = Math.sqrt(vector1Mag);
        vector2Mag = Math.sqrt(vector2Mag);

        return  innerProduct / (vector1Mag * vector2Mag);
    }
}
```

반복문 한 번 돌면서 세 개를 다 모아요. 내적 하나, 팔 길이 두 개.<br>
제곱해서 더하고 마지막에 제곱근 씌우는 게 길이 구하는 거고요.

## 근데 이거 테스트는?

저는 1024 차원 쓸 거라 실제로 넣을 벡터는 숫자가 1024개예요.<br>
이걸 제가 손으로 계산해서 "정답은 0.8237이어야 함" 이럴 수가 없잖아요.

그래서 정답 맞히는 걸 포기했습니다.<br>
대신 무조건 이렇게 나와야 하는 것들만 확인하기로 했어요.<br>
2차원이면 저도 답을 아니까요.

```java
    @Test
    void 같은_백터는_1임(){

        List<Double> vectorA = List.of(1.0, 1.0);
        List<Double> vectorB = List.of(1.0, 1.0);

        Double result1 = EmbeddingSimilarity.cosineSimilarity(vectorA,vectorB);

        assertThat(result1).isCloseTo(1.0,within(0.0001));
    }

    @Test
    void 직각은_0임(){

        List<Double> vectorA = List.of(1.0, 0.0);
        List<Double> vectorB = List.of(0.0, 1.0);

        Double result1 = EmbeddingSimilarity.cosineSimilarity(vectorA,vectorB);

        assertThat(result1).isCloseTo(0.0,within(0.0001));
    }

    @Test
    void 다른_방향_백터는_마이너스1임(){

        List<Double> vectorA = List.of(1.0, 1.0);
        List<Double> vectorB = List.of(-1.0, -1.0);

        Double result1 = EmbeddingSimilarity.cosineSimilarity(vectorA,vectorB);

        assertThat(result1).isCloseTo(-1.0,within(0.0001));
    }

    @Test
    void 길이가_달라도_방향이_같은_백터는_1임(){

        List<Double> vectorA = List.of(1.0, 1.0);
        List<Double> vectorB = List.of(3.0, 3.0);

        Double result1 = EmbeddingSimilarity.cosineSimilarity(vectorA,vectorB);

        assertThat(result1).isCloseTo(1.0,within(0.0001));
    }
```

isCloseTo 쓴 이유는, 실수는 ==로 비교하면 안 되거든요.<br>
계산하다 보면 0.9999999 같은 게 나와요. 그래서 "이 정도면 맞다고 치자"로 퉁치는거죠.

네 번째가 제일 중요해요.<br>
[1,1]이랑 [3,3]은 길이가 3배 차이 나는데 결과는 1이 나와야 하거든요.<br>
아까 말한 길이 얘기, 3줄 기사랑 30줄 기사 얘기.<br>
그게 이 테스트 하나로 확인되는 거예요.

## 배운 점

- 테스트 할때, 너무 크게 바라보지 말고, 1024차원을 어떻게 테스트 하지? 같은 생각 말고, 진짜 작은걸로 쪼개서, 이차원 배열인 x축 y축 처럼 쪼개서 해도 될것 같네요. 어떻게 보면 제일 어렵기도 합니다. 알고리즘 문제 푸는거 처럼이요.

## 다음 시간
- 지금 코드는

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
    count++;

    if (count % 50 == 0) {
        System.out.println(count + "번째 입니다.");
    }
}
```

- 300번을 전부 다 한번씩 api에 요청하는데, 이게 주고 받는 시간이 평균 200ms가 됩니다.
200ms면 빠르네? 하실 수 도 있는데, 지금 요청을 고작 300건을 DB에 넣는데 1분 내외로 소요됩니다. 엄청 느리죠? 그러니 [다음 시간](/posts/Embedding-Project-2)에는 batch이용해서, 모아서 요청하는 법을 알아 봅시다.