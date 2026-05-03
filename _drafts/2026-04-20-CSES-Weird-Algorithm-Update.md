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

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>


양의 정수 $n$을 입력으로 받는 알고리즘을 생각해 봅시다. 
- $n$이 **짝수**라면, 알고리즘은 이를 2로 나눕니다.
- $n$이 **홀수**라면, 알고리즘은 $n$에 3을 곱하고 1을 더합니다.

알고리즘은 $n$이 1이 될 때까지 이 과정을 반복합니다. 예를 들어, $n=3$일 때의 수열은 다음과 같습니다:
$$3 \rightarrow 10 \rightarrow 5 \rightarrow 16 \rightarrow 8 \rightarrow 4 \rightarrow 2 \rightarrow 1$$

당신의 임무는 주어진 $n$에 대해 이 알고리즘의 실행 과정을 시뮬레이션하는 것입니다.

### 입력
첫 번째 줄에 정수 $n$이 주어집니다.

### 출력
알고리즘이 진행되는 동안의 모든 $n$의 값을 한 줄에 공백으로 구분하여 출력합니다.

### 제한 사항
- $1 \le n \le 10^6$

### 예제
**입력:**
```
3
```

**출력:**
```
3 10 5 16 8 4 2 1
```

</details>

---

## 초기 접근 및 로직 구상 방식

- n이 짝수면 2로 나누고, 홀수이면 n * 3 + 1 해줌. n이 1이 될때까지 반복.
- 짝수면 n % 2 == 0이고, 홀수면 n % 2 != 0. if문 사용으로 n이 1까지, while문으로 n이 1까지 돌리는게 제일 간단해보인다.

## 시도

**1차 시도는 .**

-실패, 처음 n 출력을 못보고 출력하지 않아 틀렸다.

**2차 시도는 .**

<!-- 어떻게 고쳤는지, 왜 그렇게 고쳤는지 (마음에 안 들어서 같은 사소한 동기도 OK) -->
-실패. 정수값에서 오버플로우가 발생했다. 아마 int로도 충분하지 못한거 같다.

**3차 시도는 .**
-성공. int를 long으로 바꾸니 성공했다.

## 배운 점

- 데이터 크기에 따라서 int, long등을 선언 할 줄 알아야한다. 특히나 n의 범위가 int값 안에 들어간다 하더라도, 연산이 들어가면 이야기가 달라진다.

## 최종 풀이 코드

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
