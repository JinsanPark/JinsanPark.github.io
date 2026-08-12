---
title: "기능 개발"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [프로그래머스]
tags: [java, 프로그래머스]
math: true
---

- [문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/42586)


## 초기 접근 및 로직 구상 방식




> **힌트 1**
> 
{: .prompt-tip }



## 제출

**1차 제출**

- 성공.

```java

import java.util.*;

class Solution {
    public int[] solution(int[] progresses, int[] speeds) {
        List<Integer> result = new ArrayList<>();
        List<Integer> listP = new ArrayList<>();
        List<Integer> listS = new ArrayList<>();
         
        for(int i = 0; i < progresses.length; i++){
            listP.add(progresses[i]);
            listS.add(speeds[i]);
        }
        
        while(!listP.isEmpty()){
            for(int i = 0; i < listP.size(); i++){
                listP.set(i, listP.get(i) + listS.get(i));
            }
            
            int count = 0;
            while(!listP.isEmpty() && listP.get(0) >= 100){
                listP.remove(0);
                listS.remove(0);
                count++;
            }
            
            if(count != 0){
                result.add(count);
            }
        }
        
        int[] answer = new int[result.size()];
        
        for(int i = 0; i < result.size(); i++){
            answer[i] = result.get(i);
        }
        
        return answer;
    }
}

```

## 배운 점

- 

## 최종 풀이 코드


```java



```