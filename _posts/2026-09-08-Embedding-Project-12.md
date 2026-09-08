---
title: synchronized으로 동시성 일부 해결하기
date: 2026-09-08 09:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---

## 지난 시간

지난번에 이제 L1에 동시 요청이 들어오면 안 고장나게 만든다고 했죠.<br>
그전에, 왜 그런지, 어떻게 박살나는지 부터 알아 봅시다.

## 왜?

LinkedHashMap로 Lru 만들었는데, 이거는 활짝 열린 방입니다.<br>
근데 일인실이에요. 한명만 들어와서 일하고 나가야 합니다. <br>
두명 들어오면 서로 할일 하다가 동선 겹치고 일 꼬이고 그래요.

동시 요청을 그러면 한번 보내고, 어떤 문제가 발생하는지 한번 봅시다.

저는 먼저 에이전트로 동시 요청 보내보니, 막상 별 문제가 없었습니다.<br>

```text
29.472  exec-6  total_ms=0.0304
29.475  exec-7  total_ms=0.0131
```

정확히는 동시에 요청이 아니라 3ms차이에다가 작업이 0.03ms라서 테스트를 만들기로 방향을 바꿨습니다.

## 8000을 기대했는데 1039

<details markdown="1">
<summary><b>기존 put 코드 (클릭하여 펼치기)</b></summary>

```java
@Test
    void 동시에_put하면_개수_안맞음() throws InterruptedException {
        int threadCount = 8;
        int perThread = 1000;

        LruCache<String, Integer> cache = new LruCache<>(8000);

        CountDownLatch start = new CountDownLatch(1);
        Thread[] threads = new Thread[threadCount];

        for (int t = 0; t < threadCount; t++) {

            final int threadNum = t;
            threads[t] = new Thread(() -> {
                try {
                    start.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

                for (int i = 0; i < perThread; i++) {
                    cache.put(String.valueOf('a' + i), i);
                }
            });
            threads[t].start();
        }

        start.countDown();

        for (Thread thread : threads){
            thread.join();
        }
        assertThat(cache).hasSize(8000);
    }
```

</details>

일단 코드를 짜봤습니다.<br>
의도한 대로 틀리긴 했는데, 뭔가 이상합니다.<br>

```text
Expected size: 8000 but was: 1039 in:
{"1072"=975, "198"=101, "223"=126, "311"=214}
java.lang.AssertionError: 
Expected size: 8000 but was: 1039 in:
{"1072"=975, "198"=101, "223"=126, "311"=214}
```

이런 오류요.<br>
맞았나 싶어서 물어보니, 이거는 동시에 들어가서 발생한 오류가 아니라고 합니다.<br>
그래서 쓰레드를 1개로 줄여보니깐, 그래도 비슷한 결과가 나와버립니다.

왜인가 알아보니,<br>
'a' + i 니깐 97부터 1096까지, 딱 1000개인데, 맵은 같은 키면 덮어쓰니깐 1000을 넘기면 안되죠.<br>
근데 서로 다른 키가 1000개뿐인데 1039개가 나옵니다.<br>
즉, 키가 유일하다는 규칙을 어긴거죠.

## 키가 다 겹쳤음
i는 각자 갖고 있지만 여덟 개가 똑같이 0 - > 999<br>
'a'에다가 더해주더라도, 어차피 반복되죠.

threadNum * perThread + i 라는 값을 사용했습니다.<br>
요 값을 쓰면 쓰레드 0은 0 ~ 999까지, 쓰레드 1은 1000 ~ 1999까지... 이런 식으로 사용 가능합니다.

이걸로 유일한 값을 만들기 위해서는 스레드마다 다른 값이 뭔지 찾아야한다는 교훈을 배웠네요.

## 테스트 돌릴 때마다 값이 바뀜

```java
                for (int i = 0; i < perThread; i++) {
                    cache.put(String.valueOf(threadNum * perThread + i), i);
                }
```
로 바꿔서 다시 돌리니 5057나왔다가, 5182도 나왔다가, 값이 계속 왔다갔다 합니다.<br>
put 한 번이 한 동작이 아니거든요

해시로 들어갈 값 계산 -> 그 칸이 비었는지 아닌지 읽기 -> 비었으면 노드 놓기 -> 사이즈 늘리기<br>
로 굴러가는데, 이게 동시에 들어오면 꼬이는 거에요.<br>
읽었는데 비었네? A와 B가 그 자리에 동시에 넣어버립니다.<br>
나중에 넣은 게 먼저 들어간거 덮어 씌우고 난리가 나겠죠?

근데 이 숫자는 측정값이 아닙니다.<br>
컴퓨터가 바뀌면 달라져요.<br>
그냥 얼마나 깨지는지가 아니라, 깨지나 안깨지나 확인하는 테스트입니다.

## synchronized 한 줄

```java
    @Override
    public synchronized V put(K key, V value) {
        return super.put(key, value);
    }
```

LruCache put에 이렇게 붙혀 주니깐 통과를 하네요.
그러면 이제 get을 봐야겠죠?

## get도 쓰기입니다
저번에 말했듯이 accessOrder=true 일때 읽으면 맨 뒤로 옮긴다고 했죠. false면 FIFO이고.<br>
LinkedHashMap으로 구현이 되있는데, 이 친구는 양방향으로 연결입니다.<br>
이걸 읽으면 알아서 맨 뒤로 보내서 축출 대기줄에서 맨 뒤로 순위를 밀어버려요.<br>
근데 동시에 일어나면 연결 상태가 꼬입니다.

<details markdown="1">
<summary><b>get 코드 (클릭하여 펼치기)</b></summary>

```java
@Test
    @Timeout(10)
    void 동시_get하면_링크_끊김() throws InterruptedException{
        LruCache<String, Integer> cache = new LruCache<>(1000);
        int threadCount = 8;

        for (int i = 0; i < 100; i++) {
            cache.put(String.valueOf(i), i);
        }

        CountDownLatch start = new CountDownLatch(1);
        Thread[] threads = new Thread[threadCount];

        for (int t = 0; t < threadCount; t++) {

            threads[t] = new Thread(() -> {
                try {
                    start.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

                for (int i = 0; i < 100; i++) {
                    cache.get(String.valueOf(i));
                }

            });
            threads[t].start();
        }

        start.countDown();

        for (Thread thread : threads){
            thread.join();
        }

        int count = 0;
        for (String key : cache.keySet()){
            count++;
        }

        assertThat(cache.size()).isEqualTo(100);
        assertThat(count).isEqualTo(100);

    }
```
</details>

이런식으로 코드를 짜봤습니다.<br>
한번 돌려보면 의도한대로, 오류가 잘 발생합니다.<br>

```text
org.opentest4j.AssertionFailedError: 
expected: 100
 but was: 2
	at org.jin.newsfinder.LruCacheTest.동시_get하면_링크_끊김(LruCacheTest.java:108)
```

size()는 100이라는데, 오류를 보면 1일때도 있고 2일때도 있네요.<br>
이거는 연결을 따라가니깐 1에서, 2에서 끊긴다는 이야기 입니다.<br>
그러면 아까 put처럼 synchronized를 붙혀주면 되겠죠.

```java
    @Override
    public synchronized V get(Object key){
        return super.get(key);
    }
```

위 코드를 lruCache에 추가하면, 문제 없이 통과 합니다.


## 아직 안전하지 않음

containsKey는 그대로라서, 확인하고 꺼내는 사이에 밀려나면 L1 히트인데 null 값이 돌아옵니다.<br>
그래서, 이거를 만져야 진짜 동시성이 해결되기에, 다음시간에는 서비스 코드쪽을 한번 건들여봅시다.