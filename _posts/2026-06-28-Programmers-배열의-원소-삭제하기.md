---
title: "배열의 원소 삭제하기"
date: 2026-06-28 20:30:00 +0900
categories: [프로그래머스, 기초문제]
tags: [java, 문제풀이]
math: true
---

## 문제 설명

- [문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/181844)

---

## 초기 접근 및 로직 구상 방식

문제는 배열 2개(arr, delete_list)를 비교하여 delete_list의 원소들을 arr 배열에서 제거하여 출력하는 문제.
ArrayList를 사용하여 각각 배열 2개를 나눠서 저장후, removeAll() 메소드 사용.

## 시도

**1차 제출**

- 성공. 허나 코드 개선 사항에 대해서 물어봄.

<details markdown="1">
<summary><b>초기 코드</b></summary>

```java

import java.util.*;

class Solution {
    public int[] solution(int[] arr, int[] delete_list) {
        
        List<Integer> list = new ArrayList<>();
        List<Integer> dList = new ArrayList<>();
        
        for(int i : arr){
            list.add(i);
        }
        
        for(int i : delete_list){
            dList.add(i);
        }
        
        list.removeAll(dList);
        int[] answer = new int[list.size()];
        
        for (int i = 0; i < list.size(); i++){
            answer[i] = list.get(i);
        }
        
        return answer;
    }
}

```

</details>

> **힌트 1**
> "순서는 필요 없고 오직 포함 여부만 빠르게 확인하면 되는" 자료구조로 바꾸면 어떤 게 떠오르는거?
{: .prompt-tip }

- 바로 HashSet 떠오름. 위 방향대로 어차피 순서의 상관없는 delete_list를 HashSet으로 받기로 생각함.
**2차 제출**

- 성공.


## 배운 점

- 중복이 상관 없는가 하면 바로 HashSet이 떠오르지만, 막상 문제를 보면 떠오르지 않는다. '이 값이 이 안에 있냐 없냐만 따진다' 로 생각하면 HashSet을 떠올리기 더 쉽다.
- int[]를 바로 ArrayList로, Arrays.asList(arr) 처럼 받으면 int값이 들어갈거 같지만 List는 int, double 같은 기본형을 타입 인자로 받지 못해서, int[] 배열 자체가 원소 하나로 들어간 List<int[]>가 만들어진다. 함수를 Integer[]로 받거나 스트림을 사용하면 가능은 하지만, 지금 같은 문제에서는 이미 함수가 int로 선언되었기 때문에, 그대로 두는 것이 더 좋다. 


## 최종 풀이 코드

```java

import java.util.*;

class Solution {
    public int[] solution(int[] arr, int[] delete_list) {
        
        List<Integer> list = new ArrayList<>();
        Set<Integer> delete = new HashSet<>();
        
        for(int i : arr){
            list.add(i);
        }
        
        for(int i : delete_list){
            delete.add(i);
        }
        
        list.removeAll(delete);
        int[] answer = new int[list.size()];
        
        for (int i = 0; i < list.size(); i++){
            answer[i] = list.get(i);
        }
        
        return answer;
    }
}

```