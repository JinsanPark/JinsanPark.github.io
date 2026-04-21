---
title: "Missing Number"
date: 2026-04-21 09:57:21 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

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

## 풀이 코드

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

## 초기 접근 방식
- **초기 접근**: 순서가 상관 없어 보여서 HashSet을 쓰는게 빠르고 좋을거 같다고 판담함.
  
- **로직 구상**: 반복문으로 1 ~ n까지 넣고 인풋받은 숫자를 제외한 다음 남은 1개 출력.

## 시도

1차 성공

## AI 조언

```java
long sum = (long) n * (n + 1) / 2; // 1~n 합
StringTokenizer st = new StringTokenizer(br.readLine());
while (st.hasMoreTokens()) {
    sum -= Integer.parseInt(st.nextToken());
}
System.out.println(sum);
```
수학으로 더 간단하게 풀 수 있다.
빠진 숫자 찾기는 수학 공식이 정석.
long으로 캐스팅하는 이유는 n이 최대 2×10⁵이라 int 범위 초과 방지용.

**HashSet 풀이 vs 수학 풀이**

| | HashSet | 수학 |
|---|---|---|
| 시간 | O(n) | O(n) |
| 공간 | O(n) | O(1) |
| 코드 길이 | 길다 | 짧다 |



**얻은 깨달음** :
수학 공식을 사용시 long으로 받아야 오버플로우 안남.
"빠진 숫자 찾기" 유형은 전체 합 - 입력 합 공식이 정석.
굳이 알고리즘과 자료구조라는 사고에 갇혀서 간단한 길을 돌아가지 않도록 최대한 생각해볼것.

