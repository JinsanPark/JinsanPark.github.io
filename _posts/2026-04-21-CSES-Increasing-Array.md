---
title: "Increasing Array"
date: 2026-04-23 20:10:21 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-04-23 |
| 소요 시간 | 0.27s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1083/)

# Increasing Array

You are given an array of $n$ integers. You want to modify the array so that it is increasing, i.e., every element is at least as large as the previous element.

On each move, you may increase the value of any element by one. What is the minimum number of moves required?

### Input
The first input line contains an integer $n$: the size of the array.
Then, the second line contains $n$ integers $x_1, x_2, \ldots, x_n$: the contents of the array.

### Output
Print the minimum number of moves.

### Constraints
* $1 \le n \le 2 \cdot 10^5$
* $1 \le x_i \le 10^9$

### Example
**Input:**
```
5
3 2 5 1 7
```

**Output:**
```
5
```

## 풀이 코드

틀린코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;
 
public class IncreasingArray {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int[] line = new int[n];
 
        StringTokenizer st = new StringTokenizer(br.readLine());
        for (int i = 0; i < n; i++) {
 
            line[i] = Integer.parseInt(st.nextToken());
 
        }
 
        int count = 0;
 
        for (int i = 1; i < n; i++) {
            if (line[i - 1] > line[i]) {
 
                int sub = line[i - 1] - line[i];
                count += sub;
                line[i] += sub;
 
            }
        }
 
        System.out.println(count);
 
    }
}
```

정답 코드
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class IncreasingArray {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int[] line = new int[n];

        StringTokenizer st = new StringTokenizer(br.readLine());
        for (int i = 0; i < n; i++) {

            line[i] = Integer.parseInt(st.nextToken());

        }

        long count = 0;

        for (int i = 1; i < n; i++) {
            if (line[i - 1] > line[i]) {

                int sub = line[i - 1] - line[i];
                count += sub;
                line[i] += sub;

            }
        }

        System.out.println(count);

    }
}

```

## 초기 접근 방식
- **초기 접근**: 배열에서 현 배열이 이전 배열보다 커야함. +1을 몇번 하느냐 같음.
  
- **로직 구상**: 배열 만들고, 이전것과 비교하면 될듯.인풋은
3 2 5 1 7
2를 4로 +2
5는 그러면 4보다 크고
1을 6으로 +5
틀림

-조건이 큰게 아니라 작아서는 안되는것이니깐.
3 2 5 1 7
2를 3로 +1
5는 그러면 4보다 크고
1을 5으로 +4
총 5가 나옴.

## 시도

1차 실패.
출력값중 몇가지가 오버플로우로 음수값 나옴.
출력 결과 보니 long으로 받아야할듯.

두 번째 시도
long으로 바꾸고 정답 처리.

## AI 조언

-n이 최대 2×10⁵일 때 count의 이론적 최댓값을 한 번에 식으로 표현해보는 게 좋겠습니다.



**얻은 깨달음** :
count에 연산이 들어가는것을 망각하고, int로 충분하겠다고 생각한것이 실패 원인.

배열 요소의 최대값은 10^9 최대값 1,000,000,000

배열이 1000000000 1 1 1 1 1 ..... 처럼 쭉 이어진다고 생각해보면

1,000,000,000 - 1 = 999,999,999 count == 999,999,999
1,000,000,000 - 1 = 999,999,999 count == 1,999,999,998
1,000,000,000 - 1 = 999,999,999 count == 2,999,999,997 (벌써 int 최대값인 21억 돌파.)

배열의 길이 2×10⁵ 이니깐 
(2×10⁵ - 1) * 999,999,999 = 199,998,999,800,001 이라는 무지막지한 값이 나올 수 있음.
따라서 int가 아닌 long을 써야함.