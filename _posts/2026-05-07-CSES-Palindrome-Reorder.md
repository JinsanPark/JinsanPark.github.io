---
title: "CSES - Palindrome Reorder"
date: 2026-05-07 19:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-07 |
| 소요 시간 | 0.24s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1755)

---


# Palindrome Reorder

**Time limit:** 1.00 s **Memory limit:** 512 MB

Given a string, your task is to reorder its letters in such a way that it becomes a palindrome (i.e., it reads the same forwards and backwards).

### Input

The only input line has a string of length $n$ consisting of characters A–Z.

### Output

Print a palindrome consisting of the characters of the original string. You may print any valid solution. If there are no solutions, print "NO SOLUTION".

### Constraints

* $1 \le n \le 10^6$

### Example

**Input:**
```
AAAACACBA
```

**Output:**
```
AACABACAA
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

문자열이 주어졌을 때, 해당 문자들의 순서를 재배치하여 팰린드롬(앞으로 읽으나 뒤로 읽으나 같은 문자열)으로 만드는 것이 당신의 임무입니다.

### 입력
유일한 입력 줄에는 알파벳 A–Z로 구성된 길이 $n$의 문자열이 주어집니다.

### 출력
원래 문자열의 문자들로 구성된 팰린드롬을 출력합니다. 가능한 유효한 답이 여러 개라면 그중 아무거나 출력해도 좋습니다. 만약 해결책이 없다면 "NO SOLUTION"을 출력하십시오.

### 제한 사항
* $1 \le n \le 10^6$

### 예제
**입력:**
```
AAAACACBA
```

**출력:**
```
AACABACAA
```

</details>

---

## 초기 접근 및 로직 구상 방식

- 앞으로 읽어도 뒤로 읽어도 같은 문자열을 만들어야 하니함.
기준을 뭘로 세워야 할까 고민하다가, 중앙 기준으로 대칭이면 괜찮지 않을까 라는 생각을 함.
그리고 대칭이면, 알파벳 개수가 똑같을거라고 생각했음.
중앙 기준으로 나누고, 중앙에 들어간 알파벳 제외하고 대칭이 되는지 확인해봄

AABAA
ACBCA
ACBBCA
ACBBBCA

- 그러면 어떤 로직을 써야할지 고민해봄.
우선 입력받은 알파벳의 개수를 세어야 하지 않을까? 하는 고민.
대칭되는것들은 짝수일거고, 대칭 안되는 것들이면 홀수.
만약 개수가 홀수 알파벳이 2개이면 "NO SOLUTION"
홀수인거를 중앙에 배치한다고 생각하고,
모두 짝수이면 뭐가 들어가도 상관 없을거 같다고 판단함.
AABBCC를 ABCCBA CABBAC로 하나, 어차피 똑같으니깐.

- 중앙을 기준으로 반전시켜서 붙힐수는 없나?
아니면 짝수개니깐, /2 개 해서 절반 붙이고 홀수 붙친뒤 절반 뒤에 붙히는 식으로, 순서는 중앙 기준으로 반전시켜서. StringBuilder의 reverse 사용하면 될것 같다고 판단함.

- 그리고 막혔던 부분이 알파벳을 어떻게 샐거냐가 문제였는데, 처음에는 HashMap으로 하려다가, 너무 복잡해질거 같아서, 다른 방법을 생각해봤으나 떠오르지 않아, 힌트를 요청함.


> **힌트 1**
> 이 문제에서 세야 할 알파벳은 A~Z 26종으로 정해져 있다. HashMap이 가진 "아무 키나 받을 수 있다"는 유연함이 이 상황에서 꼭 필요한 기능일까?
{: .prompt-tip }
- 여기서, 여차피 고정이면 배열로 쇼부칠 수 있는거 아닌가 하는 생각이 들었음. int로 배열 26칸 짜리, 26개면 A ~ Z이고, 글자 하나씩 돌면서 c로 받고, c - 'A'하면 0번 인덱스에는 A의 개수, 1번에는 B... 25번에는 Z의 개수가 들어가니깐.



## 시도

**1차 시도.**

성공.

허나 for문이 너무 많고 줄일 수 있는 방법이 없을까 고민해봄.<br>

<details markdown="1">
<summary><b>1차 성공 코드</b></summary>

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class PalindromeReorder {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int[] countAlphabet = new int[26];
        String line = br.readLine();
 
        for (int i = 0; i < line.length(); i++) {
 
            char c = line.charAt(i);
            countAlphabet[c - 'A']++;
 
        }
 
        int countOdd = 0;
 
        for (int i = 0; i < countAlphabet.length; i++) {
            if (countAlphabet[i] % 2 == 1) {
                countOdd++;
            }
        }
 
        if (countOdd >= 2) {
            System.out.println("NO SOLUTION");
            return;
        }
 
        StringBuilder sb = new StringBuilder();
 
        for (int i = 0; i < countAlphabet.length; i++) {
 
            char alphabet = (char) (i + 65);
 
            if (countAlphabet[i] % 2 == 0) {
                for (int j = 0; j < countAlphabet[i] / 2; j++) {
                    sb.append(alphabet);
                }
            }
        }
 
        for (int i = 0; i < countAlphabet.length; i++) {
 
            char alphabet = (char) (i + 65);
 
            if (countAlphabet[i] % 2 == 1) {
                for (int j = 0; j < countAlphabet[i]; j++) {
                    sb.append(alphabet);
                }
            }
        }
 
        StringBuilder sb2 = new StringBuilder();
 
        for (int i = 0; i < countAlphabet.length; i++) {
 
            char alphabet = (char) (i + 65);
 
            if (countAlphabet[i] % 2 == 0) {
                for (int j = 0; j < countAlphabet[i] / 2; j++) {
                    sb2.append(alphabet);
                }
            }
        }
 
        sb2.reverse();
 
        sb.append(sb2);
 
        System.out.println(sb);
 
    }
}
```

</details><br>

```java
        StringBuilder sb2 = new StringBuilder();
        sb2.append(sb);
        sb2.reverse();

        for (int i = 0; i < countAlphabet.length; i++) {

            char alphabet = (char) (i + 65);

            if (countAlphabet[i] % 2 == 1) {
                for (int j = 0; j < countAlphabet[i]; j++) {
                    sb.append(alphabet);
                }
            }
        }

        sb.append(sb2);
```

마지막 for문을 바꾸니, 코드 자체도 줄고, 시간도 0.03초 단축시킴.


## 배운 점

- 자바의 String은 한 번 만들면 내용을 바꿀 수 없기에, s = s + c; 처럼 한 글자 추가하는것도 기존 내용 복사후, 글자를 추가해서 시간과 메모리를 상당히 많이 잡아 먹는다. 최악의 경우 O(N^2) 까지 갈 수 도 있음.
- 알파벳은 26개로 고정되어 있기에, HashMap같은 유연한 도구가 딱히 필요하지 않다. HashMap은 의외로 복잡한 기능을 수행하는데, int[26] 같은 배열은 훨씬 단순하기에, 이런 문제에서 더욱 적합함. 아마, 크기가 정해져있다면, 배열부터 생각하는게 맞을 것 같음.


## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class PalindromeReorder {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int[] countAlphabet = new int[26];
        String line = br.readLine();

        for (int i = 0; i < line.length(); i++) {

            char c = line.charAt(i);
            countAlphabet[c - 'A']++;

        }

        int countOdd = 0;

        for (int i = 0; i < countAlphabet.length; i++) {
            if (countAlphabet[i] % 2 == 1) {
                countOdd++;
            }
        }

        if (countOdd >= 2) {
            System.out.println("NO SOLUTION");
            return;
        }

        StringBuilder sb = new StringBuilder();

        for (int i = 0; i < countAlphabet.length; i++) {

            char alphabet = (char) (i + 'A');

            if (countAlphabet[i] % 2 == 0) {
                for (int j = 0; j < countAlphabet[i] / 2; j++) {
                    sb.append(alphabet);
                }
            }
        }

        StringBuilder sb2 = new StringBuilder();
        sb2.append(sb);
        sb2.reverse();

        for (int i = 0; i < countAlphabet.length; i++) {

            char alphabet = (char) (i + 'A');

            if (countAlphabet[i] % 2 == 1) {
                for (int j = 0; j < countAlphabet[i]; j++) {
                    sb.append(alphabet);
                }
            }
        }

        sb.append(sb2);

        System.out.println(sb);

    }
}


```
