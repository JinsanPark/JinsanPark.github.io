---
title: 간단한 LRU 코드
date: 2026-09-02 09:00:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, test, study]
---

## 지난 시간

지난 시간에 캐시 히트/미스를 짜고 그거를 테스트 해봤죠.<br>
이번 시간에는 메모리 캐시의 알맹이만 먼저 만들어봅시다. 붙이는 건 다음 시간에요.

## LRU?

LRU는 Least Recently Used라고
가장 오래 안본거 부터 처리하는 방법중 하나 입니다.

이거 말고도 여러가지가 있는데 FIFO, LFU 같은것들도 있습니다.

그리고 왜 QueryVectorService에 같이 안쓰고 따로 빼나면,
그래야 문제가 생기면, 범인 찾기가 쉽잖아요.

## 예측해 봅시다.

용량이 3인 LRU 캐시가 있다고 칩시다.

`put(A) → put(B) → put(C) → get(A) → put(D)`

위 코드를 순서대로 실행하면 뭐가 남고, 뭐가 빠질까요?

A<br>
A B<br>
A B C<br>
B C A<br>
C A D

로 B가 빠지겠죠.
가장 오래 안쓴거를 버리는 거니깐요. B를 버린거죠.

## 구현

구현은 간단합니다.
다만, HashMap에는 순서가 없어서 LinkedHashMap을 써야합니다.<br>
그리고, LinkedHashMap은 기본값이 넣은 순서(FIFO)이기 때문에, LRU가 되려면 두 가지를 손을 써야 합니다.

### 1. 조회도 순서에 반영시키기.
- 생성자에 accessOrder라는 세 번째 인자가 있는데, 여기에 true를 주면 get할때 마다 그 항목이 다시 뒤로 갑니다.<br> 생각해보면 그래야 말이 되는게, 캐시에서 압도적으로 많이 일어 나는건 쓰기가 아니라 읽기죠. <br> 대부분 get으로 꺼내쓰지 put하는 경우는 그다지 많지는 않겠죠. 아니면 캐시를 잘못 만들었거나요.

### 2. 언제 버릴지.
- removeEldestEntry라는 메서드를 오버라이드. <br> true를 리턴하면 가장 오래된 항목이 빠지게 됩니다. 사이즈가 다 찼을때 버리면 되겠죠 이제는?

코드로 한번 봅시다.

```java
public class LruCache<K, V> extends LinkedHashMap<K, V> {

    private final int capacity;

    public LruCache(int capacity) {
        super(16, 0.75f, true); // 초기 크기, 언제 배열을 정할지 결정하는 비율, false == FIFO, true == LRU
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

여기서 주의할게, removeEldestEntry가 넣은 바로 직후에 불리기 때문에, 사이즈에 주의해야 합니다.<br>
용량이 3이라고 하면, a b c를 넣고 d를 넣는다고 가정했을때, d를 넣고 바로 불러오면 사이즈는 얼마죠?<br>
4가 되잖아요? 그래서 size() > capacity; 에서 >= 가 아니라 > 입니다.


## 테스트 짜보기

그러면 만들었으니 동작하는지 테스트를 한번 굴려보면 좋겠죠?

```java
package org.jin.newsfinder;

import org.junit.jupiter.api.Test;

import static org.assertj.core.api.AssertionsForInterfaceTypes.assertThat;

public class LruCacheTest {

    @Test
    void LRU_동작확인() {

        //given
        LruCache<String, String> cache = new LruCache<>(3);
        cache.put("A", "A");
        cache.put("B", "B");
        cache.put("C", "C");

        //when
        cache.put("D", "D");

        //then
        assertThat(cache.keySet()).containsExactly("B", "C", "D");

    }
}
```

테스트를 돌려보니 잘 돌아갑니다.

그리고 일부러 코드를 한번 고장내봤습니다.<br>
`super(16, 0.75f, true);` 을 `super(16, 0.75f, false);`로 바꿨습니다.

그리고 예측을 한번 해볼까요?

예측을 해보니, 테스트를 잘못 만들었죠?<br>
A B C를 순서대로 착착착 넣고, D를 넣으면 LRU에서도 B C D가 나옵니다.<br>
FIFO에서도 B C D가 나오고요.
즉, LRU에서도, FIFO에서도 둘다 통과해요.

```java
    @Test
    void LRU_동작확인() {

        //given
        LruCache<String, String> cache = new LruCache<>(3);
        cache.put("A", "A");
        cache.put("B", "B");
        cache.put("C", "C");
        cache.put("A", "A");

        //when
        cache.put("D", "D");

        //then
        assertThat(cache.keySet()).containsExactly("C", "A", "D");

    }
```


그래서 이렇게 바꿔줬습니다.

다시 true로 되돌린다음 돌리니 통과네요.<br>
C A D 요구한 그대로 나왔습니다.

이번에는 false로 바꾸고, 그러면 FIFO죠. 따라서 제 예측은 B C D가 나올거라고 예상을 하고, 테스트를 돌렸습니다.

```text
Expecting actual:
  ["B", "C", "D"]
to contain exactly (and in same order):
  ["C", "A", "D"]
but some elements were not found:
  ["A"]
and others were not expected:
  ["B"]
```

네, 딱 예측한 결과 그대로 FIFO로 나왔죠.


## 다음 시간

다음 시간에는 이 클래스를 DB 조회할때 앞에다가 얹어보려고 합니다.<br>
NewsService는 단 한곳도 건들이지 않고요.