---
title: "CSES - Chessboard and Queens"
date: 2026-05-22 20:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-22 |
| 소요 시간 | 0.06s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1624/)

출처: CSES Problem Set by Antti Laaksonen (https://cses.fi/problemset)

라이선스: Creative Commons BY-NC-SA 4.0

---

## Chessboard and Queens

Your task is to place eight queens on a chessboard so that no two queens are attacking each other. As an additional challenge, each square is either free or reserved, and you can only place queens on the free squares. However, the reserved squares do not prevent queens from attacking each other.

How many possible ways are there to place the queens?

### Input

The input has eight lines, and each of them has eight characters. Each square is either free (.) or reserved (*).

### Output

Print one integer: the number of ways you can place the queens.

### Example
**Input:**
```
........
........
..*.....
........
........
.....**.
...*....
........
```
**Output:**
```
65
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

여덟 개의 퀸을 체스판에 배치하되, 서로 공격할 수 없도록 해야 합니다. 추가 조건으로, 각 칸은 '자유(.)' 또는 '예약된(*)' 상태이며, 퀸은 반드시 '자유' 칸에만 배치할 수 있습니다. '예약된' 칸이라고 해서 퀸의 공격을 막지는 않습니다. 이 조건들을 만족하며 퀸을 배치하는 방법의 총 가지 수를 구하세요.

### 입력

8줄의 문자열이 주어지며, 각 줄은 8개의 문자로 구성됩니다. '.'은 자유 칸, '*'은 예약된 칸을 의미합니다.

### 출력

퀸을 배치할 수 있는 경우의 수를 나타내는 정수 하나를 출력합니다.

### 예제
입력:
........
..*.....
........
........
.....**.
...*....
........
........

출력:
65

</details>

---

## 초기 접근 및 로직 구상 방식

판단한 이유는 일단 체스판의 크기가 가로 세로 8 * 8이라서 충분히 재귀 범위 안에 들어오고, 퀸을 배치 할 수 있는 모든 경우의 수이니깐 재귀라고 판단함.

예약칸은 배치 불가, 허나 공격 경로를 막지는 않는다.
퀸은 상하좌우 그리고 대각선 모두 공격 가능. 
0,0에 배치시 (0 ~ 7, 0) (0, 0 ~ 7) 까지 배치 불가.
대각선은 배치 위치에서 (+1,+1) 씩 해주면 되니깐 (1,1) (2,2) ... (7,7) 까지 배치 불가. 

플래그를 세워서 처리할까 고민. - 불린[][] 이중 배열로 '*' 이랑 퀸 배치해서 공격 불가능한 곳들 true로 바꾸기가 어떨까 생각해봄. - 에 대한 시도는 실패함. 이유는 백트래킹 과정 중에서, '*'같은 공격 불가능한 곳도 다시 false로 돌려버림.

이중 반복문으로 짜보다가, 그냥 한줄을 옮기면 되는거 아닌가 라는 생각해봄 왜냐하면 어차피 한곳에 배치하면 상하좌우 한줄은 못쓰니깐. - 그런데 여기서 다시 저번 처럼 함수 로직을 어떻게 짤것인지에 대해서 막힘 + 대각선 처리도.





## 시도

**1차 제출**

- 성공

## 배운 점

<details markdown="1">
<summary><b>받은 힌트들 (클릭하여 펼치기)</b></summary>


> **힌트 1**
> 막을때 true고, 돌아올때 false로 바꾸는데, '*'은 돌아올때 어떻게 될까?
{: .prompt-tip }

> **힌트 2**
> 같은 행에서 다음 열을 시도할때, 직전 시도가 그대로 남아있으면 어떻게 될까?
{: .prompt-tip }

</details>


- boolean[][]로 true/false(공격받을곳/공격안받을곳)로 시도. 허나 백트래킹이 돌아오면서 '*' 칸까지 false로 풀려버려서 실패. 따라서, 1/0(못놓음/놓을 수 있음)으로 변경. 특히, '*'인 곳은 처음에 1로 초기화하여 백트래킹후에도 0이 되지 않아 재귀가 돌고 돌아도 1 아래로 내려가지 않아 그곳에는 둘 수 없게 했다. 이렇게 겹쳐서 일어날 수 있는 값은 T/F말고 횟수로 세야 수월하게 풀 수 있다.
- 재귀로 들어가면서 퀸으로 놓을 때 공격 받는 곳에 +1, 나오면서 퀸을 치우니깐 공격 받는 횟수-1을 하기 때문에 초기에 1인 값('*')은 재귀를 마치고 돌아오면 1이고, 0인곳은 0이다. 초기값이 돌아오지 않는다면 == 예전 값이 남아있다면, 같은 줄 다음 번 시도에 영향을 준다.
- 못 가는 칸 = 1, 놓을 수 있는 칸 = 0, 예약 칸 = -1 으로 생각함. 그후 '*' 도 어차피 공격 불가능한 칸이니깐, 1 이상으로 생각했다. 다만, '*' 칸과 퀸을 배치할 수 있는 칸의 차이점은 '*'칸은 1 아래로 내려가지 않는다는것. 따라서 0과 1이상으로 나누어도 충분하다.
  
## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class ChessboardAndQueens {

    static int result = 0;

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int[][] chessBoard = new int[8][8];

        for (int i = 0; i < 8; i++) {

            String line = br.readLine();
            char[] lineToChar = line.toCharArray();

            for (int j = 0; j < 8; j++) {
                if (lineToChar[j] == '*') {
                    chessBoard[i][j] = 1;
                }
            }

        }

        chess(chessBoard, 0);
        System.out.println(result);

    }

    public static void chess(int[][] chessBoard, int row) {

        if (row == 8) {
            result++;
            return;
        }

        for (int i = 0; i < 8; i++) {

            if (chessBoard[row][i] == 0) {

                chessBoard[row][i] += 1;

                int k = 1;
                while (row + k < 8) {
                    if (i + k < 8) {
                        chessBoard[row + k][i + k] += 1;
                    }
                    if (i - k >= 0) {
                        chessBoard[row + k][i - k] += 1;
                    }
                    chessBoard[row + k][i] += 1;
                    k++;
                }

                chess(chessBoard, row + 1);

                chessBoard[row][i] -= 1;
                int re = 1;
                while (row + re < 8) {
                    if (i + re < 8) {
                        chessBoard[row + re][i + re] -= 1;
                    }
                    if (i - re >= 0) {
                        chessBoard[row + re][i - re] -= 1;
                    }
                    chessBoard[row + re][i] -= 1;
                    re++;
                }

            }


        }

    }
}

```
