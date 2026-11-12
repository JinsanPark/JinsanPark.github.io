---
title: (Voyage) 서버가 맛이 가면
date: 2026-10-07 13:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, study]
---


## 지난 시간

[저번 시간](/posts/Embedding-Project-15/)에 보야지 서버가 터지면 무한 대기를 해야한다고 했었죠.<br>
그러면 간단하게 해결책이랑 몇가지 현 코드에서 발생하는 문제를 수정해 보도록 합시다.

## 연결과 읽기

API를 호출할때 대기 시간이 대략 180ms 정도인데, 만약 서버가 맛이 갔다고 합시다. <br>
근데 이것도 종류가 있습니다. 일단 연결은 간단하고, 쿡쿡 찔러서 반응을 안하면 연결이 안된다는 건데, 이거는 판단하기 쉽죠.<br>
문제는 연결은 됐는데 서버가 일이 너무 많아서 처리 하다가 멈추는 거에요. 제일 흔하죠. <br>
뭐 AI 같은거 API가져다가 쓰면 한참 있다가 503 오류 왕왕 보잖아요? 그런거 처럼요. 문제가 오래 걸린다는거에요.

저는 타임 아웃을 하나로 걸려고 합니다. 간단한 대신 검색 쪽은 느슨하게요.

## 구현

```java
SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
factory.setConnectTimeout(Duration.ofMillis(1000));
factory.setReadTimeout(Duration.ofMillis(5000));

RestClient restClient = RestClient.builder()
        .requestFactory(factory)
        .build();
```

이런거 쓸건데 어디다가 박아야 할까요?<br>

저는 처음에 embed안에다가 넣었는데, 이러면 안돌아 갑니다.<br>
검색할 때마다 RestClient를 새로 만들어버리는데, 객체가 만들어질 때 딱 한 번 넣으면 됩니다.

```java
    SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();

    public VoyageEmbeddingClient(@Value("${voyage.model}") String voyageModel) {
        this.voyageModel = voyageModel;
        this.restClient = RestClient.builder()
                .requestFactory(factory)
                .build();

        factory.setConnectTimeout(Duration.ofMillis(1000));
        factory.setReadTimeout(Duration.ofMillis(5000));
    }
```

저는 그래서 이런식으로 작성을 해봤습니다.<br>
curl.exe로 6번 쟀더니 순수 연결은 대부분 5 ms, 제일 느린 게 21 ms라서 그냥 넉넉 잡아서 1초로 잡았습니다.<br>
읽기는 배치 3번에 3.5초 → 128개 한 번이 1.2초 이상일 수 있으니, 1.5초로 하면 적재가 죽을 수 있어서 5초로 잡았구요.<br>
근데 이러면 단점이 있는데, 검색은 평소 180ms라서 5초는 느슨하죠. 그래도 둘로 나누지는 않았습니다.<br>
어차피 문제가 지금 당장은 없어서, 놔두도록 했습니다.

## 테스트인데 테스트 코드는 안짤거임

타임아웃 테스트 할건데 테스트 짜서 할거는 아닙니다. 저번에 말했다 싶히, Hello World에도 테스트를 만드는건 낭비잖아요?<br>
그냥 숫자만 바꿔볼겁니다.

```java
factory.setConnectTimeout(Duration.ofMillis(1));
factory.setReadTimeout(Duration.ofMillis(50));
```
이렇게 연결과 읽기의 시간을 수정을 해줄겁니다.<br>

그러면 어떻게 터질까요?<br>
저는 매번 ai 사용하다보면 자주보는 429나 503 오류가 나올거라고 생각을 했는데, 아니더라구요. 500이 뜹니다.<br>
429나 503은 서버가 보낸 답이고, 타임아웃은 서버가 답을 주지 않은거기 때문입니다.<br>
읽기는 터지는데 API 연결에서 안터지네요.

https://10.255.255.1/v1/embeddings 으로 보내보니깐, 이제서야 잘 터지네요.<br>
콘솔에서는 Connect timed out이 잘 보입니다.

## 예외 처리

다음은 이 예외를 어디서 잡아서 뭘로 바꿀지 정해야합니다.<br>
이걸 VoyageEmbeddingClient에서 잡을지, GlobalExceptionHandler에서 잡을지 어디에서 잡아야할까요?

제 생각은<br>
예외는 GlobalExceptionHandler 잡는게 맞지 않을까요?<br>
그러려고 따로 빼둔 예외 아닐까요?

아닙니다. 지금 올라오는 예외는 EmbeddingApiException 아니에요.<br>

역할을 둘로 쪼개야 합니다.<br>

바깥 예외를 우리 예외로 번역하고<br>
우리 예외를 화면으로 바꿔야 하는데 얘는 이미 있죠. GlobalExceptionHandler 이미 하고 있습니다.

ResourceAccessException을 catch한 다음, 50ms로 잡고 터트려봤는데 Whitelabel 500이 뜨네요. <br>
로그로 확인해보니깐, RestClientException이였습니다.<br>
RestClientException이 부모고, ResourceAccessException은 자식인데, catch는 자기랑 자식만 잡더라고요.<br> 
그래서 부모로 바꿔주니깐 제가 의도했던 GlobalExceptionHandler를 통해 "검색 중 문제가 발생했습니다" 라는 화면이 잘 보이네요.


## NPE

```java
         if (response == null || chunk.size() != response.data().size()) {
            log.debug("chunk_size={}", chunk.size());
            log.debug("data_size={}", response.data().size());
            throw new EmbeddingApiException("데이터 매칭 실패", null);
        }
```

이 코드에서도 문제가 있는데 이 참에 수정해봅시다.<br>
개수가 다르거나 null이거나 인데, response.data().size()를 불러버리면 무슨 일이 벌어질까요?<br>

NPE 오류가 나겠죠?<br>
개수가 다르면 괜찮지만, null일때 response.data().size() 이 친구가 실행이 되어버리 잖아요. 그러면 어떻게 해야할까요?<br>
간단합니다. 그냥 if문 2개로 나눠서 책임 지는 부분을 나눠버리면 됩니다.

```java
        if(response == null){
            log.debug("data_size is null");
            throw new EmbeddingApiException("데이터 null", null);
        }

        if (chunk.size() != response.data().size()) {
            log.debug("chunk_size={}", chunk.size());
            log.debug("data_size={}", response.data().size());
            throw new EmbeddingApiException("데이터 매칭 실패", null);
        }
```

```java
            throw new EmbeddingApiException("데이터 매칭 실패 / " + " chunk_size = " + chunk.size() + " data_size = " + response.data().size(), null);
```

근데 지금 로그가 debug일때 찍히는데, 이거를 Error로 올리려고 했습니다.<br>
왜냐하면 프로젝트 핵심 기능이니깐요.

근데 에러로 하기전에 그냥 예외메시지랑 합치기로 했습니다.<br>
두줄에서 한줄로 줄이려고요.

### 커밋

[306c005](https://github.com/JinsanPark/NewsFinder/commit/306c0055d57c0cc3f31bdbb77ae80f3253581c6f)<br>


## 다음 시간

지금 코드 보면, 상당히 중복된 부분이 많아 보입니다.<br>
embed와 embedBatch 보면,

```java
        EmbeddingRequest embeddingRequest;
        embeddingRequest = new EmbeddingRequest(List.of(text), voyageModel, inputType);
        EmbeddingResponse response;

        try {
            response = restClient.post()
                    .uri("https://api.voyageai.com/v1/embeddings")
                    .header("Authorization", "Bearer " + apiKey)
                    .body(embeddingRequest)
                    .retrieve()
                    .onStatus(HttpStatusCode::isError, (req, res) -> {
                        throw new EmbeddingApiException("Voyage 오류 : " + res.getStatusCode(), null);
                    })
                    .body(EmbeddingResponse.class);
        } catch (RestClientException e) {
            throw new EmbeddingApiException("Voyage 오류 : ", e);
        }
```

이 부분은 거의 같죠.<br>
다음 시간에는 간단하게 중복 제거를 한번 해 봅시다.

참고로, 검색이 지금 상당히 널널하죠. 검색은 180ms인데, 타임아웃은 5초니깐요. <br>
지금은 문제가 없지만, 나중에 검색이 3 ~ 4초 정도이면 검색은 성공인데, 매번 이렇게 기다려야 하고 에러도 안남겠죠.<br>
성공이니까요.

또한, 키별 락 때문에 같은 검색어는 요청을 기다리는데, 앞에서 쓴 시간 만큼 써버리면 뒤에서 기다려야 하잖아요.<br> 
이거는 나중에 RSS를 도입하면 해결해 보도록 하겠습니다. locks 크기랑 함께요.
