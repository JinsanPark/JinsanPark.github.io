---
title: "CSES - Permutations"
date: 2026-04-27 15:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-04-27 |
| 소요 시간 | 0.27s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/view/1070/)


---
# Permutations

A permutation of integers $1, 2, \ldots, n$ is called **beautiful** if there are no adjacent elements whose difference is 1.

Given $n$, construct a beautiful permutation if such a permutation exists.

### Input
The only input line contains an integer $n$.

### Output
Print a beautiful permutation of integers $1, 2, \ldots, n$. If there are several solutions, you may print any of them. If there are no solutions, print "NO SOLUTION".

### Constraints
* $1 \le n \le 10^6$

### Example 1
**Input:**
```
5
```
**Output:**
```
4 2 5 3 1
```

### Example 2
**Input:**
```
3
```
**Output:**
```
NO SOLUTION
```
---


## 초기 접근 및 로직 구상 방식

- 인접한 숫자의 차이가 1이 되지 않게 하려면 홀수와 짝수를 그룹 지어 분리해서 배치해야겠다고 생각함.


## 시도

1차 시도 실패.
엣지 케이스 n = 4일때 실패.

2차 시도 - 성공?
n == 4일때 

```java
         if (n == 4) {
 
                System.out.print(2 + " ");
                System.out.print(4 + " ");
                System.out.print(1 + " ");
                System.out.print(3);
                check = false;
                break;
            }
```

로 해결하긴 했는데, 하드코딩이라 별로 안 좋아보임.

1 3 2 4 - 틀린 값(홀수 먼저 작은 순서대로 오는 기존 방식)

3 - 2는 1
홀수 먼저 오면 n = 4일때 3 - 2로 문제 요구와 안맞음.

2 4 1 3 - 맞는 값(짝수 먼저 작은 순서대로 오는 방식)
짝수 먼저 오면 앞쪽 짝수는 항상 커지고, 홀수는 항상 1부터 시작하니, 경계에서의 두수의 차가 1이 나올 수 없음.

1 3 5 2 4 6 - 홀수 먼저

2 4 6 1 3 5 - 짝수 먼저

1 3 5 7 2 4 6 8 - 홀수 먼저

2 4 6 8 1 3 5 7 - 짝수 먼저.

짝수 먼저 들어오니 맞음.

3차 시도 - 실패. 변경하니 n == 1일때 엣지 케이스 발생.

4차 시도 - 성공

```java
if (n == 1) {
            System.out.println(1);
            check = false;
} else {...}
```
으로 해결.


## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class Permutations {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int[] line = new int[n];
        boolean isEvenToOdd = false;
        boolean check = true;
        int input = 2;
        StringBuilder sb = new StringBuilder();
 
        if (n == 1) {
            System.out.println(1);
            check = false;
        } else {
 
 
            for (int i = 0; i < n; i++) {
 
 
                line[i] = input;
                sb.append(input).append(" ");
                input += 2;
 
                if (isEvenToOdd && Math.abs(line[i] - line[i - 1]) == 1) {
 
                    System.out.println("NO SOLUTION");
                    check = false;
                    break;
 
                }
 
                if (input > n) {
 
                    input = 1;
                    isEvenToOdd = true;
 
                }
 
            }
        }
 
        if (check) {
            System.out.println(sb);
        }
 
    }
}
```

