---
title: "CSES - Coin Piles"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-06 |
| 소요 시간 | 0.32s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1754/)

---

# Coin Piles

You have two coin piles containing $a$ and $b$ coins. On each move, you can either remove one coin from the left pile and two coins from the right pile, or two coins from the left pile and one coin from the right pile.

Your task is to efficiently find out if you can empty both the piles.

### Input

The first input line has an integer $t$: the number of tests.

After this, there are $t$ lines, each of which has two integers $a$ and $b$: the numbers of coins in the piles.

### Output

For each test, print "YES" if you can empty the piles and "NO" otherwise.

### Constraints

* $1 \le t \le 10^5$
* $0 \le a, b \le 10^9$

### Example

**Input:**
```
3
2 1
2 2
3 3
```

**Output:**
```
YES
NO
YES
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>


# 동전 무더기 (Coin Piles)

$a$개와 $b$개의 동전이 들어있는 두 개의 동전 무더기가 있습니다. 각 차례마다 당신은 다음 두 가지 작업 중 하나를 수행할 수 있습니다.

*   왼쪽 무더기에서 동전 **1개**를 제거하고, 오른쪽 무더기에서 동전 **2개**를 제거합니다.
*   왼쪽 무더기에서 동전 **2개**를 제거하고, 오른쪽 무더기에서 동전 **1개**를 제거합니다.

당신의 임무는 두 무더기를 모두 정확히 비울 수 있는지 효율적으로 판별하는 것입니다.

### 입력
첫 번째 줄에는 테스트 케이스의 개수 $t$가 주어집니다.
그 후 $t$개의 줄이 이어지며, 각 줄에는 두 개의 정수 $a$와 $b$가 주어집니다. 이는 각 무더기에 있는 동전의 개수를 의미합니다.

### 출력
각 테스트 케이스에 대해, 두 무더기를 비울 수 있다면 "YES"를, 비울 수 없다면 "NO"를 출력합니다.

### 제한 사항
*   $1 \le t \le 10^5$
*   $0 \le a, b \le 10^9$

### 예제
**입력:**
```
3
2 1
2 2
3 3
```

**출력:**
```
YES
NO
YES
```

</details>

---

## 초기 접근 및 로직 구상 방식

- 단순 연산으로 2빼고 1빼고 한번씩 반복문으로 계산하면 시간내에 못할거 같음.
2 1 가능
2 2 불가능 1개 남음
3 3 가능 
4 1 불가능 2개 남음
4 2 가능 
5 1 불가능 3 0
6 1 불가능 4 0
5 2 불가능 1 0
5 3 불가능 3 1 -> 1 0
5 4 -> 3 3 가능

- 일단 합이 3의 배수여야 하는거 같음.5 1은 합이 6인데 2 1 을 해버리면 한쪽이 비어서 3 0 이되어 버리니 불가능 10 2. 이런 케이스들 해결 못해 힌트 요청
  


> **힌트 1**
> 한 번의 연산이 두 더미에 미치는 영향을 생각해보기
{: .prompt-tip }

- 10 2 면 2번 돌릴 수 있는데, 2 번 돌리면 6 0 임.

> **힌트 2**
> 두 더미를 (a, b), a ≥ b 라고 둘 때 — 작은 쪽 b가 "고갈되기 전까지" 큰 쪽 a를 줄일 수 있는 양의 한계를 b에 대한 식으로 써보면 어떻게 될까요? 그리고 그 한계 안에서 a를 0까지 끌어내리려면 a와 b 사이에 어떤 부등식이 성립해야 하는지?
{: .prompt-tip }

계산해보면, 하나는 2, 하나는 1만큼 줄어드니깐,
2빼면 다음에 1빼고 4빼면 다음거에는 2빼고...
즉, 작은 수의 크기만큼 큰수에서 2를 뺄 수 있다.

- 큰 수 <= 2 * 작은 수 라는 공식 도출함.
10 2는 10 <= 4 불가능
3 3, 6 6 같은, 3의 배수이고 a b 서로 같은 수 이면 가능
4 2 는 4 <= 4 이니 가능
5 4 는 5 <= 8 이니 가능



## 시도

**1차 시도.**

성공.


## 배운 점

- 연산 한 번의 효과가 비대칭이고, 어느 쪽이 먼저 0 닿는가를 생각해보면, 생각보다 쉬운 문제였음.
- 규칙을 찾았다고 바로 제출하지 말고, 그 규칙이 작은 케이스에서도 끝까지 성립하는지 한 번 더 따라가 보는 습관 필요함.


## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class CoinPiles {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int t = Integer.parseInt(br.readLine());
        StringBuilder sb = new StringBuilder();
        int max = 0;
        int min = 0;

        for (int i = 0; i < t; i++) {

            StringTokenizer st = new StringTokenizer(br.readLine());

            int a = Integer.parseInt(st.nextToken());
            int b = Integer.parseInt(st.nextToken());

            if (a > b) {
                max = a;
                min = b;
            } else {
                max = b;
                min = a;
            }

            if ((max + min) % 3 == 0) {
                if (max <= 2 * min) {
                    sb.append("YES");
                } else {
                    sb.append("NO");
                }
            } else {
                sb.append("NO");
            }
            sb.append("\n");
        }

        System.out.println(sb);

    }
}


```
