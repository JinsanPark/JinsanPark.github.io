---
title: "CSES - Creating Strings"
date: 2026-05-20 17:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-20 |
| 소요 시간 | 0.13s |

## 요약

- 문제 유형: 중복 문자가 있는 순열 생성
- 핵심 아이디어: 문자 개수를 배열로 관리하면서 재귀적으로 한 글자씩 선택
- 포인트: 선택 후 상태를 복구하는 백트래킹 구조

## 문제 설명

- [문제 링크](https://cses.fi/problemset/task/1623)

- [출처](https://cses.fi/problemset) : CSES Problem Set by Antti Laaksonen

- [라이선스](https://creativecommons.org/licenses/by-nc-sa/4.0/) : Creative Commons BY-NC-SA 4.0

---

## Creating Strings

Given a string, your task is to generate all different strings that can be created using its characters.

### Input
The only input line has a string of length $n$. Each character is between a–z.

### Output
First print an integer $k$: the number of strings. Then print $k$ lines: the strings in alphabetical order.

### Constraints
* $1 \le n \le 8$

### Example
**Input:**
```text
aabac
```

**Output:**
```text
20
aaabc
aaacb
aabac
aabca
aacab
aacba
abaac
abaca
abcaa
acaab
acaba
acbaa
baaac
baaca
bacaa
bcaaa
caaab
caaba
cabaa
cbaaa
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

## 문자열 만들기

문자열이 주어졌을 때, 해당 문자열의 문자들을 사용하여 만들 수 있는 서로 다른 모든 문자열을 생성하는 것이 당신의 임무입니다.

### 입력
유일한 입력 줄에는 길이가 $n$인 문자열이 주어집니다. 각 문자는 a–z 사이의 알파벳 소문자입니다.

### 출력
먼저 문자열의 개수를 나타내는 정수 $k$를 출력합니다. 그다음 $k$개의 줄에 걸쳐 생성된 문자열들을 사전 순으로 출력합니다.

### 제한 사항
* $1 \le n \le 8$

### 예제
**입력:**
```text
aabac
```

**출력:**
```text
20
aaabc
aaacb
aabac
aabca
aacab
aacba
abaac
abaca
abcaa
acaab
acaba
acbaa
baaac
baaca
bacaa
bcaaa
caaab
caaba
cabaa
cbaaa
```

</details>

---

## 초기 접근 및 로직 구상 방식

일단은 개수를 어떻게 구할 수 있을까?

ab/ba는 2개, aab/aba/baa는 3개. 처음엔 그냥 n!인 줄 알았다. 근데 aab가 3개지 6개(3!)가 아니다. a 두 개를 자기들끼리 바꿔도 똑같은 aab니까. 여기서 막힘. 

같은 글자끼리 자리 바꾼 건 같은 문자열.말로는 알겠는데 이걸 식으로 어떻게 빼지?

결국 물어봄. 답은 다중집합 순열: 전체 n!에서 중복된 글자 개수의 팩토리얼로 나눠 과counting을 없앤다. aab면 3!/2! = 3. 
그래서 아까 6이 아니라 3임.

<details markdown="1">
<summary><b> 계산 노가다 (클릭하여 펼치기)</b></summary>
ab
ba
2개

aab
aba
baa
3개

abc
acb
bac
bca
cab
cba
6개

aaabb
aabab
abaab
baaab
aabba
ababa
baaba
abbaa
babaa
bbaaa
10개

aaabc
aabac
abaac
baaac
aabca
abaca
baaca
abcaa
bacaa
bcaaa
aaacb
aacab
acaab
caaab
aacba
acaba
caaba
acbaa
cabaa
cbaaa
a = 1 b = 1
일때 2개 == 2!

a = 2 b = 1
일때 3개 == 2! + 1

a= 1 b = 1 c = 1
일때 6개 == 3!

a = 3 b = 2일때
10개 5 * 4 * 3 * 2 * 1

a = 3 b = 1 c = 1
20개 (a = 3 b = 2일때) * 2
</details>

어떻게 중복없는 배열을 만들것이냐 인데 이거를 풀 알고리즘이나 로직이 떠오르지 않음. 같은 문자끼리 어떻게 비교하는지, list로 하나하나 비교하기에는 반복 횟수가 커지면 너무 느려질거 같음.

> **힌트 1**
> 이 문제는 가짓수 개수만 묻나요, 아니면 가능한 경우를 하나하나 다 만들어 출력해야 하나요? 후자라면 어떤 풀이 방식이 떠오르나요? 그리고 "첫 칸을 정하고 나면 남은 부분은 무슨 문제"가 되는지 말로 표현해보세요.
{: .prompt-tip }

- 가능한 경우를 하나하나 다 만들어 출력해야하는 문제인데, 여기에 알맞는 알고리즘이 딱히 떠오르지 않음. 브루트포스를 생각했으나 틀린답이였음. 브루트포스는 좀 더 큰 틀이고, 이 문제 같은 경우에는 재귀와 백트래킹으로 푼다고 함. 가능한 모든 경우를 빠짐없이 만들어내라 라는 문제에서는 재귀와 백트레킹에 대한 신호라고 한다.


> **힌트 2**
> 재귀 함수 한 번이 책임지는 일을 "____ 한 칸을 정한다"로 본다면, 그 함수 안에서 일어나야 할 일을 세 덩어리로 나눠보세요. 언제 멈추고(출력하고), 무엇을 반복하고, 반복이 끝나면 무엇을 하나요?
{: .prompt-tip }

- 종료 조건, 문자 하나 붙히고, 다시 호출 반복. 반복이 끝나면 출력해줌.

> **힌트 3**
> "이 글자를 이미 썼는지"를 어떻게 알 수 있을까요? 따로 표시를 남기는 방법 말고, 이미 들고 있는 무엇 하나로 판단할 수 있다면 그게 뭘까요? 그리고 그 무엇은 재귀 함수에 어떻게 전달되어야 할까요?
{: .prompt-tip }

- 이미 내가 가지고 있는 alpCount안에 들어있음. 예를 들어 aaabc이면 0번인덱스에 3, 1번인덱스에 1, 2번 인덱스에 1을 가지고 있으니깐 이걸 인자값으로 사용하면 될듯. alpCount자체를 인자로 넘겨버려서 사용하면 될듯함.


> **힌트 4**
> 한 칸에서 한 후보를 쓰고 재귀로 들어갔다가, 돌아오면 그 선택을 되돌려야 해요. 되돌리는 건 버그가 아니라 의도 — 같은 칸에서 다음 후보를 깨끗한 상태에서 다시 시도하려는 거예요. 들어갈 때 한 동작은 돌아올 때 정확히 짝으로 복구한다, 이게 백트래킹의 심장이에요.
{: .prompt-tip }

- 들어갈 때 alpCount[i]--, 돌아올 때 ++. 힌트의 들어갈 때 한 동작을 돌아올 때 짝으로 복구 라는 말에서 풀렸다. alpCount 인덱스가 먼저 다 돌고 그 다음 바깥 i가 증가하니까, 안쪽 ++/--가 짝을 맞춰 상태를 깨끗하게 되돌려서 중복이 안 생긴다. 이걸로 가장 큰 고민이였던 중복된 배열 생성에 관해서 해결함.

## 제출

**1차 제출**

- 성공. 허나 지금 코드는 매반복마다 sout를 해줘서 무척이나 느려보임. 

<details markdown="1">
<summary><b>1차 제출 코드 (클릭하여 펼치기)</b></summary>

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class CreatingStrings {
    public static void main(String[] arg) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String line = br.readLine();
        StringBuilder sb = new StringBuilder();
        int length = line.length();
        char[] arr = line.toCharArray();
        int[] alpCount = new int[26];
        int maxCount = 0; // 팩토리얼 최대값인데 어차피 조건 따지면 8!이니깐 8! = 40320 즉, int로 충분히 담김.
 
        for (int i = 0; i < arr.length; i++) {
            int a = arr[i] - 'a';
            alpCount[a]++;
        }
 
        if (length > 1) {
            maxCount = factorial(line.length()); 
        } else {
            System.out.println(1);
            System.out.println(arr[0]);
            return;
        }
 
        for (int i = 0; i < 26; i++) {
            if (alpCount[i] > 1) {
             maxCount /= factorial(alpCount[i]);
            }
        }
 
        System.out.println(maxCount);
        creatingStrings(length, alpCount,sb);
 
    }
 
    public static void creatingStrings (int length, int[] alpCount, StringBuilder sb ) {
 
        if (sb.length() == length) {
            System.out.println(sb);
            return;
        }
 
        for (int i = 0; i < 26; i++) {
            if (alpCount[i] > 0) {
 
                char c = (char) ('a' + i);
                sb.append(c);
                alpCount[i]--;
                creatingStrings(length, alpCount, sb);
                sb.deleteCharAt(sb.length() - 1);
                alpCount[i]++;
 
            }
        }
    }
 
    public static int factorial (int n) {
        if (n <= 1) {
            return 1;
        }
        return n * factorial(n - 1);
    }
}

```

</details>

**2차 제출**

- 그래서 2차(최종) 제출에서는 함수의 결과만 저장하는 StringBuilder를 또 하나 생성하여 값을 그곳에 저장한후, 함수가 끝난후 그 값들만 따로 출력함. 이에 따라 시간이 0.33초에서 0.13초로 줄어듬.

## 배운 점

- 가능한 모든 경우를 빠짐없이 만들어내라 라는 문제에서는 재귀와 백트레킹에 대한 힌트로 받아드리면 좋다. (백트레킹에 대한 신호. 부분집합, 순열, N-Queen 같은 것들에서도 사용함.)
- StringBuilder도 당연하지만 객체다. 함수 인자로 넘겨줄 수 있다.
- 재귀란 정해야 할 칸의 수 따라 정해지는 동적인 중첩 반복으로 봐도 좋다. 이 문제에서는 문자열의 길이.
- System.out.println을 매 결과마다 부르면, 한 번 호출할 때마다 출력 비용이 매번 듬. 그래서 결과를 StringBuilder에 다 모아 마지막에 한 번만 출력하면 훨씬 빠르다.

## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class CreatingStrings {
    public static void main(String[] arg) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb = new StringBuilder();
        String line = br.readLine();
        int length = line.length();
        char[] arr = line.toCharArray();
        int[] alpCount = new int[26];
        int maxCount = 0;
        StringBuilder result = new StringBuilder();
        br.close();
 
        for (int i = 0; i < arr.length; i++) {
            int a = arr[i] - 'a';
            alpCount[a]++;
        }
 
        if (length > 1) {
            maxCount = factorial(line.length());
        } else {
            System.out.println(1);
            System.out.println(arr[0]);
            return;
        }
 
        for (int i = 0; i < 26; i++) {
            if (alpCount[i] > 1) {
             maxCount /= factorial(alpCount[i]);
            }
        }
 
        System.out.println(maxCount);
        creatingStrings(length, alpCount, sb, result);
        System.out.println(result);
 
    }
 
    public static void creatingStrings (int length, int[] alpCount, StringBuilder sb, StringBuilder result ) {
 
        if (sb.length() == length) {
            result.append(sb).append("\n");
            return;
        }
 
        for (int i = 0; i < alpCount.length; i++) {
            if (alpCount[i] > 0) {
 
                char c = (char) ('a' + i);
                sb.append(c);
                alpCount[i]--;
                creatingStrings(length, alpCount, sb, result);
                sb.deleteCharAt(sb.length() - 1);
                alpCount[i]++;
 
            }
        }
 
    }
 
    public static int factorial (int n) {
        if (n <= 1) {
            return 1;
        }
        return n * factorial(n - 1);
    }
}

```
