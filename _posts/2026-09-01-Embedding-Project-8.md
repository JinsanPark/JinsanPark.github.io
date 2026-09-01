---
title: 테스트 짜다가 코드를 고치기
date: 2026-09-01 11:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---

## 지난 시간

지난 시간에 캐시 히트 테스트를 짜봤습니다.<br>
통과했는데 아무것도 안 지키고 있었고, 이리저리 코드를 박살내고 나서야 무언가 잘못된걸 알았습니다.

이번에는 반대쪽인 캐시 미스를 볼 차례입니다.<br>
근데 이거 짜다가 결국 QueryVectorService 코드를 고치게 됐습니다. 그 얘기까지 해보겠습니다.

## 캐시 미스

getVector의 else를 봅시다.

```java
} else {
    normTovector = embeddingClient.embedQuery(norm);
    QueryVectorCache cache = new QueryVectorCache(norm, voyageModel, normTovector, LocalDateTime.now());
    queryVectorCacheRepository.save(cache);
}
```

이걸 글로 설명하면<br>
캐시에 값 없음 -> API 호출 + 받은 값 리턴 -> 결과 저장.

확인할게 2개인데, API를 부르는지, 그리고 저장을 하는지.

저장을 하는지 안하는지가 제일 중요합니다.<br>
API를 부르는지 아닌지는 바로 테스트에서 확인이 가능한데, save를 빼먹어도 앱은 잘 돌아가고 결과도 잘 나오거든요.<br>
근데 저장을 안하면, 캐시가 영영 안 채워져서 매번 미스가 납니다. 캐시를 만든 의미가 통째로 사라지는데 화면으로는 알 수가 없는 찐빠가 발생합니다.

## when? verify?

getVector가 일꾼들을 부르는 호출이 딱 3개인데, 규칙을 정리하자면,

| 호출 | 하는 일 | 쓰는 것 |
|---|---|---|
| findByNormalizedQueryAndModel | 값을 받아온다 | when |
| embedQuery | 값을 받아온다 | when |
| save | 시키기만 한다 | verify |

값을 받아오는 호출은 when으로 일감을 줍니다. 안 주면 일꾼이 빈손으로 오니깐요.<br>
반대로 save는 리턴값을 안 쓰니깐 일감을 줄 이유가 없고, 일을 했나 안 했나만 확인하면 되겠죠.


## 캐시 미스 짜보기.

처음 짠 코드입니다.

```java
@Test
void 캐시미스시_API_호출_및_저장() {

    EmbeddingClient embeddingClient = mock(EmbeddingClient.class);
    QueryVectorCacheRepository queryVectorCacheRepository = mock(QueryVectorCacheRepository.class);

    when(queryVectorCacheRepository.findByNormalizedQueryAndModel(any(), any()))
            .thenReturn(Optional.of(null));                                      // 1번

    QueryVectorService queryVectorService = new QueryVectorService(embeddingClient, queryVectorCacheRepository);

    float[] result = queryVectorService.getVector("테스트");

    when(embeddingClient.embedQuery(any())).thenReturn(new float[]{0.1f, 0.1f}); // 2번

    assertThat(result).containsExactly(0.1f, 0.1f);
    verify(queryVectorCacheRepository).save(any());
}
```


### 1번, Optional.of(null)

캐시가 비어있는 상황을 만들려고 null을 넣었는데 NPE가 났습니다.

지난 6편에서 Optional을 값과 있음/없음을 같이 전달한다고 했었는데,<br>
그런데 of()는 있음 전용이라 이 값은 확실히 있다고 선언하는 거라, null을 넣으면 있다며? 하고 바로 터집니다.<br>
애초에 Optional이 null을 몰아내려고 만든 물건인데 null을 담아주면 앞뒤가 안 맞는 상황이 되는거죠.

그래서, 진짜로 없음을 만들려면 파라미터를 아예 안 받는 Optional.empty()를 써야 했습니다.

### 2번, when이 실행 아래에 있음

이건 좀 어이없는 실수인데, when을 getVector 호출 뒤에 썼습니다.

일꾼한테 일감을 퇴근하고 나서 준거죠.<br>
getVector가 돌아가는 그 순간에 일꾼은 아직 일감이 없어서 빈손으로 오고, result는 null이 됩니다.

when으로 시작하는 줄은 전부 실행보다 위에 있어야 합니다.

두 개 고치니 통과했습니다.

참고로 0.1f, 0.1f라는 값은 별 의미가 없습니다. 진짜 Voyage API가 뭘 돌려줄지랑도 별 상관없고요.<br>
중요한건 캐시 히트 테스트에서 쓴 1.0f, 0.0f랑 다르다는 것 뿐입니다.<br>
result에서 0.1f, 0.1f가 나왔으면 API를 거쳐서 나왔다는 증거가 됩니다.

## 저장은 했는데, 뭘 저장함?

```java
verify(queryVectorCacheRepository).save(any());
```

이 줄은 save가 불렸다까지만 봅니다. any()가 뭐가 들어오든 다 통과시키거든요.

그런데 이런 버그를 생각해봅시다.

```java
QueryVectorCache cache = new QueryVectorCache(query, voyageModel, normTovector, LocalDateTime.now());
//                                            norm이 아니라 원본 query
```

한 글자 차이로 정규화 안 된 문자열이 저장됩니다.<br>
저장은 됐으니 위 테스트는 통과하고요.

근데 다음번에 조회할 때는 정규화된 문자열로 찾잖아요?<br>
" AI 투자 "로 저장해놓고 "ai 투자"로 찾으니 영영 안 맞습니다. 캐시 히트가 절대 안 나는 상태가 되는거죠.<br>
6편에서 공들여 만든 정규화가 통째로 무의미해집니다.

그래서 저장했다가 아니라 뭘 저장했는지를 봐야 합니다.

## ArgumentCaptor

save에 넘어간 객체를 붙잡는 도구가 있습니다.

```java
ArgumentCaptor<QueryVectorCache> captor = ArgumentCaptor.forClass(QueryVectorCache.class);  // 그물 준비
verify(queryVectorCacheRepository).save(captor.capture());                                  // any() 자리에 그물
QueryVectorCache saved = captor.getValue();                                                 // 잡힌거 꺼내기
```

capture()가 any() 자리를 대신합니다.<br>
여전히 아무거나 통과시키면서, 지나가는 놈을 몰래 기록해두는거죠.<br>
이제 saved는 서비스가 실제로 save에 넘긴 그 객체입니다.

## 다이어트 성공?

이제 saved.getNormalizedQuery()로 저장된 문자열을 볼 수 있는데, 문제가 하나 있었습니다.

지금 테스트는 getVector("테스트")로 부르고 있거든요.<br>
저번에 정규화를 할때 어떻게 되도록 했냐면,

```
NFC 정규화 -> 앞뒤 공백 제거 -> 연속 공백 한 칸으로 -> 소문자
```

"테스트"에는 자를 공백도, 줄일 연속 공백도, 소문자로 바꿀 대문자도 없습니다.<br>
정규화를 하든 안 하든 결과가 똑같아요.

이러면 저장된 값이 "테스트"인걸 확인해봤자 아무 의미가 없습니다.<br>
다이어트 선언하고 70kg에서 70kg으로 다이어트 성공한 셈이죠.

그래서 입력을 좀 분별 가능한걸로 바꿨습니다.
```
" QVC  API  테스트 "   ->   "qvc api 테스트"
```

앞뒤 공백, 연속 공백, 대소문자 세가지가 한번에 검증됩니다.

## 굳이 불편하게 왜?

아니 정규화 메소드 있는데 왜 안씀? 이라고 물으실 수 있겠는데,

```java
assertThat(saved.getNormalizedQuery()).isEqualTo(normalizeQuery(" QVC  API  테스트 "));
```

안 됩니다. 양쪽이 같은 메서드를 쓰니깐요.

normalizeQuery에서 실수로 toLowerCase가 빠졌다고 치면,<br>
왼쪽도 소문자가 안 되고, 오른쪽도 소문자가 안 됩니다. 그래서 똑같고, 통과하겠죠.<br>
버그가 있는데 초록불이에요. 지난 시간의 embedDocuments랑 똑같은 함정입니다.

기대값은 코드가 아니라 사람 머리에서 나와야 합니다.

```java
assertThat(saved.getNormalizedQuery()).isEqualTo("qvc api 테스트");
```

하드코딩이 좀 찜찜한데, 테스트에서는 이게 맞다고 합니다.<br>
테스트는 코드가 이렇게 동작한다를 적는 곳이 아니라, 코드가 이렇게 동작해야 한다를 알려주는 곳이니깐요.

생각해보면 1편에서 코사인 유사도 테스트 할 때도 비슷했죠.<br>
코드를 다 짜고 테스트를 짰는데, 1024차원은 손으로 못 푸니깐 2차원으로 줄인 다음에 테스트 했었죠.<br>
같은 방향이면 1, 다른 방향이면 -1.<br>
그거랑 같은 방식입니다. 다만 normalizeQuery가 private이라 직접 못 부르니 captor로 잡은 객체를 통해 간접적으로 볼 뿐이죠.

## 그런데 모델 이름이 null?

정규화까지 통과했으니 모델 이름도 확인해보기로 했습니다.

```java
assertThat(saved.getModel()).isEqualTo("아무모델");
```

근데 값이 null이네요?

```java
@Value("${voyage.model}")
private String voyageModel;
```

@Value는 스프링이 빈을 만들면서 값을 꽂아주는 건데,<br>
테스트에서는 제가 new로 직접 만들었잖아요? 스프링은 이 객체가 있는지도 모르겠죠.<br>
아무도 안 꽂아줬으니 String 필드 초기값인 null이 그대로 남은거죠.

지금까지 안 터진건 조회할 때 any()로 뭉개고 있었기 때문입니다.

여기서 같은 클래스 안에 있는 세 개가 갈리는데,

| 필드 | 어떻게 들어오나 | 테스트에서 |
|---|---|---|
| embeddingClient | 생성자 | 내가 넣으면 됨 |
| queryVectorCacheRepository | 생성자 | 내가 넣으면 됨 |
| voyageModel | @Value 필드 주입 | 손댈 방법 없음 |

앞의 둘은 문제가 없었습니다. 생성자로 받으니깐 제가 직접 넣을 수 있었거든요.<br>
voyageModel만 스프링한테 의존하고 있었던겁니다.

## 생성자 주입으로

그래서 그냥 voyageModel도 생성자 주입으로 바꿨습니다.<br>
embeddingClient랑 repository는 생성자로 받는데 voyageModel만 다른 방식이었어요.

사실 기존 코드를 안 건드리는 방법도 있긴 있습니다. ReflectionTestUtils라고, private 필드에 강제로 값을 꽂아넣는 도구가 있는데,<br>
근데 그러면 필드 이름을 문자열로 적어야 해서, 나중에 이름 바꾸면 컴파일러가 못 잡아내고 실행할 때 터집니다.

사실 다른 이유가 더 중요한데, 테스트가 불편하다고 알려준 지점이 실제로 설계가 약한 지점이었기 때문이죠.

<details markdown="1">
<summary><b>테스트는 어떻게 설계의 결함을 알려주는가 (클릭하여 펼치기)</b></summary>

현 코드에서는 QueryVectorService는 스프링 하나에서만 사용합니다.
즉, 스프링에 맞춰져 있어서 문제가 없어 보이죠.

반면, 이걸 테스트를 끌고 온다면? 사용층이 하나 더 생기는거죠. 스프링이랑 테스트로.
테스트에서 QueryVectorService가 스프링에 종속되어 있다는걸 발견한거죠.

```java
@Value("${voyage.model}")
private String voyageModel;
// 스프링 없이는 불완전
```

```java
public QueryVectorService(..., @Value("${voyage.model}") String voyageModel)
// 생성하려면 voyageModel이 필요한데, 누가 주든 상관 X
```

추후에 확장하다 보면 발생 할 수 있는 문제를 테스트에서 드러낸겁니다.<br>
즉, 테스트가 불편하면, 그 클래스를 다른 데서 쓰기도 불편하다. 이겁니다.

근데, 늘 그런건 아닙니다.
저 처럼 테스트 도구를 잘 몰라서 불편 할 수도 있겠죠.
또, 갑과 을이 뒤바뀌는, 테스트 코드를 위해서 기존 코드를 뜯어 고치는 경우도 발생 할 수 있습니다.

### 그러면 어떻게?

그래서, 위 처럼 불편한 신호를 받았으면 먼저 확인을 해봐야 합니다.
내가 도구를 모르는 원시인인지, 아니면 진짜 설계 문제인지.

이번 건은, 같은 클래스 안에서 비교 대상이 명확하게 있었기 때문에 쉽게 찾을 수 있었습니다.

| | embeddingClient | repository | voyageModel |
|---|---|---|---|
| 이 클래스가 굴러가는 데 필요한가 | 필요 | 필요 | 필요 |
| 앱 켤 때 정해지고 안 바뀌나 | 그렇다 | 그렇다 | 그렇다 |
| 스프링이 채워주나 | 채워준다 | 채워준다 | 채워준다 |
| **주입 방식** | **생성자** | **생성자** | **필드** |
| **테스트에서** | **문제없음** | **문제없음** | **막힘** |

보시면 공통점이 있는데, 차이점이 있죠.
이거 때문에 쉽게 찾을 수 있었습니다.

</details>

## 고친 코드

어노테이션은 필드뿐 아니라 생성자 파라미터에도 붙습니다.

```java
private final String voyageModel;                    // @Value 떼고 final

public QueryVectorService(EmbeddingClient embeddingClient,
                          QueryVectorCacheRepository queryVectorCacheRepository,
                          @Value("${voyage.model}") String voyageModel) {
    this.embeddingClient = embeddingClient;
    this.queryVectorCacheRepository = queryVectorCacheRepository;
    this.voyageModel = voyageModel;
}
```

이러면 세 개가 전부 같은 방식이 됩니다.<br>
스프링은 빈을 만들 때 이걸 보고 설정 파일에서 가져오라는 거구나 하고 채워주고, 테스트에서는 그냥 문자열을 넘기면 되는거고요.

VoyageEmbeddingClient도 모델 이름을 같은 식으로 받고 있어서 같이 고쳤습니다.<br>
apiKey는 아직 필드 주입인데, 그 클래스를 테스트할 때 연습겸 해서 같이 옮기려고 남겨놨습니다.

<details markdown="1">
<summary><b>AI가 추천하는 @InjectMocks는 여기서 안 됩니다 (클릭하여 펼치기)</b></summary>

중복을 줄이는 방법을 물어보면 @ExtendWith(MockitoExtension.class) + @Mock + @InjectMocks 조합을 추천받을 가능성이 높습니다.

그런데 이 클래스에는 안 맞습니다.<br>
생성자에 String voyageModel이 있는데 @InjectMocks는 String을 못 채워서 null을 넣습니다.<br>
방금 만든 모델 이름 검증이 도로 깨지는거죠.

그래서 @BeforeEach로 직접 만드는 쪽을 택했습니다.

</details>

## 정리하고 나니

테스트 두 개에 mock 만드는 코드가 똑같이 들어가 있길래 @BeforeEach로 뺐습니다.<br>
@BeforeEach가 붙은 메서드는 각 테스트 직전에 매번 실행됩니다.

필드로 올리면 테스트끼리 값이 섞이지 않냐 싶은데, 안 섞입니다.<br>
JUnit5는 테스트 메서드마다 클래스 인스턴스를 새로 만들거든요. 게다가 @BeforeEach가 매번 mock을 새로 할당하니 이전 테스트의 when 설정도 사라지고요.


## 최종 코드

```java
public class QueryVectorServiceTest {

    private static final String MODEL = "테스트 모델";
    private EmbeddingClient embeddingClient;
    private QueryVectorCacheRepository queryVectorCacheRepository;
    private QueryVectorService queryVectorService;

    @BeforeEach
    void setUp() {
        embeddingClient = mock(EmbeddingClient.class);
        queryVectorCacheRepository = mock(QueryVectorCacheRepository.class);
        queryVectorService = new QueryVectorService(embeddingClient, queryVectorCacheRepository, MODEL);
    }

    @Test
    void 캐시히트시_API_호출_안함() {
        QueryVectorCache cached = new QueryVectorCache("테스트", "모델1", new float[]{1.0f, 0.0f}, LocalDateTime.now());
        when(queryVectorCacheRepository.findByNormalizedQueryAndModel(any(), any())).thenReturn(Optional.of(cached));

        float[] result = queryVectorService.getVector("테스트");

        assertThat(result).containsExactly(1.0f, 0.0f);
        verify(embeddingClient, never()).embedQuery(any());
    }

    @Test
    void 캐시미스시_API_호출_및_저장() {
        ArgumentCaptor<QueryVectorCache> argumentCaptor = ArgumentCaptor.forClass(QueryVectorCache.class);
        when(queryVectorCacheRepository.findByNormalizedQueryAndModel(any(), any())).thenReturn(Optional.empty());
        when(embeddingClient.embedQuery(any())).thenReturn(new float[]{0.1f, 0.1f});

        float[] result = queryVectorService.getVector(" QVC  API  테스트 ");

        assertThat(result).containsExactly(0.1f, 0.1f);
        verify(queryVectorCacheRepository).save(argumentCaptor.capture());

        QueryVectorCache saved = argumentCaptor.getValue();
        assertThat(saved.getNormalizedQuery()).isEqualTo("qvc api 테스트");
        assertThat(saved.getModel()).isEqualTo(MODEL);
    }
}
```

이 두 개가 잡아내는 버그입니다.

| 망가뜨리는 방법 | 어느 테스트가 잡나 |
|---|---|
| 캐시 조회를 아예 안 함 | 캐시 히트 |
| 두 분기를 뒤바꿈 | 둘 다 |
| save를 빼먹음 | 캐시 미스 |
| 정규화 안 된 문자열을 저장 | 캐시 미스 |
| 모델 이름을 엉뚱하게 저장 | 캐시 미스 |

전부 앱은 멀쩡히 돌아가는데 API 요금만 새는 종류입니다.

## 배운 점

- 테스트를 연습해볼 겸 시작했는데, 다 하고 나니 프로덕션 코드가 바뀌어 있더라고요. 테스트하기 불편하다는게 설계가 약하다는 신호일 때가 있는거 같네요. ReflectionTestUtils로 우회할 수도 있다고 하는데, 그러면 그 신호를 못 본 척 하는 셈이었고요.
- 기대값은 검증할 코드로 만들면 안 됩니다. 양쪽이 같이 틀려서 무조건 통과하거든요. 손으로 계산해서 박아넣는 하드코딩이 여기서는 오히려 맞습니다.

## 아직 한참 남음

아직 많은게 남았습니다.

캐시 히트 테스트를 보면 findByNormalizedQueryAndModel(any(), any())라서 조회할 때 뭘 넘겼는지는 안 따져요.<br>
저장은 정규화를 검증했는데 조회는 안 한거죠. 저장은 "qvc api 테스트"로 하고 조회는 " QVC API 테스트 "로 하면 캐시가 영영 안 맞을테니깐요.

그리고 캐시 미스 테스트가 지금 네가지를 검증하고 있습니다. 리턴값, save 호출, 정규화, 모델 이름.<br>
근데 테스트 이름에는 정규화 얘기가 없어요. 이름이랑 내용이 어긋나 있는겁니다.

Voyage API가 터졌을 때 예외가 밖으로 잘 나가는지도 아직 안 봤고요.

차차 해결해 나가도록 합시다.

## 다음 시간

이제 진짜로 메모리 캐시를 얹을 차례입니다.<br>
메모리 -> 미스 -> DB -> 미스 -> Voyage API 이 구조요.

LRU라는걸 직접 만들어 볼까 하는데, 스프링에 붙이기 전에 순수 자바 클래스로 먼저 만들고 테스트부터 짜보려고 합니다.
