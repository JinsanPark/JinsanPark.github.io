---
title: "CSES - Grid Coloring I"
date: 2026-06-25 01:30:30 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-06-25 |
| 소요 시간 | 0.15s |

## 문제 설명

- [문제 링크](https://cses.fi/problemset/task/3311)

- [출처](https://cses.fi/problemset) : CSES Problem Set by Antti Laaksonen

- [라이선스](https://creativecommons.org/licenses/by-nc-sa/4.0/) : Creative Commons BY-NC-SA 4.0

---

You are given an $n \times m$ grid where each cell contains one character `A`, `B`, `C` or `D`.

For each cell, you must change the character to `A`, `B`, `C` or `D`. The new character must be different from the old one.

Your task is to change the characters in every cell such that no two adjacent cells have the same character.

### Input

The first line has two integers $n$ and $m$: the number of rows and columns.

The next $n$ lines each have $m$ characters: the description of the grid.

### Output

Print $n$ lines each with $m$ characters: the description of the final grid.

You may print any valid solution.

If no solution exists, just print `IMPOSSIBLE`.

### Constraints

* $1 \le n, m \le 500$

### Example

**Input:**
```text
3 4
AAAA
BBBB
CCDD
```

**Output:**
```text
CDCD
DCDC
ABAB
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

※ 원문을 한국어로 번역하였습니다.

각 칸에 `A`, `B`, `C`, `D` 중 하나의 문자가 들어 있는 $n \times m$ 크기의 격자가 주어집니다.

각 칸에 대해, 문자를 `A`, `B`, `C`, `D` 중 하나로 변경해야 합니다. 이때 새롭게 변경한 문자는 원래 있던 문자와 반드시 달라야 합니다.

당신의 임무는 서로 인접한 어떤 두 칸도 같은 문자를 갖지 않도록 모든 칸의 문자를 변경하는 것입니다.

### 입력

첫 번째 줄에 행과 열의 수를 나타내는 두 정수 $n$과 $m$이 주어집니다.

그 다음 $n$개의 줄에는 각각 격자의 상태를 나타내는 $m$개의 문자가 주어집니다.

### 출력

최종 격자의 상태를 나타내는 $m$개의 문자로 이루어진 $n$개의 줄을 출력합니다.

올바른 답이 여러 개 존재한다면 그중 아무거나 출력해도 좋습니다.

만약 만족하는 방법이 존재하지 않는다면 `IMPOSSIBLE`을 출력합니다.

### 제약 조건

* $1 \le n, m \le 500$

### 예제

**입력:**
```text
3 4
AAAA
BBBB
CCDD
```

**출력:**
```text
CDCD
DCDC
ABAB
```

</details>

---

## 초기 접근 및 로직 구상 방식

처음에는 그리드를 채우는 문제라 BFS를 사용하기로 함.
dx, dy 배열로 방향을 정하고, 큐에 x,y 좌표와 입력한 글자로 채운 배열을 넘긴다.
2개 만들어서 기존 배열과 비교하면서 문제 조건에 맞게 새로운 배열을 만든다.
코드를 짜다가 복잡해져서, 도움을 요청하기로 함.



> **힌트 1**
> 현재 칸을 채우는 데 필요한 이웃이, 큐 없이 단순 순회만으로도 이미 정해져 있지 않은지?
{: .prompt-tip }

이중 반복문으로 새로운 배열을 채울때, 조건에 맞게(좌,상, 기존 배열과 같은 글자가 아닌지)에 맞게 채우다 보면, 어차피 조건에 맞게 채워져서 BFS를 쓸 필요가 전혀 없다.


기존 배열이

```
AAAA
BBBB
CCDD
```

일때

[0][0]에는 A가 아닌 B가 들어가고, [0][1]에는 좌의 B와 기존배열의 A가 아닌 C, [0][2]에는 A도 아니고 좌의 C도 아닌 B, [0][3]에는 기존 배열의 A, 좌의 들어간 B도 아닌 C가 들어가 BCBC가 먼저 채워진다.

[1][0]에는 기존 배열의 B, 상단의 B가 아닌 A가, [1][1]에는 기존 배열 B, 좌측의 A, 상단의 C가 아닌 D... 이런식으로 반복해서 들어가도 문제 조건에 맞게 할 수 있다.


## 제출

**1차 제출**

- 성공

## 배운 점

- 행 우선으로 채우면 좌·상 이웃이 현재 칸보다 먼저 정해지므로, 따로 큐를 만들어 BFS로 돌릴 필요가 없다. 필요한 이웃이 단순 순회로는 먼저 안 정해질 때에는, BFS를 사용한다.
- 비교 대상이, 현재칸 + 좌측칸 + 상단칸으로 총 3칸 각각의 칸이 A,B,C를 가진다 하더라도, D가 무조건 들어갈 수 있다. 즉, 조건은 3가지, 주어진 건 4가지임으로, IMPOSSIBLE 은 따로 출력안해도 무방하다.


## 최종 풀이 코드

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class GridColoring {

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int n = Integer.parseInt(st.nextToken());
        int m = Integer.parseInt(st.nextToken());
        StringBuilder sb = new StringBuilder();

        char[][] curMatrix = new char[n][m];
        char[][] newMatrix = new char[n][m];


        for (int i = 0; i < n; i++) {
            String line = br.readLine();
            for (int j = 0; j < m; j++) {
                curMatrix[i][j] = line.charAt(j);
            }
        }

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {

                char c = curMatrix[i][j];

                for (int k = 0; k < 4; k++) {

                    char abcd = (char)('A' + k);

                    if ( j - 1 >= 0 && newMatrix[i][j - 1] == abcd || i - 1 >= 0 && newMatrix[i - 1][j] == abcd || c == 'A' + k){
                        continue;
                    }

                    newMatrix[i][j] = abcd;
                    sb.append(abcd);
                    break;
                }
            }
            sb.append("\n");
        }

        System.out.println(sb);

    }
}

```
