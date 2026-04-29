---
title: "CSES - Two Sets"
date: 2026-04-28 17:50:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-04-29 |
| 소요 시간 | 0.32s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1092)

---

Your task is to divide the numbers $1,2,\ldots,n$ into two sets of equal sum.

### Input
The only input line contains an integer $n$.

### Output
Print "YES", if the division is possible, and "NO" otherwise.

After this, if the division is possible, print an example of how to create the sets. First, print the number of elements in the first set followed by the elements themselves in a separate line, and then, print the second set in a similar way.

### Constraints
* $1 \le n \le 10^6$

### Example 1
**Input:**
```
7
```
**Output:**
```
YES
4
1 2 4 7
3
3 5 6
```

### Example 2
**Input:**
```
6
```
**Output:**
```
NO
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

$1, 2, \ldots, n$의 숫자를 합이 같은 두 개의 집합으로 나누는 것이 목표입니다.

### 입력 (Input)
첫 번째 줄에 정수 $n$이 주어집니다.

### 출력 (Output)
나누는 것이 가능하다면 **"YES"**, 불가능하다면 **"NO"**를 출력합니다.

만약 가능하다면, 그 다음 줄에 집합을 어떻게 구성했는지 출력합니다.
1. 첫 번째 집합의 원소 개수를 출력한 뒤, 다음 줄에 해당 원소들을 출력합니다.
2. 두 번째 집합의 원소 개수를 출력한 뒤, 다음 줄에 해당 원소들을 출력합니다.

### 제약 사항 (Constraints)
* $1 \le n \le 10^6$

---

### 예제 1
**입력:**
```
7
```
**출력:**
```
YES
4
1 2 4 7
3
3 5 6
```

### 예제 2
**입력:**
```
6
```
**출력:**
```
NO
```

</details>

---

## 초기 접근 및 로직 구상 방식

문제는 입력 받은 n값을 2세트로 나눠서 그 세트의 합이 서로 같냐는 물어보는데, 접근을 어떻게 해야할지 몰라, 많은 시간을 허비했다. 따라서 AI에게 힌트를 요청했다.

> **힌트 1**
> 두 그룹의 합이 같으려면, 전체 합이 어떤 조건을 만족해야 할까?
{: .prompt-tip }

```
n = 1 일때 X
n = 2 일때 X
n = 3 일때 1 2 ; 3 ;  합 = 3
n = 4 일때 1 4 ; 2 3 합 5
n = 5 일때 1 2 3 4 5 X 
n = 6 일때 X
n = 7 일때 1 2 4 7 ; 3 5 6 ; 합 = 14
n = 8 일때 1 2 3 4 8 ; 5 6 7 ; 합 = 18

n = 1 일때 전체 합 = 1
n = 2 일때 전체 합 = 3
n = 3 일때 전체 합 = 6 O
n = 4 일때 전체 합 = 10 O
n = 5 일때 전체 합 = 15
n = 6 일때 전체 합 = 21
n = 7 일때 전체 합 = 28 O
n = 8 일때 전체 합 = 36 O
```

이니깐, 전체 합이 짝수일때만 성립한다고 판단했다.

> **힌트 2**
> n부터 1까지 내려오면서 탐욕적으로 수를 고른다면 어떤 기준으로 한 세트에 넣을지 말할 수 있을까.
{: .prompt-tip }

이 힌트로 그리디 라는것을 파악하고, 가장 큰 수 부터 차근차근 넣으면 되겠다고 생각했다.

n = 7이라고 가정해보면,
28 / 2 = 14 - 각각 세트가 14가 되도록 나누어야 하니깐,
14를 채우려면 가장 큰거 부터 하나씩 넣어서 14가 되도록 만들면 되니깐,
7 먼저 넣고, 그 다음 6, 5를 넣으면 18, 4를 넣으면 17, ... 1을 넣으면 14.
1 + 6 + 7 = 14
남은 수는
2 + 3 + 4 + 5 = 14
맞는 듯하다.

그러면 세트1번은 1 6 7
세트 2번은 2 3 4 5
이런식으로 해결하면 될것같다.

## 시도

**1차 시도는 실패했다.**

이유는 StringBulider에 reverse()를 활용하여 출력값 예제와 출력을 똑같이 맞추려고 했으나, reverse() 사용시 StringBulider에 들어간 값이 1234; 이면 4321; 로 나온다는 것을 확인하고, 또 문제에서 출력 순서는 상관이 없게 정답으로 인정해줘서 reverse()가 필요 없다고 판단함.

**2차 시도는 성공했다.**

허나 `sb1.append(" ").append(i);` 에서 1(공백)2(공백)3... 이런 식으로 출력되는데 마음에 안들어 `sb1.append(i).append(" ");` 로 변경했다. 그리고 맨 뒤에 1(공백)..... n(공백) 이런식으로 출력되는게 마음에 들지 않아서

```java
sb1.deleteCharAt(sb1.length() - 1);
sb2.deleteCharAt(sb2.length() - 1);
```
를 추가했다.

또 System.out.println이 5번 반복되어 나중에 병목이 일어날까 고려하여,

```java
StringBuilder answer = new StringBuilder();
answer.append("YES").append("\n");
answer.append(count1).append("\n");
answer.append(sb1).append("\n");
answer.append(count2).append("\n");
answer.append(sb2);

System.out.println(answer);
```

로 변경하였다.

## 배운점

- StringBulider에 reverse()는 내용값도 뒤집어 버린다.
- 또 적어도 이 문제는 아니지만, 왜냐하면 sum이 짝수이여야 하고, target은 항상 1 이상이고, 1부터 n까지 순회.
- 따라서 최소한 1은 sb1에 들어가고, 나머지가 sb2에 들어가는 구조라 비어있을 수 가 없으니깐.

```java
sb1.deleteCharAt(sb1.length() - 1);
sb2.deleteCharAt(sb2.length() - 1);
```

이 부분에서 sb1, sb2가 비어있다면 오류가 발생할 수 있으니, 다른 방안도 모색해 봐야한다.

## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class TwoSets {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        long sum = (long) n * (n + 1) / 2;
        StringBuilder sb1 = new StringBuilder();
        StringBuilder sb2 = new StringBuilder();
        int count1 = 0;
        int count2 = 0;
 
 
        if (sum % 2 != 0) {
            System.out.println("NO");
            return;
        } else {
 
            long target = sum / 2;
 
            for (int i = n; i > 0 ; i--) {
 
                if (target >= i) {
                    sb1.append(i).append(" ");
                    target -= i;
                    count1++;
                } else {
                    sb2.append(i).append(" ");
                    count2++;
                }
 
            }
 
        }
 
        sb1.deleteCharAt(sb1.length() - 1);
        sb2.deleteCharAt(sb2.length() - 1);
 
        StringBuilder answer = new StringBuilder();
        answer.append("YES").append("\n");
        answer.append(count1).append("\n");
        answer.append(sb1).append("\n");
        answer.append(count2).append("\n");
        answer.append(sb2);
 
        System.out.println(answer);
 
    }
}
```

