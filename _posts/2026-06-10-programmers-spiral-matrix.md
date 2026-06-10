---
title: "정수를 나선형으로 배치하기"
date: 2026-06-10 11:00:00 +0900
categories: [프로그래머스]
tags: [java, 문제풀이]
math: true
---

## 문제 설명

[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/181832)

---

## 초기 접근 및 로직 구상 방식

문제에서 나선형으로 숫자를 +1씩 증가 시켜 가며 채우는 문제. 어떻게 숫자를 채울까 고민.
예전 CSES 문제였던 계차 수열 문제 처럼 규칙을 찾다가 포기하고, 그냥 방향을 사용하여 조건에 맞게 방향 전환으로 하기로 결정함.
{% raw %}
```java

int[][] direction =  {{0,1},{1,0},{0,-1},{-1,0}}; // 순서대로 오른쪽, 아래, 왼쪽, 위쪽

d = (d + 1) % 4;// 으로 방향 번경

```
{% endraw %}


```java

for(int i = 1; i <= n*n; i++) {
            if(0 <= nextRow && nextRow < n && 0 <= nextCol && nextCol < n && answer[nextRow][nextCol] == 0) {
                answer[nextRow][nextCol] = i;
                curRow = nextRow;
                curCol = nextCol;
                nextRow = curRow + direction[d][0];
                nextCol = curCol + direction[d][1];
                
            } else {
                d = (d + 1) % 4;
                nextRow = curRow + direction[d][0];
                nextCol = curCol + direction[d][1];
            }    
        }

```
에서 if - else 에서 else 구문으로 들어가도 i는 증가하여, 다 채워지지 못한 상태로 종료가 되어,

```java
for(int i = 1; i <= n*n; ) {
            if(0 <= nextRow && nextRow < n && 0 <= nextCol && nextCol < n && answer[nextRow][nextCol] == 0) {
                answer[nextRow][nextCol] = i;
                curRow = nextRow;
                curCol = nextCol;
                nextRow = curRow + direction[d][0];
                nextCol = curCol + direction[d][1];
                i++; // i++ 의 위치를 이동시켜, 배열에 삽입이 되었을때만 실행되도록 바꿈.
                
            } else {
                d = (d + 1) % 4;
                nextRow = curRow + direction[d][0];
                nextCol = curCol + direction[d][1];
            }    
        }

```

## 제출

**1차 제출**

성공함. 허나 코드 내부에 변수가 너무 많다고 생각함.

<details markdown="1">
<summary><b>1차 제출 코드 (클릭하여 펼치기)</b></summary>

{% raw %}
```java
class Solution {
    public int[][] solution(int n) {
        int[][] answer = new int[n][n];
        int[][] direction = {{0,1},{1,0},{0,-1},{-1,0}};
        int curRow = 0;
        int curCol = 0;
        int d = 0;
        int nextRow = curRow;
        int nextCol = curCol;

        for(int i = 1; i <= n*n;) {
            if(0 <= nextRow && nextRow < n && 0 <= nextCol && nextCol < n && answer[nextRow][nextCol] == 0) {
                answer[nextRow][nextCol] = i;
                curRow = nextRow;
                curCol = nextCol;
                nextRow = curRow + direction[d][0];
                nextCol = curCol + direction[d][1];
                i++;
            } else {
                d = (d + 1) % 4;
                nextRow = curRow + direction[d][0];
                nextCol = curCol + direction[d][1];
            }    
        }

        return answer;
    }
}
```
{% endraw %}

</details>

변수가 너무 많다 보니, 코드가 지저분해 보인다고 생각함.

**2차 시도는 .**

그래서 nextCol, nextRow 를 쓰지 않고, col과 rol를 갱신해 다음에 채울 곳으로 이동시키는 방법을 생각해봄.

<details markdown="1">
<summary><b>2차 제출 코드</b></summary>

{% raw %}
```java
class Solution {
    public int[][] solution(int n) {
        int[][] answer = new int[n][n];
        int[][] direction = {{0,1},{1,0},{0,-1},{-1,0}};
        int row = 0;
        int col = 0;
        int d = 0;

        for(int i = 1; i <= n*n; i++) {

            answer[row][col] = i;

            if(0 <= row + direction[d][0] && row + direction[d][0] < n && 0 <= col + direction[d][1] && col + direction[d][1] < n && answer[row + direction[d][0]][col + direction[d][1]] == 0) {
                row = row + direction[d][0];
                col = col + direction[d][1];
            } else {
                d = (d + 1) % 4;
                row = row + direction[d][0];
                col = col + direction[d][1];
            }    
        }

        return answer;
    }
}

```
{% endraw %}

next 변수 없이 현재 col, row를 갱신시켜 다음 지점을 가르키게함.

</details>

허나 if문 내부가 오히려 더 보기 풀편해진다는걸 알게됨. 

## 배운 점

- 변수가 많더라도, 그 변수의 이름과 역할이 명확하면, 놔두는게 더 좋을 수 도 있다고 알게됨. 선택은 쓰는 사람의 몫.


