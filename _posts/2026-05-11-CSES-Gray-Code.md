---
title: "CSES - Gray Code"
date: 2026-05-11 12:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-11 |
| 소요 시간 | 0.18s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/2205/)

---

# Gray code

A Gray code is a list of all $2^n$ bit strings of length $n$, where any two successive strings differ in exactly one bit (i.e., their Hamming distance is one).

Your task is to create a Gray code for a given length $n$.

### Input
The only input line has an integer $n$.

### Output
Print $2^n$ lines that describe the Gray code. You can print any valid solution.

### Constraints
* $1 \le n \le 16$

### Example
**Input:**
```
2
```

**Output:**
```
00
01
11
10
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>


그레이 코드는 길이가 $n$인 $2^n$개의 모든 비트 문자열을 나열한 리스트로, 임의의 연속된 두 문자열은 정확히 한 비트만 다릅니다 (즉, 해밍 거리가 1입니다).

당신의 임무는 주어진 길이 $n$에 대한 그레이 코드를 생성하는 것입니다.

### 입력
유일한 입력 줄에는 정수 $n$이 주어집니다.

### 출력
그레이 코드를 나타내는 $2^n$개의 줄을 출력합니다. 가능한 유효한 답이 여러 개라면 그중 아무거나 출력해도 좋습니다.

### 제한 사항
* $1 \le n \le 16$

### 예제
**입력:**
```
2
```

**출력:**
```
00
01
11
10
```

</details>

---

## 초기 접근 및 로직 구상 방식

그레이 코드 - 한번에 딱 한 자리만 바뀜.
2^n 만큼 0의 개수 배열 만들고, sb에 넣는다.

기존거랑 비교해서 하나만 다르다. - 이런게 하면 전체 문자열을 비교하는거라 너무 오래걸릴거 같아.

> **힌트 1**
> n = 3 이라고 했을때, 그레이 코드와 이진 사이의 규칙이 있지 않을까요?
{: .prompt-tip }

 000 000 <br>
 001 001 <br>
 010 011 <br>
 011 010 <br>
 100 110 <br>
 101 111 <br>
 110 101 <br>
 111 100 <br>

 000 <br>
 000 xor <br>
 000
 
 001 <br>
 000 xor <br>
 001
 
 010 <br>
 001 xor <br>
 011
 
 011 <br>
 001 xor <br>
 010

 2번 xor 연산후 마스크값에 +1한다. 인거 같은데 확실치 않아서 한번 더 물어봄.

> **힌트 2**
> i = 0,1,2,3,4,5,6,7에 대해 mask가 0,0,1,1,2,2,3,3이 되려면, i에 어떤 연산을 한 번 적용하면 될까요?
{: .prompt-tip }
- 이진으로 생각해보면, i가 0, 1, 10, 11, 100, 101, 110, 111이고, 마스크는 0, 0, 1, 1, 10, 10, 11, 11이니깐 왼쪽으로 하나씩 밀어버리는 쉬프트 연산을 해버리면 된다.

따라서 i ^ (i >> 1) 으로 계산하면  2번 xor 연산후 마스크값에 +1한다. 보다 간단하게 값을 구할 수 있다.

또 코드를 작성하다보니, 앞쪽에 0을 채우기 위해, 

```java
for (int i = 0; i < nSquare; i++) {

            int target = i ^ (i >> 1);
            String biStr = Integer.toBinaryString(target);
            sb.append(biStr);

            for (int j = biStr.length(); j < n; j++) {

                sb.insert(j,0);

            }

            sb.append("\n");

        }
```
이런 코드를 활용했으나, 맨 처음에만 0 이 추가 되어

0000000<br>
1<br>
11<br>
101<br>
11100<br>

이런식으로 값이 이상해저버림. 이에대한 해결책을 오랜 시간 고민했으나 도출하지 못하고 다시 도움을 요청함.

> **힌트 3**
> sb에 붙이기 전에 biStr 자체를 n자리로 맞춰놓은 다음에 sb에 한 번만 append하면 되지 않을까요? 즉 "한 줄짜리 문자열"을 먼저 완성하고, 그걸 sb에 통째로 넘기는 거죠.
{: .prompt-tip }
- 힌트를 보고, 순서를 바꿔서 문자열의 길이 부터 시작해서, n까지 0을 먼저 채운후에, 나중에 sb에 넣으면 되겠다고 생각해 코드를 번경함.

```java

for (int i = 0; i < nSquare; i++) {

            int target = i ^ (i >> 1);
            StringBuilder biStr = new StringBuilder(Integer.toBinaryString(target));

            for (int j = biStr.length(); j < n; j++) {
                biStr.insert(0, "0");
            }

            sb.append(biStr);
            sb.append("\n");
            
        }

```

## 시도

**1차 시도.**
성공

## 배운 점

- for문을 돌려서 0을 붙히는거는 많이 느릴 줄 알았는데, 그렇지 않았음. Java는 1초에 단순 연산 약 1억(10^8)회 정도 처리하기에, 내가 생각하는것보다 훨씬 빠름.
- 쉬프트 연산을 처음 활용해봤음. 교재같은 곳에서는 봤는데, int nSquare = 1 << n; 처럼 거듭 제곱을 계산할때 간단하게 활용가능함. i >> 1은 2의 나눈 몫으로 활용 가능.
- sb.insert(j, 0)의 j는 sb 전체 기준 위치임. sb에 계속 쌓인다는걸 인지하지 못하고, 계속 앞줄에 0넣기를 반복하니 코드가 의도와는 다르게 동작을 하지 않음.

## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class GrayCode {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb = new StringBuilder();
        int n = Integer.parseInt(br.readLine());
        int nSquare = 1 << n;
 
        for (int i = 0; i < nSquare; i++) {
 
            int target = i ^ (i >> 1);
            StringBuilder biStr = new StringBuilder(Integer.toBinaryString(target));
 
            for (int j = biStr.length(); j < n; j++) {
                biStr.insert(0, "0");
            }
 
            sb.append(biStr);
            sb.append("\n");
 
        }
 
        System.out.println(sb);
 
    }
}

```
