---
title: 테스트는 통과했는데 통과가 아니다?
date: 2026-08-31 12:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---

## 지난 시간

지난 시간에 벡터를 재활용 하려고, DB에 캐시로 활용 할 수 있게 저장을 했죠.
원래라면 메모리 캐시를 얹을 차례이긴한데,
저번에 코사인 유사도 테스트를 짜봤는데 익숙하지가 않아서 테스트를 좀 연습해볼 겸 정리해볼까 합니다.

## 그래서 테스트를 왜함?

테스트를 왜 할까요? 어차피 그냥 서버 키고 localhost에서 할 수 있는데, 굳이 귀찮게 테스트 코드를 짜야 할까요? 제 코드에서 차이가 나는 지점은 4군데가 있겠네요.

### 1. 다시 확인하기 귀찮음
로컬로 확인 하는 방법은, 앱 실행 - 검색 - 결과 확인.<br>
이런 구조로 돌아갑니다. 문제는 코드를 고칠 때마다 매번 확인을 해줘야 하죠.<br>
정규화 코드를 손봤는데, 매번 판단을 해야 합니다. `캐시 조회는? 저장은?` 이게 많아 지고 귀찮아 지면 건너뛰겠죠.<br>
허나, 자동 테스트는 이 판단 자체가 필요 없습니다. 몇 초면 전부 다 돌아가니깐요.

### 2. 캐시는 눈으로 안 보임.
그리고, 제 프로젝트 같은 경우는, 캐시가 눈에 보이지 않습니다.<br>
API를 불러서 나온 결과인지, 캐시에서 꺼낸 결과인지 판단이 서지 않습니다.<br>
만약 캐시에 이상한 값들이 잔뜩 들어갔다고 가정하면, 캐시 히트는 절대 안나던가 이상하게 나겠죠.<br>
이걸 확인하려면 로그를 찍어서 눈으로 확인해야하는데, 이게 결국 일일히 해야하는 테스트거든요.

### 3. 만들기 어려움.
캐시 미스를 로컬에서 만들려면 매번 DB를 비우거나, 새 검색어를 찾아야 합니다.<br>
근데 만약 Voyage API가 먹통이면? 이거는 로컬에서는 해결 할 수 있는 방법이 없습니다.<br>
API 서버 멀쩡해질때까지 손가락 빨면서 기다리던가, 아니면 테스트를 만들던가죠.

### 4. 테스트 이름으로 알 수 있음.
저는 테스트 이름을 이렇게 만들어봤고요.

`캐시히트시_API_호출_안함()`
`캐시미스시_API_호출_및_저장()`

이름만 보고도, 이 2개가 뭘 하는건지 바로 알 수 있겠죠?
주석은 낡아도 아무도 모르지만, 테스트는 낡으면 빨개져서 티가 납니다.

## 다만
모든 코드를 테스트 할 필요는 없습니다. 테스트도 코드잖아요.<br> 
뭐 Hello World에도 테스트 코드를 만들겁니까? <br>
그래서 이 무엇을 테스트 코드로 할지 기준점을 세우는 게 중요한거 같습니다.<br>
 저 같은 경우에는 반복적이고, 눈으로 하나하나 확인 하기 힘들걸 위주로 할까 합니다.

## 그럼 진짜 QueryVectorService 테스트 시작.

저는 저번에 코사인 유사도 테스트를 할때는, 어느정도 이해가 갔습니다.<br>
내가 넣은 값이, 정해진 정답과 같으냐를 푸는, 그냥 새로운 문법으로 쉬운 알고리즘 문제를 푸는거 같았으니깐요. 

허나 이번에는 조금 달랐습니다. 처음에는 도통 뭘 검증해야하는지 감이 도통 안오더라고요.

```java
@Service
public class QueryVectorService {

    private final EmbeddingClient embeddingClient;
    private final QueryVectorCacheRepository queryVectorCacheRepository;

    @Value("${voyage.model}")
    private String voyageModel;

    public float[] getVector(String query) {
        String norm = normalizeQuery(query);
        Optional<QueryVectorCache> cached =
                queryVectorCacheRepository.findByNormalizedQueryAndModel(norm, voyageModel);
        float[] normTovector;

        if (cached.isPresent()) {
            QueryVectorCache cache = cached.get();
            return cache.getEmbedding();
        } else {
            normTovector = embeddingClient.embedQuery(norm);
            QueryVectorCache cache = new QueryVectorCache(norm, voyageModel, normTovector, LocalDateTime.now());
            queryVectorCacheRepository.save(cache);
        }
        return normTovector;
    }
}
```

여기서 getVector가 뭘 받는지 곰곰히 생각해보고 정리해 봤는데,

| 값 | 출처 |
|---|---|
| `norm` | 자기가 만듦 |
| `cached` | repository한테 받음 |
| `normTovector` | embeddingClient한테 받음 |
| **리턴하는 `float[]`** | 결국 남한테 받은 걸 그대로 넘김 |

대강 이렇습니다.

getVector("테스트")가 {1.0f, 0.0f}를 돌려주는지 확인한다고 합시다.<br>
근데 이 값은, 제가 테스트 코드한테 시킨 값이잖아요? 답을 정해주고, 이 답이 맞는지 확인하는 셈이죠.

애초에 이 클래스가 하는 일은 계산이 아니죠.<br>
코사인 유사도는 위 처럼 집어넣은 값이 정해진 값으로 나오냐? 이것만 테스트 하면 끝이였죠.<br>
이 테스트는 일을 지정된 곳에 잘 시켰냐를 봐야했습니다.

이 클래스가 하는건
1. 입력 받은 검색어를 정규화
2. 캐시를 먼저 본다 - 있으면 API 호출 X
3. 없으면 API 호출해서 받아온걸 DB에 저장한다.

## 첫 테스트 작성

그러면 제 코드에서는 테스트를 몇개를 만들어야 할까요?<br>
getVector는 목적이 2개죠. if문에서 캐시 히트, 캐시 미스를 검증하죠.<br>
즉, 테스트 코드도 2개입니다. 뭘 검증해야할지 감이 안왔는데, 머리를 싸맬게 아니라, 정답이 그냥 코드 안에 있었습니다.

그러면, 들어가기 전에, 간단하게 문법부터 살피고 가봅시다.
```java
mock() // 가짜 일꾼
when(...).thenReturn(...) // 가짜 일꾼에게 일감 주기
verify(...) // 일꾼이 일 했나 안했나 확인.
```

가짜를 왜 쓰냐하면, API를 쓰면 작지만 돈이 나가고, 또 API가 먹통이면 테스트를 할 수 없어지니깐요.<br>
Voyage 무료 토큰 많이 주지만, 그래도 아낄건 아껴야죠.<br>
그러면 캐시 히트부터 한번 봅시다.

```java
@Test
void 캐시에_있으면_API_호출X() {

    EmbeddingClient embeddingClient = mock(EmbeddingClient.class);
    QueryVectorCacheRepository queryVectorCacheRepository = mock(QueryVectorCacheRepository.class);

    QueryVectorCache cached = new QueryVectorCache("테스트", "모델1", new float[]{1.0f, 0.0f}, LocalDateTime.now());

    when(queryVectorCacheRepository.findByNormalizedQueryAndModel(any(), any()))
            .thenReturn(Optional.of(cached));

    QueryVectorService queryVectorService = new QueryVectorService(embeddingClient, queryVectorCacheRepository);

    float[] result = queryVectorService.getVector("테스트");

    assertThat(result).containsExactly(1.0f, 0.0f);
    verify(embeddingClient, never()).embedDocuments(any());
}
```

코드를 작성을 했고, 그런데 문제가 있었습니다.<br>
`embedDocuments`는 getVector가 쓰지 않는 메서드여서, 지금 테스트는 안 쓰는 메소드가 안 불렸어? 를 검증하는게 되어버린거죠.

근데 통과되니깐, 신나서 AI에게 맞게 짰냐고 물어봤는데, 네. 틀렸답니다. 그러면서 기준을 하나 배웠는데,

- 이 코드에 일부러 버그를 심으면, 내 테스트가 빨개지나?

이 기준으로 다시 보니 답이 나왔습니다.
캐시 로직을 통째로 지워도, `if`문을 뒤집어도 제 테스트는 초록불입니다.<br>
안 부르는 메서드가 안 불렸는지 확인하고 있으니, 당연한 결과죠.

그래서 `embedDocuments`를 `embedQuery`로 고쳤습니다.

## 이번엔 진짜 빨개지나?

고치고 다시 돌리니 통과합니다. 근데 아까 엉뚱한 테스트 코드에서도 통과는 했었잖아요.

그래서 일부러 코드를 뭉개봤습니다.<br>
테스트 코드는 그대로 두고, `QueryVectorService` 쪽만 코드를 바꿔봤습니다.

QueryVectorService의 getVector()에서 캐시 히트 분기인 if문을

```java
if (cached.isPresent())   ->   if (false)

```

캐시 히트를 아예 안타도록 바꾸어 봤습니다.<br>

## 잠깐 퀴즈

그리고 제 테스트에서 검증하는 부분은 이 두 줄이었죠.

​```java
assertThat(result).containsExactly(1.0f, 0.0f);      // A
verify(embeddingClient, never()).embedQuery(any());  // B
​```

A랑 B 중에 어디가 빨개질지 잠깐 생각을 해볼까요?

저는 B라고 예상했습니다.<br>
A는 result 값만 보는 거고, result에는 제가 캐시에 넣어둔 {1.0f, 0.0f}가 그대로 들어있을 테니까요.<br>
캐시를 안 탔으니 B가 터지겠죠.

근데 A가 터졌습니다.

## 엥?

if (false)로 바꿨으니 else로 빠져서 embedQuery가 불립니다.<br>
그런데 저는 이 일꾼 한테 일감을 안 줬습니다. never()로 검증할 거니깐 줄 이유가 없었으니깐요.<br>
일감 없는 일꾼한테 일을 시키면 뭘 들고 올까요? 빈손으로 옵니다. null이요.<br>
그러니깐 result는 제가 준비한 {1.0f, 0.0f}가 아니라 null이었고, A가 터진 겁니다.

여기가 바로 제가 헤맨 부분이고요.

result는 제가 준비한 값이 아니라, getVector가 돌려준 값이더라고요.<br>
캐시 히트면 cache.getEmbedding()이 나오고, 미스면 embedQuery가 준 값이 나옵니다.<br>
제가 if (false)로 미스 쪽으로 몰아버렸으니, 캐시에 넣어둔 값이 나올 리가 없었죠.

## 초록색은 기분이 좋다

이번에 배운 건 이겁니다.<br>
테스트가 통과했다고 해서, 그 테스트가 뭘 보장을 하지는 않습니다.

제 첫 테스트는 분명 초록불이었습니다. 근데 캐시를 통째로 지워도 초록불이였죠?
실상 아무것도 안 지키고 있었던 겁니다.

더 무서운 건, 초록불이라서 못 알아챘다는 거예요.<br>
빨간 줄이었으면 바로 봤을 텐데, 통과하니깐 잘 짠 줄 알고 신나서 넘어갈 뻔했습니다.

그래서 테스트를 짜고 나면 한 번쯤 뭉개봐야 하는 것 같습니다.<br>
일부러 코드 박살내보고, 빨간줄 뜨나 안뜨나 확인해야하겠죠.<br>
안 빨개지면 그 테스트는 그냥 초록불만 켜주는, 잠시 기분이 좋아지는 장치입니다.

## 다음 시간

이제 캐시 미스 쪽을 봐야겠죠. API를 부르고, 받아온 걸 저장하는 부분이요.

그런데 이걸 파다 보니 좀 이상한 데로 흘러갔습니다.<br>
테스트를 제대로 짜려니깐 QueryVectorService 코드 자체를 고쳐야 하더라고요.

다음 시간에 이어서 해보도록 합시다.