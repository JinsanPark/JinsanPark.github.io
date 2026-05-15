---
title: "CSES - Missing Number"
date: 2026-04-21 09:57:21 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

# 2026 05 05 수정

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-04-21 |
| 소요 시간 | 0.44s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1083/)

You are given all numbers between 1,2,\ldots,n except one. Your task is to find the missing number.

**Input**

The first input line contains an integer n.
The second line contains n-1 numbers. Each number is distinct and between 1 and n (inclusive).

**Output**

Print the missing number.

**Constraints**

$$2 \le n \le 2 \cdot 10^5$$

**Example**

Input:
```
5
2 3 1 5
```

Output:
```
4
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

1부터 $n$ 사이의 모든 숫자 중 하나를 제외한 나머지 숫자들이 주어집니다. 당신의 임무는 빠진 그 하나의 숫자를 찾는 것입니다.

### 입력
첫 번째 줄에는 정수 $n$이 주어집니다.
두 번째 줄에는 $n-1$개의 숫자가 주어집니다. 각 숫자는 모두 서로 다르며, 1과 $n$ 사이(양 끝값 포함)의 값입니다.

### 출력
빠진 숫자를 출력합니다.

### 제한 사항
- $2 \le n \le 2 \cdot 10^5$

### 예제
**입력:**
```
5
2 3 1 5
```

**출력:**
```
4
```
</details>

<br>



## 초기 접근 및 로직 구상 방식

- 순서가 상관 없어 보여서 HashSet을 쓰는게 빠르고 좋을거 같다고 판단함. 반복문으로 1 ~ n까지 넣고 인풋받은 숫자를 제외한 다음 남은 1개 출력으로 구상함.


## 시도

**1차 시도**
- 성공

## 배운 점

- AI에게 더 좋은 방법을 물어본 결과, 수학으로 더 간단하게 풀 수 있다고 한다. 빠진 숫자 찾기는 수학 공식(전체 합 - 입력 합 )이 정석. 
- 굳이 알고리즘과 자료구조라는 사고에 갇혀서 간단한 길을 돌아가지 않도록 최대한 생각해볼것.
  
## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashSet;
import java.util.StringTokenizer;

public class MissingNumber {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        HashSet<Integer> hs = new HashSet<>();
        for (int i = 1; i <= n; i++) {
            hs.add(i);
        }

        StringTokenizer st = new StringTokenizer(br.readLine());

        while (st.hasMoreTokens()) {
            int m = Integer.parseInt(st.nextToken());
            hs.remove(m);
        }

        for (int num : hs) {
            System.out.println(num);
        }

    }
}
```