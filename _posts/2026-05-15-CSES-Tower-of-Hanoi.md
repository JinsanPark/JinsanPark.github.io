---
title: "CSES - Tower of Hanoi"
date: 2026-05-15 20:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-15 |
| 소요 시간 | 0.09s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/2165)

---

# Tower of Hanoi

The Tower of Hanoi game consists of three stacks (left, middle and right) and $n$ round disks of different sizes. Initially, the left stack has all the disks, in increasing order of size from top to bottom.

The goal is to move all the disks to the right stack using the middle stack. On each move you can move the uppermost disk from a stack to another stack. In addition, it is not allowed to place a larger disk on a smaller disk.

Your task is to find a solution that minimizes the number of moves.

### Input
The only input line has an integer $n$: the number of disks.

### Output
First print an integer $k$: the minimum number of moves.

After this, print $k$ lines that describe the moves. Each line has two integers $a$ and $b$: you move a disk from stack $a$ to stack $b$.

### Constraints
* $1 \le n \le 16$

### Example
**Input:**
```
2
```

**Output:**
```
3
1 2
1 3
2 3
```


<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

하노이의 탑 게임은 세 개의 기둥(왼쪽, 가운데, 오른쪽)과 서로 다른 크기의 $n$개의 원판으로 구성됩니다. 처음에는 모든 원판이 왼쪽 기둥에 위에서부터 아래로 크기가 커지는 순서대로 쌓여 있습니다.

이 게임의 목표는 가운데 기둥을 이용하여 모든 원판을 오른쪽 기둥으로 옮기는 것입니다. 각 차례마다 기둥의 가장 위에 있는 원판을 다른 기둥으로 옮길 수 있습니다. 단, 큰 원판을 작은 원판 위에 놓는 것은 허용되지 않습니다.

당신의 임무는 이동 횟수를 최소화하는 해결 방법을 찾는 것입니다.

### 입력
유일한 입력 줄에는 원판의 개수를 나타내는 정수 $n$이 주어집니다.

### 출력
먼저 최소 이동 횟수 $k$를 출력합니다.
그 후, 이동 과정을 설명하는 $k$개의 줄을 출력합니다. 각 줄에는 두 개의 정수 $a$와 $b$가 포함되며, 이는 원판을 $a$번 기둥에서 $b$번 기둥으로 옮겼음을 의미합니다.

### 제한 사항
* $1 \le n \le 16$

### 예제
**입력:**
```
2
```

**Output:**
```
3
1 2
1 3
2 3
```

</details>

---

## 초기 접근 및 로직 구상 방식

스택 사용하면 될듯
맨 왼쪽에 n부터 큰거부터 넣는다.
각각 스택에서 옮기면서 작은거 위해 큰거 못 올리니깐, 넣기전에 스택 맨 위에 있는 거랑,
집어 넣을거랑 비교해주고 넣기.

<details markdown="1">
<summary><b>하노이탑 계산 (클릭하여 펼치기)</b></summary>

n == 1 이면

```
1 0 0
```

```
0 0 1
```
1번

n == 2이면, 

```
1
2
```

```
2 1 0
```

```
0 1 2
```

```
    1
0 0 2
```

4번

n = 3 이면
```
1
2
3 0 0
```

```
2
3 0 1 
```

```
3 2 1
```

```

  1
3 2 0
```

```
  1
0 2 3
```

```
1 2 3
```

```
    2
1 0 3
```

```
    1
    2
0 0 3
```
8 번

n = 4
```
1
2
3
4 0 0
```

```
2
3
4 1 0
```

```
3
4 1 2
```

```
3   1
4 0 2
```

```
    1
4 3 2
```

```
1 
4 3 2 
```

```
1 2
4 3 0
```

```
  1
  2
4 3 0
```

```
  1
  2
0 3 4
```

```
  2 1
0 3 4
```

```
    1
2 3 4
```

```
1 
2 3 4
```

```
1   3
2 0 4
```

```
    3
2 1 4
```

```
    2
    3
0 1 4
```

``` 
    1 
    2
    3
0 0 4
```

16번

1 2 4 8 16
2^n번
맨 처음꺼 제외하면 2^n -1 같음.

</details>

근데 문제가 옮기는 걸 어떻게 카운팅 하고, 어떻게 최소인지 판단할지, 그게 제일 문제인데, 이 부분을 긴 시간동안 해결하지 못해, 힌트 요청함.


> **힌트 1**
>'어떻게 옮길지'와 '왜 최소인지'는 사실 따로 풀 문제가 아니라 같은 답에서 함께 풀리는 문제예요. 둘 다 이미 적어둔 트레이스 안에 단서가 있어서, 그 구조 한 번만 잡히면 동시에 해결됩니다. n=4 트레이스를 다시 펴놓고, 16개 상태의 가운데 어디쯤에서 한 번 끊어보세요. 끊은 앞쪽과 뒤쪽이 각각 어디서 본 적 있는 트레이스랑 닮아 보이지 않나요?
{: .prompt-tip }

```
  1
  2
0 3 4
```

- 여기서 부터 n = 3일때랑 유사해 보임. 4는 지금부터 절대 움직일 필요가 없고, 중앙의 막대를 왼쪽으로 옮기면,

```
1
2
3 0 4
```

n = 3일때와 똑같다.

> **힌트 2**
> 기둥 셋에 이름이 붙어있긴 하지만 "시작 기둥 / 도착 기둥 / 보조 기둥"이라는 역할만 있을 뿐이고, 누가 어느 역할을 맡느냐는 그때그때 다르 다. n=4 문제 한 개가 "n=3 한 번 → 원반 4 한 번 옮김 → n=3 한 번"으로 정확히 쪼개집니다. 횟수가 2·(2^3 - 1) + 1 = 15 = 2^4 - 1로 떨어지는 것도 이 구조의 결과. 함수 하나가 자기보다 작은 같은 모양의 문제 두 개를 호출하는 구조를 사용하면 된다.
{: .prompt-tip }

- 힌트를 보고 재귀함수 사용 결정. 허나 막대기를 계속 추적하려고 하니, 이 부분에서 또 막혀버림.

> **힌트 3**
> anoi(?, ?, ?, n-1)의 세 인자 자리에, 지금 함수가 받은 depart/via/arrive 중 무엇이 어디에 들어가야 할까요? 특히 "보조 기둥" 자리에는 뭐가 들어갈지 짚어보세요.
{: .prompt-tip }

- hanoi(왼, 오, 중, n -1 ) 이거는 왼쪽에 있는 수를 맨 오른쪽 기둥을 통해서, 중앙으로 옮긴다.(중, 왼, 오, n-1) 이거는 중앙에 있는 수를 왼쪽을 통해서 오른쪽으로 옮긴다.
그리고 이 처음 호출하는 hanoi(n,0,0,n); 요 함수의 인자값을 어떻게 줘야할지 모르겠어서 다시 힌트 요청함.


> **힌트 4**
> 문제에서 기둥은 1, 2, 3번으로 번호가 붙어 있어요(출력도 그 번호로 찍어야 하고요). 그러니 depart/via/arrive는 기둥 번호이지 원반 개수가 아니에요. n이 거기 들어가면 안 되고, 0도 기둥 번호가 아니에요. 처음 호출에서 모든 원반은 1번 기둥에 있고 3번 기둥으로 옮겨야 하죠. 그럼 depart, via, arrive에는 각각 어떤 숫자가 들어가야 할까요?
{: .prompt-tip }

- 음 호출하는 hanoi()에 1,2,3,n 이 들어가면 된다고 판단함.

## 시도

**1차 시도.**

- 성공

## 배운 점

- 맞긴 했는데, 한번 더 풀어봐야할거 같음.
- 왼쪽/중앙/오른쪽이라는 고정에서 출발/경유/도착으로 유동적으로 바라보기
- 큰 문제 안에서 같은 문제를 발견 했을 때는 재귀 함수로 풀어보기.
- 어느 기둥에 몇 개 있는지 변수로 들고 다니려 했는데, 재귀에서는 그게 필요 없었다. 함수가 몇 개를 어디서 어디로라는 인자로 불리는 순간 그 호출의 상태는 인자 안에 다 들어있으니깐.
- 원반 n을 옮기려면 그 위 n-1개가 반드시 보조 기둥에 모여 있어야 하고(다른 자리엔 둘 데가 없으니까), 옮긴 뒤엔 그 n-1개를 다시 도착지로 가져와야 해서 2^n - 1이 최소임. 

## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class TowerOfHanoi {

    static StringBuilder sb = new StringBuilder();

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int result = (1 << n) - 1;
        hanoi(1,2,3,n);

        System.out.println(result);
        System.out.println(sb);

    }

    public static void hanoi(int depart, int via, int arrive, int n) {

        if (n == 0) {
           return;
        }

        hanoi(depart,arrive,via, n - 1);

        sb.append(depart).append(" ").append(arrive).append("\n");

        hanoi(via, depart, arrive, n - 1);

    }

}

```
