---
title: "CSES - Trailing Zeros"
date: 2026-05-08 19:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-01 |
| 소요 시간 | 0.05s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1618/)

---

# Trailing Zeros

Your task is to calculate the number of trailing zeros in the factorial $n!$.

For example, $20! = 2432902008176640000$ and it has 4 trailing zeros.

### Input
The only input line has an integer $n$.

### Output
Print the number of trailing zeros in $n!$.

### Constraints
* $1 \le n \le 10^9$

### Example
**Input:**
```
20
```

**Output:**
```
4
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

당신의 임무는 팩토리얼 $n!$의 끝에 붙는 0의 개수를 계산하는 것입니다.

예를 들어, $20! = 2432902008176640000$ 이며, 이 숫자의 끝에는 4개의 0이 있습니다.

### 입력
유일한 입력 줄에는 정수 $n$이 주어집니다.

### 출력
$n!$의 끝에 붙는 0의 개수를 출력합니다.

### 제한 사항
* $1 \le n \le 10^9$

### 예제
**입력:**
```
20
```

**출력:**
```
4
```

</details>

---

## 초기 접근 및 로직 구상 방식

팩토리얼계산후 맨뒤에 0의 개수인데
10^9 인데 이걸 팩토리얼 계산하면 너무 크고, 배열에 넣는것도 말이 안되는거 같아서 규칙을 먼저 찾아보기로 함.

1 ~ 4 | 0 없음.
5! 12'0'
6! 72'0'
7! 504'0'
8! 4032'0'
9! 36288'0'

10! 36288'00'
14! 871782912'00'

15! 1307674368'000'
19! 121645100408832'000'

20! 243290200817664'0000'
24! 62044840173323943936'0000'


규칙이 딱 봐도 있어 보임.
5로 나눈 몫의 값 같음.
5 ~ 9 까지 0은 1개
10 ~ 14까지 0은 2개
15 ~ 19까지 0은 3개
20 ~ 24까지 0은 4개

값을 5로 나누면 된다고 판단함.


## 시도

**1차 시도.**

실패.
값이 커지니깐 문제 요구 출력값이랑 다르게 나옴.


<details markdown="1">
<summary><b>1차 실패 코드</b></summary>

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class TrailingZeros {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        System.out.println(n / 5);
 
    }
}
```

</details><br>

**2차 시도.**

n = 100부터면

93326215443944152681699238856266700490715968264381621468592963895217599993229915608941463976156518286253697920827223758251185210916864000000000000000000000000
24개

104까지 0이 24개

n = 105
25개

n = 110
00000000000000000000000000
26개

0이 중간에 한번 불어나는 구간이 있는데,

25! 15511210043330985984'000000'
29! 8841761993739701954543616'000000'

30! 26525285981219105863630848'0000000'
34! 29523279903960414084761860964352'0000000'

35! 103331479663861449296666513375232'00000000'
39! 203978820811974433586402817399028973568'00000000'

n = 45 119622220865480194561963161495657715064383733760000000000
기존에 세운 규칙이라면 0이 9개여야 하는데 45에서는 10개. 25에서도 5가 아니라 6

n = 50 12개


25 (5 ^ 2)에서 0 한개 더 추가
50에서 0한개 더 추가

25마다 0하나씩 추가인가?

125면 31개

625면 156개

5의 거듭제곱?

n = 625;

625 / 5 = 125
625 / 25 = 25
625 / 100 = 6

5의 거듭제곱으로 나누니깐 답이 나오는거 같은데 왜인지는 모르겠네.

<details markdown="1">
<summary><b>2차 성공 코드</b></summary>

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class TrailingZeros {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int square = 5;
        int result = n / 5;
 
        while (n >= square) {
            result += n / (square * 5);
            square *= 5;
        }
 
        System.out.println(result);
 
    }
}

```

</details><br>

성공함. 근데 왜 5의 거듭제곱으로 나누면 답이 나오는지 고심해 봤으나, 답이 나오지 않아서, AI에게 물어봄.

> **힌트 1**
> 끝자리에 0이 붙으려면 그 수가 무엇으로 나누어떨어지고, 그 수를 소인수분해 하면 뭐가 나오는지.
{: .prompt-tip }
- 끝자리에 0이 나오려면 10으로 나눠야 하고, 10은 2와 5로 이루어져 있음.

> **힌트 2**
> 2와 5의 배수는 어느게 더 많은지, 1 ~ 10 까지 2의 배수와 5의 배수를 세어보면 알 수 있다. 그렇다면 2와 5를 짝지어 10을 만들 때, 쌍의 개수를 결정짓는 건 둘 중 어느 쪽인가요? 부족한 쪽일까요, 남아도는 쪽일까?"
{: .prompt-tip }
- 2의 배수가 더 많음. 쌍의 개수를 결정짓는건 부족한쪽. 따라서 0의 개수를 결정하는건 5이다.

> **힌트 3(리팩토링 관련)**
> 25 안에 소인수 5가 몇 개 들어있는가?
{: .prompt-tip }
- 2개가 들어있음. 아까 발견한 규칙인 '25 (5 ^ 2)에서 0 한개 더 추가' 에서 왜 0이 하나 더 붙는지 이해함.


> **힌트 4(리팩토링 관련)**
> n을 5로 계속 나누면 몇 번 만에 0이 되나?"
{: .prompt-tip }
- n = 625
625 / 5 = 125
125 / 5 = 25
25 / 5 = 5
5 / 5 = 1
총합 = 156

- 따라서 square없이 코드를 더 줄일 수 있어보임.


## 배운 점

- 끝자리 0의 개수 = 소인수 2와 5의 쌍 개수. 2가 항상 남으므로 5만 세면 충분.
- int로도 풀리긴 했으나, 이런 계산 문제에서는 int의 범위를 넘는지 안넘는지 매번 고민하지 말고, 차라리 long으로 접근하는게 좋아보임.
- 누적 변수 없이 반복문안에서 해결할 수 없나 다시 한번 생각해보기.
## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class TrailingZeros {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int result = 0;
 
        while (n > 0) {
            n /= 5;
            result += n;
        }
 
        System.out.println(result);
 
    }
}
```
