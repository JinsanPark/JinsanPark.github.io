---
title: "중복된 문자 제거"
date: 2026-07-08 23:30:00 +0900
categories: [프로그래머스, 입문 문제]
tags: [java, 문제풀이]
math: true
---

- [문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/120888)

---

## 초기 접근 및 로직 구상 방식

문자열에 중복을 제거하고 담는 것이니, set을 생각함. 이 문제에서는 출력값이 순서대로이니 입력값을 순서대로 받는 LinkedHashSet을 사용하기로 함.


## 시도

**1차 제출**

- 성공.

<br>

<details markdown="1">
<summary><b>1차 제출 코드</b></summary>

```java

import java.util.*;

class Solution {
    public String solution(String my_string) {
        
        Set<Character> set = new LinkedHashSet<>();
        char[] c = my_string.toCharArray();
        StringBuilder sb = new StringBuilder();
        
        for(int i = 0; i < c.length; i++){
            set.add(c[i]);
        }
        
        for(char i : set){
            sb.append(i);
        }

        return sb.toString();
    }
}

```

</details>
<br>

답은 맞았으나, 상당히 유사해보이는 for문이 2번 돌아, for문을 하나로 줄일 수 있는지 물어봄.

> **힌트 1**
>  set.add()는 반환값이 있는데, 이 반환값이 뭘 알려주는지 확인해보면?
{: .prompt-tip }

set.add()는 값이 있으면 false, 값이 없으면 true를 반환해주니, set.add()가 참이면, 값이 없다는 소리고, for문은 어차피 문자의 순서대로 돌고, 그 순서대로 StringBuilder에 넣어줌.

```java
        for(int i = 0; i < c.length; i++){
            if(set.add(c[i])){
                sb.append(c[i]);
            }
        }
```

따라서 위 처럼 코드를 짠다면, 문자를 앞에서 하나씩 돌면서 set에 없으면 sb에 넣고, sb를 String으로 return 해주면 됨.

**2차 제출**

- 성공

## 배운 점

- LinkedHashSet을 고른건 틀린 판단은 아니었음. 허나, 루프를 하나로 합치면서 순서를 set이 아니라, sb에 넣을 때 정해지도록 함. 즉, set이 순서를 안 지켜도 되니 HashSet보다 좀 더 느린 LinkedHashSet을 사용할 이유가 없음. 따라서, 처음 선택이 맞더라도, 코드 구조가 바뀌면 그 선택이 최선인지 다시 한번 살펴 볼 필요가 있어 보임.

## 최종 풀이 코드

```java

import java.util.*;

class Solution {
    public String solution(String my_string) {
        
        Set<Character> set = new HashSet<>();
        char[] c = my_string.toCharArray();
        StringBuilder sb = new StringBuilder();
        
        for(int i = 0; i < c.length; i++){
            if(set.add(c[i])){
                sb.append(c[i]);
            }
        }

        return sb.toString();
    }
}

```