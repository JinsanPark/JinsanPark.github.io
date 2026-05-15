---
title: "CSES - Two Knights"
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
| 제출 일자 | 2026-04-28 |
| 소요 시간 | 0.08s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1072/)


---
# Two Knights

Your task is to count for $k=1,2,\ldots,n$ the number of ways two knights can be placed on a $k \times k$ chessboard so that they do not attack each other.

### Input
The only input line contains an integer $n$.

### Output
Print $n$ integers: the results.

### Constraints
$1 \le n \le 10000$

### Example
**Input:**
```
8
```
**Output:**
```
0
6
28
96
252
550
1056
1848
```
---


## 초기 접근 및 로직 구상 방식

- 체스판에 나이트 2개 서로 공격 못하게하는 경우의 수 같음.
DFS나 BFS 사용하면 될거 같음.
나이트는 직선 2칸 이동후 좌 또는 우로 이동.
즉, 나이트 끼리 직선 2칸 이동후 겹치는 수만 빼주면 될거 같음.
공식은 k * (k-1) / 2; 에서 여기서 나이트 이동 범위에 있는 것들 마저 빼주면 정답이 나오지 않을까? 
dx = {1,-1,0,0}; dy = {0,0,1,-1} 을 써야할듯.
dx 2칸 이동후 dy 확인.
dy 2칸 이동후 dx 확인.
이미 갔던 곳도 체크 해야하고 visitied[][] = true false 쓰고.
그런데 시간복잡도가 충분할까 모르겠네.

수학 공식이 따로 있을까?
k = 1 이면 차는 0
k = 2 이면 차는 6
k = 3 이면 차는 8 
k = 4 이면 차는 24
k = 5 이면 차는 48
k = 6 이면 차는 80
k = 7 이면 차는 120
k = 8 이면 차는 168

k가 3일때
2 * 4 = 8

k가 4일때
4 * 6 = 24

k가 5일때
6 * 8 = 48

k가 6일때
8 * 10 = 80

k가 7일때
10 * 12 = 120

k가 8일때
12 * 14 = 168

변수 하나 만들어서
규칙을 봤을때,

x * (x + 2) 하면 될듯.

k = 1이랑 k = 2일때는 따로 빼버리고.

## 시도

1차 시도 성공.

## 배운점
- 이번는 k가 2일때 차가 6이 나오는 부분때문에 쉽사리 규칙을 찾지 못했음.
이 부분 넘겨서 차가 8부터 나오는 부분부터 생각해 봤다면 훨씬 수월하게 풀었을듯.
그냥 많이 틀려보고, 손으로 계산하고, 기하학(도형)적으로 생각해보는게 좋을듯함.

## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class TwoKnights {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int x = 2;
        StringBuilder sb = new StringBuilder();

        for (int i = 1; i <= n ; i++) {

            long square = (long)i * i;

            if (i == 1) {
                sb.append(0).append("\n");
            } else if (i == 2) {
                sb.append( square * (square - 1) / 2 ).append("\n");
            } else {
                sb.append( (square * (square - 1) / 2) - ((long) x * (x + 2)) ).append("\n");
                x += 2;
            }

        }

        System.out.println(sb);

    }
}
```

