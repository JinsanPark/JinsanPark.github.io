---
title: L1 캐시
date: 2026-09-03 09:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---


## 지난 시간

지난 시간에 간단한 LRU 코드를 만들어 봤습니다.<br>
이번 시간에는 진짜 L1에다가 한번 올려봅시다.

일단 저번에 봤듯이, getVector의 흐름은, 

`normalize -> DB 조회 -> 있으면 반환 / 없으면 API 호출 후 DB 저장`

요런 식이죠.

그런데 이제는

`normalize -> L1 조회 -> 있으면 반환 / 없으면 DB 조회 -> 있으면 반환 / 없으면 API 호출 후 DB 저장`

이런식으로 DB 바로 위에다가 L1을 얹어줄겁니다.

## 구현은 별거없음

if문을 하나 더 만들면 되겠죠.

```java
       if (lruCached.containsKey(norm)) {
           normToVector = lruCached.get(norm);
           return normToVector;
       }
```

이런 식으로요.<br>
그러면 L1 캐시 미스는요? 라고 물으실 수 있겠는데, 어차피 L1 캐시가 미스나면, DB단에서 처리 해주면 되죠.<br>
굳이 L1 미스 분기를 하나 더 만들어서, 거기서 API 호출해서 L1에 넣고 다시 DB단에서 미스나서 다시 API 호출하고... <br>
이럴 필요 없으니깐요. 미스 나면 자연스럽게 밑으로 넘기면 됩니다.

## 벡터 모델 바뀌면 큰일난다며

예리하신 분은, 어? 벡터값 바뀌면 큰일 나는거 아니에요? <br>
저번에 `findByNormalizedQueryAndModel(norm, voyageModel)` 처럼 DB는 모델값 안섞이게 따로 넣었잖아요?<br>
라고 물어보실수 있는데, 아닙니다.<br>
이유는 메모리 캐시잖아요. 앱으로 켤때 모델은 딱 한번 정해집니다.<br>
그리고 끄거나 재부팅하면 다 사라져요. 즉, 벡터끼리 모델이 바뀌여도 뒤섞일 일이 아예 없습니다. 반면 DB는 오래 가잖아요. <br>
막 몇달씩 쓰다가 좋은 모델 나와서 바꿨는데, DB에는 캐시 그대로 있어서, 있는데 이상한 결과를 돌려줄 수 있잖아요.

참고로 `new LruCache<>(10)` 지금 LRU 크기가 고작 10인데, 메모리값 비싸다고 아끼는게 아니라, 눈으로 캐시값 밀려 나는거 보려고 작게 잡았어요. 나중에 히트율 보고 늘리려고요. 

## 테스트와 예측 해보기

일단 기존에 테스트는 깨지지 않았습니다.

if문이 하나 늘어나고 그 부분이 lruCache 히트 니깐 그 부분 테스트랑, DB캐시 히트/미스 부분에 L1 캐시가 들어가는걸 추가 해야 겠죠.

일단 LRU 캐시 히트시에 DB를 조회하는지 확인해야겠죠.

```java
    @Test
    void LRU_캐시히트시_DB_조회_1번 (){
        QueryVectorCache cached = new QueryVectorCache("테스트", MODEL, new float[]{1.0f, 0.0f}, LocalDateTime.now());
        when(queryVectorCacheRepository.findByNormalizedQueryAndModel(any(), any())).thenReturn(Optional.of(cached));
        queryVectorService.getVector("테스트");

        queryVectorService.getVector("테스트");

        verify(queryVectorCacheRepository, times(1)).findByNormalizedQueryAndModel(any(), any());
        verify(embeddingClient, never()).embedQuery(any());
    }
```

우선 위 코드는 잘 통과 합니다.<br>
첫번째 "테스트"에서는 DB캐시와 L1에 넣고,
두번째 "테스트"에서는 L1에 있는 벡터값을 확인하고,
마지막 then에서는 DB캐시가 몇번 불렸는지 확인하는겁니다.

DB캐시가 한번 호출되겠죠? 첫번째 "테스트"에서 L1에다가 벡터값을 넣어줬으니깐요.

근데, lruCached.put()을 지우면 잘 통과 할까요? <br>
제 예상은 put이 없으니깐 DB를 2번 호출한다고 예상했습니다.

```text
-> at org.jin.newsfinder.queryVector.QueryVectorServiceTest.LRU_캐시히트시_DB_조회_1번(QueryVectorServiceTest.java:42)
But was 2 times:
```

정확했죠.

그러면 LRU 캐시 미스시 테스트도 한번 봐야겠죠.

```java
    @Test
    void 두번_불러도_API는_1번(){
        when(queryVectorCacheRepository.findByNormalizedQueryAndModel(any(), any())).thenReturn(Optional.empty());
        when(embeddingClient.embedQuery(any())).thenReturn(new float[]{0.1f, 0.1f});
        queryVectorService.getVector("테스트");

        float[] result = queryVectorService.getVector("테스트");

        assertThat(result).containsExactly(0.1f, 0.1f);
        verify(embeddingClient, times(1)).embedQuery(any());
    }
```

getVector의 마지막 else에서 lruCached.put()을 지우고 돌리면 어떻게 될까요?

API 2번 호출하겠죠. 왜냐하면, put이 없으니 L1에 안올라가고, 그렇게 되면 L1에 캐시 히트는 영영 없을 거고, 그러면 API를 호출하도록 되어 있으니깐요.

```text
-> at org.jin.newsfinder.queryVector.QueryVectorServiceTest.두번_불러도_API는_1번(QueryVectorServiceTest.java:52)
But was 2 times:
```

이것도 맞았네요.


## 마무리

저번 시간에 말한거 처럼, NewsService. 진짜 단 한글자도 안건들였습니다.

그래서 L1 조회를 왜 넣어요? 왜 빨라요?<br>
L1 조회는 메모리 안에서 끝나요. 사실상 공짜죠. 메모리 먹는거 빼고는.<br>
DB를 조회하는데는 아마 지금은 와이파이로 연결된 랩탑(DB)과 데스크탑의 왕복이라, DB왕복 시간을 줄인다고 해서 엄청 빨라지지는 않을 것 같긴합니다.

근데 단점은 아까 말했듯이 앱 재부팅하면 사라져요.<br>
DB에 넣는것도 캐시를 최대한 재활용해서 쌀먹을 하기 위함이고요.

## 다음 시간

아직 할게 많이 남긴 했습니다.<br>
문제가 아직 있긴 합니다.<br>
이게 웹사이트를 혼자 쓰는게 아니잖습니까? 저랑 제 친구가 동시에 검색하면, 얘는 여러 요청을 못 버텨요.<br>
근데, 아쉽게도 다음 시간에는 이거를 안 다룰거고요.<br> 
다다음시간이나 다다다음 시간에 다뤄 볼거고, <br>
다음 시간에는 진짜 DB 조회 시간까지 아낄 수 있는 지, 측정이나 한번 해보려고 합니다.