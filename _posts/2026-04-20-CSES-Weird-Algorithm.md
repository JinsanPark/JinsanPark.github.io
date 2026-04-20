---
title: "Weird Algorithm"
date: 2026-04-20 13:56:21 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-04-20 |
| 소요 시간 | 0.06s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1068)

Consider an algorithm that takes as input a positive integer n. If n is even, the algorithm divides it by two, and if n is odd, the algorithm multiplies it by three and adds one. The algorithm repeats this, until n is one. For example, the sequence for n=3 is as follows:

$$ 3 \rightarrow 10 \rightarrow 5 \rightarrow 16 \rightarrow 8 \rightarrow 4 \rightarrow 2 \rightarrow 1$$

Your task is to simulate the execution of the algorithm for a given value of n.

**Input**

The only input line contains an integer n.

**Output**

Print a line that contains all values of n during the algorithm.

**Constraints**

$$1 \le n \le 10^6$$

**Example**

Input:
```
3
```

Output:
```
3 10 5 16 8 4 2 1
```

## 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class WeirdAlgorithm {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        long n = Integer.parseInt(br.readLine());
        StringBuilder sb = new StringBuilder();
        sb.append(n).append(" ");

        while (n != 1) {
            if (n % 2 == 0) {
                n /= 2;
            } else {
                n = n * 3 + 1;
            }
            sb.append(n).append(" ");
        }

        System.out.println(sb);
    }
}
```

## 초기 접근 방식
- **초기 접근**: n이 짝수면 2로 나눔. 홀수이면 n * 3 + 1 해줌. n이 1이 될때까지.
  
- **로직 구상**: 짝수면 n % 2 == 0이고, 홀수면 n % 2 != 0. if문 사용으로 n이 1까지. while문으로 n이 1까지 돌리는게 제일 간단해보임.

## 시도

1차 실패 : 맨 처음 n 출력 까먹음.

2차 실패 : 런타임 에러 + 오버 플로우(음수 나옴) 발생.

3차 성공 : int 말고 long 사용

**얻은 깨달음** :
데이터 사이즈에 따라서, int말고 다른 long 같은 변수도 선언할 줄 알아야함.
그렇지 않으면, 오버플로우가 발생 할 수 있음.
