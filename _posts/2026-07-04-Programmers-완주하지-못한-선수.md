---
title: "완주하지 못한 선수"
date: 2026-07-04 18:00:00 +0900
categories: [프로그래머스, 해시]
tags: [java, 문제풀이]
math: true
---

- [문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/42576)

---

## 초기 접근 및 로직 구상 방식

일단 저번에 [풀었던 문제](/posts/Programmers-배열의-원소-삭제하기/)가 떠올라 HashSet을 이용하여 중복값을 제거하기로 결정함. 단, removeAll이면 중복 값이 전부 사라지니, remove를 사용.
다 짜고 보니 for문이 3개라 굉장히 느려보인다고 생각함.그래도 O(n)이라 생각했는데, O(n^2)이 나온다함.


<details markdown="1">
<summary><b>초기 코드</b></summary>

```java

import java.util.*;

class Solution {
    public String solution(String[] participant, String[] completion) {
        List<String> list = new ArrayList<>();
        HashSet<String> set = new HashSet<>();
        
        for(String i : participant){
            list.add(i);
        }
        
        for(String i : completion){
            set.add(i);
        }
        
        for(String i : set){
            list.remove(i);
        }
        
        return list.get(0);
    }
}

```
- 애초에 중복 처리로 인해 통과가 안되는 코드였음.

</details>


> **힌트 1**
>  값 하나하나에 대해 "몇 번 나왔는지" 개수를 저장하고 O(1)로 꺼내볼 수 있는 도구는?
{: .prompt-tip }

따라서 HashMap으로 선회.
participant에 있는 사람 1명씩 추가하고, completion에 있는 사람은 1명씩 빼기
그러면 완주 못한 1명만 1의 값을 가지게 되는고, 그 사람만 출력하면 끝나는 문제 같음.


## 시도

**1차 제출**

- 성공.

## 배운 점

- 처음에는 단순히 반복문이 3개라서 O(n)으로 생각했으나, 세 번째 루프가 set을 돌면서 list.remove(i)를 호출하는데, 한번 호출 할때 마다 O(n)이고, 그 반복을 n만큼 반복 하니깐 O(n ^ 2)이 된다. 반면, HashMap으로 접근을 하면 O(1)이기 때문에, O(1)를 n번 반복하니 O(n)으로 훨씬 빠르게 처리가 가능하다. 그리고 애초에 HashSet으로는 개수 정보를 판단 할 수 없어서 해결 할 수 없는 문제였다.

- 만일 Integer == Integer 였다면, 이 코드는 실패였음. 왜냐하면, Integer끼리 비교해버린다면 객체라서 캐쉬되는 범위가 -128 ~ 127로 전부이기 때문에, Integer a = 100, b = 100;이면 a == b 는 맞지만, Integer a = 200, b = 200; 이면 a == b가 false가 나와 버린다. 따라서  자동으로 풀어서 값 비교를 해주는 int == Integer 처럼 사용하거나, equals()를 사용해야 한다.

## 최종 풀이 코드

```java

import java.util.*;

class Solution {
    public String solution(String[] participant, String[] completion) {
        String answer = "";
        HashMap<String, Integer> map = new HashMap<>();
        
        for(String i : participant){
            map.put(i, map.getOrDefault(i, 0) + 1);
        }
        
        for(String i : completion){
            map.put(i, map.getOrDefault(i,0) - 1);
        }
        
        for (Map.Entry<String, Integer> i : map.entrySet()){
            if(i.getValue() >= 1){
               answer = i.getKey();
            }
        }
        
        return answer;
    }
}

```