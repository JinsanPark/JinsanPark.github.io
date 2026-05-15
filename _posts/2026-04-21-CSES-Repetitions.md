---
title: "CSES - Repetitions"
date: 2026-04-23 20:00:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-04-22 |
| 소요 시간 | 0.18s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1069/)


---

# Repetitions

You are given a DNA sequence: a string consisting of characters A, C, G, and T. Your task is to find the longest repetition in the sequence. This is a maximum-length substring containing only one type of character.

### Input
The only input line contains a string of $n$ characters.

### Output
Print one integer: the length of the longest repetition.

### Constraints
- $1 \le n \le 10^6$

### Example
**Input:**
```
ATTCGGGA
```

**Output:**
```
3
```

## 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class Repetitions {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String inputLine = br.readLine();
        char[] line = inputLine.toCharArray();
        int n = line.length;
        int curCount = 1;
        int biggestCount = 0;
 
        for (int i = 0; i < n; i++) {
 
            if (i < n - 1 && line[i] == line[i + 1]) {
 
                curCount++;
 
            } else if (i < n - 1 && line[i] != line[i + 1]) {
 
                curCount = 1;
 
            }
 
            if (curCount > biggestCount) {
 
                biggestCount = curCount;
 
            }
 
        }
 
        System.out.println(biggestCount);
 
 
    }
 
}
```

## 초기 접근 방식
- **초기 접근**: 배열, 연속된 글자중 가장 긴 글자의 개수를 출력.
  
- **로직 구상**: 인덱스 0부터 돌면서 count++; (같은 글자시) 기존보다 count 크면 count 교체;

## 시도

1차 성공

## AI 조언

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class Repetitions {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String inputLine = br.readLine();
        char[] line = inputLine.toCharArray();
        int n = line.length;
        int curCount = 1;
        int biggestCount = 0;
 
        for (int i = 0; i < n; i++) {
 
            if (i < n - 1 && line[i] == line[i + 1]) {
 
                curCount++;
 
            } else  {
 
                curCount = 1;
 
            }
 
            if (curCount > biggestCount) {
 
                biggestCount = curCount;
 
            }
 
        }
 
        System.out.println(biggestCount);
 
 
    }
 
}
```

 else if (i < n - 1 && line[i] != line[i + 1]) 에서 어차피
 line[i] == line[i + 1] 여기서 걸러지기 때문에 굳이 쓸 필요가 없다.



**얻은 깨달음** :
else if 작성할때 위의 조건식을 한번 생각해보자.
