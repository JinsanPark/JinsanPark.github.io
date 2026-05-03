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

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

$n$개의 정수로 이루어진 배열이 주어집니다. 당신은 이 배열을 모든 원소가 이전 원소보다 크거나 같도록(즉, 비내림차순 정렬이 되도록) 수정하려고 합니다.

한 번의 연산으로, 당신은 어떤 원소든 그 값을 1만큼 증가시킬 수 있습니다. 배열을 증가하는 상태로 만들기 위해 필요한 **최소 연산 횟수**는 얼마일까요?

### 입력
첫 번째 줄에 배열의 크기인 정수 $n$이 주어집니다.
두 번째 줄에 배열의 내용인 $n$개의 정수 $x_1, x_2, \dots, x_n$이 주어집니다.

### 출력
최소 연산 횟수를 출력합니다.

### 제한 사항
- $1 \le n \le 2 \cdot 10^5$
- $1 \le x_i \le 10^9$

### 예제
**입력:**
```
5
3 2 5 1 7
```

**출력:**
```
5
```

</details>

## 초기 접근 및 로직 구상 방식

- 배열에서 현 배열이 이전 배열보다 커야한다. +1을 몇번 하느냐 같아 보임.

3 2 5 1 7
2를 4로 +2
5는 그러면 4보다 크고
1을 6으로 +5
라고 가정하면 7이라는 틀린값이 나온다.

- 조건이 큰게(>) 아니라 작지만 않으면 된다(>=) 
3 2 5 1 7
2를 3로 +1
5는 그러면 4보다 크고
1을 5으로 +4
총 5가 나옴. 예제 출력과 일치. 따라서 맞는 방향이라고 판단.


## 시도

**1차 시도는 .**
- 실패. 저번 처럼 값에 오버플로우가 났다.

<details markdown="1">
<summary><b>1차 시도 코드 (클릭하여 펼치기)</b></summary>


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
 
        int count = 0; // 오버플로우 발생
 
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

</details>

**2차 시도는 .**
-값에 오버플로우가 났다는 뜻은, int값으로 충분하지 못하다는 이야기이니, - int count = 0;를 long count = 0; 으로 수정.
-성공.


## 배운 점

- count에 연산이 들어가는것을 망각하고, int로 충분하겠다고 생각한것이 실패 원인.

배열 요소의 최대값은 10^9 최대값 1,000,000,000
배열이 1000000000 1 1 1 1 1 ..... 처럼 쭉 이어진다고 생각해보면, <br>
1,000,000,000 - 1 = 999,999,999 count == 999,999,999 <br>
1,000,000,000 - 1 = 999,999,999 count == 1,999,999,998; <br>
1,000,000,000 - 1 = 999,999,999 count == 2,999,999,997; (벌써 int 최대값인 21억 돌파.)

배열의 길이 2×10⁵ 이니깐 
(2×10⁵ - 1) * 999,999,999 = 199,998,999,800,001 이라는 무지막지한 값이 나올 수가 있다.
따라서 int가 아닌 long을 써야한다.


## 최종 풀이 코드

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