---
title: "CSES - Bit Strings"
date: 2026-05-14 21:00:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-04-24 |
| 소요 시간 | 0.40s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1617/)

---

# Bit Strings

Your task is to calculate the number of bit strings of length $n$.

For example, if $n = 3$, the correct answer is 8, because the possible bit strings are 000, 001, 010, 011, 100, 101, 110, and 111.

### Input
The only input line has an integer $n$.

### Output
Print the result modulo $10^9 + 7$.

### Constraints
* $1 \le n \le 10^6$

### Example
**Input:**
```
3
```

**Output:**
```
8
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

# 비트 문자열

당신의 임무는 길이가 $n$인 비트 문자열의 개수를 계산하는 것입니다.

예를 들어, $n = 3$인 경우 정답은 8입니다. 가능한 비트 문자열은 000, 001, 010, 011, 100, 101, 110, 111이기 때문입니다.

### 입력
유일한 입력 줄에는 정수 $n$이 주어집니다.

### 출력
결과를 $10^9 + 7$로 나눈 나머지를 출력합니다.

### 제한 사항
* $1 \le n \le 10^6$

### 예제
**입력:**
```
3
```

**출력:**
```
8
```

</details>

---

## 초기 접근 및 로직 구상 방식

비트의 개수를 세서, 10^9 + 7 로 나눈 나머지 값을 출력하면 된다.
n자리 비트의 개수는 2^n 이니깐, 2^n % (10^9 + 7) 의 값을 출력하면 끝

## 시도

**1차 시도.**
- 성공

<details markdown="1">
<summary><b>1차 코드 (클릭하여 펼치기)</b></summary>

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
 
public class BitStrings {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int result = 1;
 
        for (int i = 0; i < n; i++) {
            result = (2 * result) % 1000000007;
        }
 
        System.out.println(result);
 
    }
 
}
```
</details>

근데 값이 큰데 왜 오버플로우가 안나지? long으로도 안될거 같은데, 통과는 함.
오버플로우 관해서, 계산을 좀 해봤는데, 범위가 벗어나기 전에, 1000000007으로 나눈 나머지 값이니깐, 범위를 벗어나지 않는듯. int 최댓값이 약 2.1×10^9 이고, 지금 계산이 2 * result 니깐, 2×10^9. 겨우겨우 int안에 들어감.

허나 빠른 거듭제곱(fast exponentiation 또는 binary exponentiation) 더 좋은 풀이가 있다고 해서 시도해봄.

<details markdown="1">
<summary><b>빠른 거듭 제곱이란? (클릭하여 펼치기)</b></summary>

# a^n을 빠르게 구하는 법임.
- a^1 제곱 a^2
- a^2 제곱 a^4
- a^4 제곱 a^8

제곱만 반복시 a의 1 2 4 8 16 ... 제곱들이 빠르게 만들어짐. 즉, 필요한 자리만 골라서 곱하면 끝.
10을 이진수로 바꾸면 1010 (8 4 2 1). 여기서 자리가 1인것만 골라서, a^8, a^2 골라서 곱하면 a^10을 쉽게 계산 가능함.

n을 이진수로 바꾸고 자리수 만큼 돌면 되니깐 기존 반복문 O(n)보다 훨씬 빠르게 처리가 가능함. O(log n)

</details>

## 배운 점

- (a * b) % m == ((a % m ) * (b % m)) % m 처럼 mod가 곱셈/덧셈에 분배됨.즉, 곱셈 도중 어디서 나머지 값을 계산해도 최종 결과는 같다.
- 이진수로 쪼갤때, n & 1로 마지막 비트를 검사후, n = n >> 1; 로 비트 시프트 해가며 구할 수 있음.
- a^n mod p, 페르마의 소정리를 이용한 모듈러 역원 계산(a^(p-2) mod p), 행렬 거듭제곱(피보나치 O(log n) 같은 것)에도 사용 할 수 있다.

## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
 
public class BitStrings {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        long result = 1;
        long base = 2;
        int mod = 1000000007;
 
        while (n > 0) {
            if ((n & 1) == 1) {
                result = (result * base) % mod;
            }
            base = (base * base) % mod;
            n = n >> 1;
        }
 
        System.out.println(result);
 
    }
}
```
