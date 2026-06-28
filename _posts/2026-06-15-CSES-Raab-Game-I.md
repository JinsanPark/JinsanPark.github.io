---
title: "CSES - Raab Game I"
date: 2026-06-15 17:50:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-06-15 |
| 소요 시간 | 0.19s |

## 문제 설명

- [문제 링크](https://cses.fi/problemset/task/3399/)

- [출처](https://cses.fi/problemset) : CSES Problem Set by Antti Laaksonen

- [라이선스](https://creativecommons.org/licenses/by-nc-sa/4.0/) : Creative Commons BY-NC-SA 4.0

- 이 게시물은 CC BY-NC-SA 4.0을 따릅니다

---

Consider a two player game where each player has $n$ cards numbered $1, 2, \dots, n$. On each turn both players place one of their cards on the table. The player who placed the higher card gets one point. If the cards are equal, neither player gets a point. The game continues until all cards have been played.

You are given the number of cards $n$ and the players' scores at the end of the game, $a$ and $b$. Your task is to give an example of how the game could have played out.

### Input

The first line contains one integer $t$: the number of tests.  

Then there are $t$ lines, each with three integers $n, a$ and $b$.

### Output

For each test case print `YES` if there is a game with the given outcome and `NO` otherwise.  

If the answer is `YES`, print an example of one possible game. Print two lines representing the order in which the players place their cards. You can give any valid example.

### Constraints

* $1 \le t \le 1000$
* $1 \le n \le 100$
* $0 \le a, b \le n$

### Example

**Input:**
```text
5
4 1 2
2 0 1
3 0 0
2 1 1
4 4 1
```

**Output:**
```text
YES
1 4 3 2
2 1 3 4
NO
YES
1 2 3
1 2 3
YES
1 2
2 1
NO
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

두 명의 플레이어가 각각 $1, 2, \dots, n$의 번호가 매겨진 $n$장의 카드를 가지고 하는 게임을 생각해보자. 매 턴마다 두 플레이어는 자신의 카드 중 한 장을 테이블에 내려놓는다. 더 높은 숫자의 카드를 낸 플레이어가 1점을 획득한다. 두 카드의 숫자가 같다면 아무도 점수를 얻지 못한다. 게임은 모든 카드가 사용될 때까지 계속된다.

카드의 수 $n$과 게임이 끝났을 때 두 플레이어의 점수 $a$와 $b$가 주어진다. 당신의 임무는 이 게임이 실제로 어떻게 진행되었을 수 있는지 보여주는 예시를 하나 제시하는 것이다.

### 입력

첫 번째 줄에는 테스트 케이스의 개수 $t$가 주어진다.  

그 다음 $t$개의 줄에는 각 테스트 케이스마다 세 개의 정수 $n, a, b$가 주어진다.

### 출력

각 테스트 케이스에 대해, 주어진 결과가 나올 수 있는 게임이 존재한다면 `YES`를, 존재하지 않는다면 `NO`를 출력한다.  

만약 답이 `YES`라면, 가능한 게임 진행 예시를 출력한다. 이어서 두 플레이어가 카드를 낸 순서를 나타내는 두 줄을 출력해야 한다. 조건을 만족하는 올바른 예시라면 어떤 것을 출력해도 무방하다.

### 제약 조건

* $1 \le t \le 1000$
* $1 \le n \le 100$
* $0 \le a, b \le n$

### 예제

**입력:**
```text
5
4 1 2
2 0 1
3 0 0
2 1 1
4 4 1
```

**출력:**
```text
YES
1 4 3 2
2 1 3 4
NO
YES
1 2 3
1 2 3
YES
1 2
2 1
NO
```

</details>

---

## 초기 접근 및 로직 구상 방식

초기에 결과가 주어지고 그 과정중 하나를 찾아야 하니깐 브루트 포스라고 판단함.
반복으로 처리하려다가 n의 최대 100부터 -1씩 줄여가며 찾으니, 최악에는 시간 복잡도가 O(n!)로 느릴것 같다고 판단해, 배열이 아니라 바로 StringBuilder의 넣는 방식으로 구상함.

<details markdown="1">
<summary><b>손계산 노가다 부분 1 (클릭하여 펼치기)</b></summary>

```
n = 3일때

a 1 2 3
b 1 2 3
a 0, b 0

a 1 2 3
b 1 3 2
a 1, b 1

a 1 2 3
b 2 1 3
a 1, b 1

a 1 2 3
b 2 3 1
a 1, b 2

a 1 2 3
b 3 1 2
a 2, b 1

a 1 2 3
b 3 2 1
a 1, b 1
```

</details>

위 계산에 따르면 a,b에서 같은 값이 나오는 순간 동점이 나올 수 밖에 없다. 또한 안나온 쌍들의 공통점으로는 0이 한곳에 몰려있고, a,b의 값을 확인했을때 (0,0)(1,1)(1,1)(1,2)(2,1)(1,1)으로 규칙성이 있어보임.

(1,1) 이 나올때는 같은 수의 카드를 1번 제출 했을때(세가지의 수중 1가지의 수만 같고, 나머지는 다를때),
1,2,3이 페어로 있으니깐 (1,1)은 총 3번 나올 수 있다.
그렇다면 (1,2) or (2,1)은? 3가지 수가 서로 다를때, (0,0)은 전부 같은 수를 냈을때 나올 수 있음.

<details markdown="1">
<summary><b>손계산 노가다 부분 (클릭하여 펼치기)</b></summary>

```
n = 2

a 1 2
b 1 2

a 1 2
b 2 1
```

```
n = 4

a 1 2 3 4
b 1 2 3 4
(0,0)

a 1 2 3 4
b 2 3 4 1
(1,3)

a 1 2 3 4
b 3 4 1 2
(2,2)

a 1 2 3 4
b 4 1 2 3
(3,1)
```

</details>

즉, 한번 회전할때 마다 a가 1씩 증가, b는 1씩 감소.
이 규칙에 만족하면 "YES"출력 후 예시 한개 출력
만족 안하면 "NO"출력.
0회전은 0 부터 출발한다고 가정.

<details markdown="1">
<summary><b>예제 부분 계산 (클릭하여 펼치기)</b></summary>

예제로

1 4 3 2
2 1 3 4

출력

1 2 3 4
1 3 4 2

a + b 만큼만 회전 시키면 된다.
n - a+b 만큼 고정.

1 2 3 4

1 3 4 2

n = 4 a = 1 b = 2

n - (a + b)로 0번 인덱스 고정


1 2 3 4
1 4 2 3

고정한 배열을 하나 만들고, 나머지 회전하는 배열을 만들어서 두개를 이어 붙히기.

</details>

## 시도

**1차 제출 성공 .**

<details markdown="1">
<summary><b>1차 제출 코드(클릭하여 펼치기)</b></summary>

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;
 
public class RaaBGameI {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int t = Integer.parseInt(br.readLine());
 
        for (int i = 0; i < t; i++) {
 
            StringTokenizer st = new StringTokenizer(br.readLine());
            int n = Integer.parseInt(st.nextToken());
            int a = Integer.parseInt(st.nextToken());
            int b = Integer.parseInt(st.nextToken());
 
            int[] line = new int[n];
            int[] lineB = new int[n];
            StringBuilder sbA = new StringBuilder();
            StringBuilder sbB = new StringBuilder();
 
            if (a + b > n) {
                System.out.println("NO");
                continue;
            }
 
            for (int j = 0; j < n; j++) {
                line[j] = j + 1;
                sbA.append(line[j]).append(" ");
            }
 
            for (int j = 0; j < n - (a + b); j++) {
                lineB[j] = j + 1;
                sbB.append(lineB[j]).append(" ");
            }
 
            for (int j = n - (a + b); j < n; j++) {
                lineB[j] = line[(j - (n-(a+b)) + a) % (a+b) + (n-(a+b))];
                sbB.append(lineB[j]).append(" ");
            }
 
            int countA = 0;
            int countB = 0;
 
            for (int j = 0; j < n; j++) {
 
                if (line[j] < lineB[j]) {
                    countB++;
                } else if (line[j] > lineB[j]) {
                    countA++;
                }
 
            }
 
            if (a == countA && b == countB) {
                System.out.println("YES");
                System.out.println(sbA);
                System.out.println(sbB);
            } else {
                System.out.println("NO");
            }
 
        }
    }
}

```



</details>

- 허나 lineB[j] = line[(j - (n-(a+b)) + a) % (a+b) + (n-(a+b))]; 이 부분이 상당히 복잡해 보여서 따로 변수를 만들기로 함.

```java
            int fixed = n - (a + b);
            int rolled = a + b;
            int startRoll = 0;
 
            for (int j = n - (a + b); j < n; j++) {
                startRoll = j - fixed;
                lineB[j] = line[((startRoll + a) % rolled) + fixed];
                sbB.append(lineB[j]).append(" ");
            }
```

- 좀 더 가독성 있게, 무슨 역할인줄 명확하게 수정함.

## 배운 점

- lineB[j] = line[(j - (n-(a+b)) + a) % (a+b) + (n-(a+b))]; 이런식으로 난잡하게 정리하니, 각 수가 무엇을 나타내는 것인지 상당히 알아채기 힘들어, 식을 짜는데 굉장히 애를 먹음. 허나, 각 변수를 빼서 역할을 정리하고 변수명을 명확하게 만들니, lineB[j] = line[((startRoll + a) % rolled) + fixed]; 같이 어떻게 [] 내부가 굴러가는지 알 수 있게됨.
- (startRoll + a) % rolled로 구간 안에서 먼저 굴리고 그 결과에 + fixed로 밀어 인덱스를 이동시켜준다.
  

## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;
 
public class RaaBGameI {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int t = Integer.parseInt(br.readLine());
 
        for (int i = 0; i < t; i++) {
 
            StringTokenizer st = new StringTokenizer(br.readLine());
            int n = Integer.parseInt(st.nextToken());
            int a = Integer.parseInt(st.nextToken());
            int b = Integer.parseInt(st.nextToken());
 
            int[] line = new int[n];
            int[] lineB = new int[n];
            StringBuilder sbA = new StringBuilder();
            StringBuilder sbB = new StringBuilder();
 
            if (a + b > n) {
                System.out.println("NO");
                continue;
            }
 
            for (int j = 0; j < n; j++) {
                line[j] = j + 1;
                sbA.append(line[j]).append(" ");
            }
 
            for (int j = 0; j < n - (a + b); j++) {
                lineB[j] = j + 1;
                sbB.append(lineB[j]).append(" ");
            }
 
            int fixed = n - (a + b);
            int rolled = a + b;
            int startRoll = 0;
 
            for (int j = n - (a + b); j < n; j++) {
                startRoll = j - fixed;
                lineB[j] = line[((startRoll + a) % rolled) + fixed];
                sbB.append(lineB[j]).append(" ");
            }
 
            int countA = 0;
            int countB = 0;
 
            for (int j = 0; j < n; j++) {
 
                if (line[j] < lineB[j]) {
                    countB++;
                } else if (line[j] > lineB[j]) {
                    countA++;
                }
 
            }
 
            if (a == countA && b == countB) {
                System.out.println("YES");
                System.out.println(sbA);
                System.out.println(sbB);
            } else {
                System.out.println("NO");
            }
 
        }
    }
}
```

