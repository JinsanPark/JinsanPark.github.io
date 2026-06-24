---
title: "CSES - Knight Moves Grid"
date: 2026-06-17 22:00:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-06-17 |
| 소요 시간 | 0.35s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/3217/)

출처: CSES Problem Set by Antti Laaksonen (https://cses.fi/problemset)

라이선스: Creative Commons BY-NC-SA 4.0

---

There is a knight on an $n \times n$ chessboard. For each square, print the minimum number of moves the knight needs to do to reach the top-left corner.

### Input

The only line has an integer $n$.

### Output

Print the number of moves for each square.

### Constraints

* $4 \le n \le 1000$

### Example

**Input:**
```text
8
```

**Output:**
```text
0 3 2 3 2 3 4 5
3 4 1 2 3 4 3 4
2 1 4 3 2 3 4 5
3 2 3 2 3 4 3 4
2 3 2 3 4 3 4 5
3 4 3 4 3 4 5 4
4 3 4 3 4 5 4 5
5 4 5 4 5 4 5 6
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

$n \times n$ 크기의 체스판 위에 나이트가 있습니다. 체스판의 각 칸에 대해, 해당 칸에서 나이트가 맨 왼쪽 위(좌측 상단) 모서리에 도달하기 위해 필요한 최소 이동 횟수를 출력하십시오.

### 입력

유일한 줄에 정수 $n$이 주어집니다.

### 출력

각 칸에 대해 나이트의 최소 이동 횟수를 격자 형태로 출력합니다.

### 제약 조건

* $4 \le n \le 1000$

### 예제

**입력:**
```text
8
```

**출력:**
```text
0 3 2 3 2 3 4 5
3 4 1 2 3 4 3 4
2 1 4 3 2 3 4 5
3 2 3 2 3 4 3 4
2 3 2 3 4 3 4 5
3 4 3 4 3 4 5 4
4 3 4 3 4 5 4 5
5 4 5 4 5 4 5 6
```

</details>

---

## 초기 접근 및 로직 구상 방식

문제는 0에 도착하려면 몇번 움직여야하는것 이지만, 거꾸로 생각해서, (0,0)에서 몇번 움직여야 그 자리에 도달하는가를 채우면 되는 문제. 나이트는 x축 만큼 2칸 가면 y축으로 1칸 또는 반대로. 따라서 dx dy 방향 사용. 움직일때 n * n 안에서 움직이고, 이미 간곳은 갈 수 없다. 즉, 0인 곳만 갈 수 있다.
또, 나이트의 이동범위를 생각해야 하니깐 bfs로 범위내를 다 넣는게 좋을 듯함.

dx = {1,-1,0,0};
dy = {0,0,1,-1};
로 하려고 했으나, 좌우 2칸 + 상하 1칸. 상하 2칸 + 좌우 1칸을 하기에는 복잡해보임.

따라서 총 8개이니깐
int[] dx = {2, 2, -2, -2, 1, -1, 1, -1};
int[] dy = {1, -1, 1, -1, 2, 2, -2, -2};
이렇게 처리하기로 함.


## 시도

**1차 제출**

- 성공


## 배운 점

- dx[i]와 dy[i]를 같은 인덱스끼리 짝지어 읽는것이 방향 인덱스임.
- 원래는 큐에 {x,y,거리}를 넣었으니 grid[x][y]가 곧 거리임으로 큐에 거리를 포함하지 않아도 됨.
- bfs는 갈 수 있는 거리를 모두 처리한 후에 다음 거리를 가기 때문에, 첫 도착이 곧 최소임. dfs는 한 방향으로 쭉 파고들다 보니 최소 거리가 아닌 거리가 어떤 칸에 먼저 박혀서 고정 될 수 있음.

## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayDeque;
import java.util.Queue;
 
public class KnightMovesGrid {
 
    static int[] dx = {2, 2, -2, -2, 1, -1, 1, -1};
    static int[] dy = {1, -1, 1, -1, 2, 2, -2, -2};
    static int n;
 
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        n = Integer.parseInt(br.readLine());
        br.close();
 
        int[][] grid = new int[n][n];
 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                grid[i][j] = -1;
            }
        }
 
        grid[0][0] = 0;
        knight(grid);
 
        StringBuilder sb = new StringBuilder();
 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                sb.append(grid[i][j]).append(" ");
            }
            sb.append("\n");
        }
 
        System.out.println(sb);
 
    }
 
    public static void knight(int[][] grid) {
 
        Queue<int[]> queue = new ArrayDeque<>();
        queue.add(new int[]{0, 0});
 
        while(!queue.isEmpty()) {
 
            int[] cur = queue.poll();
            int curX = cur[0];
            int curY = cur[1];
            int curDist = grid[curX][curY];
 
            for (int i = 0; i < 8; i++) {
 
                int nextX = curX + dx[i];
                int nextY = curY + dy[i];
                int nextDist = curDist + 1;
 
                if (nextX < n && nextY < n && nextX >= 0 && nextY >= 0 && grid[nextX][nextY] == -1) {
 
                    grid[nextX][nextY] = nextDist;
                    queue.add(new int[]{nextX,nextY});
 
                }
            }
        }
    }
}

```
