---
title: "최빈값 구하기"
date: 2026-07-02 14:50:00 +0900
categories: [프로그래머스, 입문 문제]
tags: [java, 문제풀이]
math: true
---

## 문제 설명

- [문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/120812)

---

## 초기 접근 및 로직 구상 방식

배열을 하나 더 만들어, 인덱스가 나온 숫자이고, 그 안의 요소가 나온 횟수로 해서 풀기로 함.
허나 코드를 짜다가, 최빈값이 2개 이상일때를 조건분기를 구현하지 못했음.
기존의 코드로만 구현하려니, 최빈값을 비교할 대상이 없었음.



> **힌트 1**
>  [1, 1, 2, 2]처럼 가장 많이 나온 값이 둘 이상이면, 최댓값 하나만 찾는 코드는 무엇을 놓치게 될까요?
{: .prompt-tip }

- [1,1,2,2]라면 count 배열로 옮기면 [0,2,2]; 이고 이렇게 하면, 최빈값이 2개라 -1를 돌려줘야 하지만, 기존 코드대로라면 if(count[i] > max) 에 따라서, 1을 돌려줌. 따라서 무엇이 최빈값인가를 알려줄 변수가 필요하다고 판단함. int mode로 최빈값을 판단하고, 같은 값이 있다면 mode++를 해줌. mode가 1이면, 그에 해당하는 i가 최빈값. mode == 1이면 answer = i; mode > 1 이라면 -1을 돌려준다. 위의 [1,1,2,2]에서는 mode가 2가 됨으로, -1이 return된다.

```java
        for(int i = 0; i < count.length; i++){
            if(count[i] > max){
                max = count[i];
                answer = i;
                mode = 1;
            } else if (max == count[i]){
                mode++;
            }
        }
```

> **힌트 2**
>  count 배열의 크기가 [array.length + 1] 인데, 문제 조건에 따라서 만약 [999]가 입력이라면?
{: .prompt-tip }

- count[999]++; 가 되어야 하는데, 이렇게 된다면 범위를 벗어남. 따라서 문제에서 주어진 1000으로 고정.

## 시도

**1차 제출**

- 성공.

## 배운 점

- 입력값은 다른 배열로 그대로 넣을때는, 배열의 길이가 아니라 값의 범위로 잡아야 범위를 벗어나지 않음.

- 최대값은 카운팅이 끝나기 전에는 몇 개 인지 알 수 없기에, 첫번째 반복문에서 count[]를 채운뒤, 다음 반복문에서 max, mode, answer를 처리해야함.

## 최종 풀이 코드

```java

class Solution {
    public int solution(int[] array) {
        int[] count = new int[1000];

        for(int i = 0; i < array.length; i++){
            count[array[i]]++;
        }
        
        int answer = 0;
        int max = 0;
        int mode = 0;
        
        for(int i = 0; i < count.length; i++){
            if(count[i] > max){
                max = count[i];
                answer = i;
                mode = 1;
            } else if (max == count[i]){
                mode++;
            }
        }

        return mode == 1 ? answer : -1;
    }
}

```