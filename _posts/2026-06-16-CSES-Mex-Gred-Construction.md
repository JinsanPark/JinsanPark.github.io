---
title: "CSES - Mex Grid Construction"
date: 2026-06-16 18:30:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-06-16 |
| 소요 시간 | 0.09s |

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/3419/)

[출처](https://cses.fi/problemset) : CSES Problem Set by Antti Laaksonen

[라이선스](https://creativecommons.org/licenses/by-nc-sa/4.0/) : Creative Commons BY-NC-SA 4.0

---

Your task is to construct an $n \times n$ grid where each square has the smallest nonnegative integer that does not appear to the left on the same row or above on the same column.

### Input

The only line has an integer $n$.

### Output

Print the grid according to the example.

### Constraints

* $1 \le n \le 100$

### Example

**Input:**
```text
5
```

**Output:**
```text
0 1 2 3 4
1 0 3 2 5
2 3 0 1 6
3 2 1 0 7
4 5 6 7 0
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

당신의 임무는 각 칸이 같은 행의 왼쪽이나 같은 열의 위쪽에 등장하지 않는 가장 작은 음이 아닌 정수(0 이상의 정수)를 가지는 $n \times n$ 격자를 구성하는 것입니다.

### 입력

유일한 줄에 정수 $n$이 주어집니다.

### 출력

예제 형식에 맞게 격자를 출력합니다.

### 제약 조건

* $1 \le n \le 100$

### 예제

**입력:**
```text
5
```

**Output:**
```text
0 1 2 3 4
1 0 3 2 5
2 3 0 1 6
3 2 1 0 7
4 5 6 7 0
```

</details>

---

## 초기 접근 및 로직 구상 방식

대각선 0을 기준으로 대칭이고, 0의 위쪽을 기준으로 봤을때, 열이 0이 아니고, 짝수번 인덱스면 행인덱스 + 열인덱스, 홀수면 열인덱스 - 행인덱스로 생각함. [i][0] [0][j] 는 0 ~ n 까지 채우고, [i][i]는 0으로 채움. 또 대칭이니 [i][j]의 값을 [j][i]에도 넣으면 될것이라고 판단함.


## 시도

**1차 제출**

<details markdown="1">
<summary><b>1차 제출 코드 (클릭하여 펼치기)</b></summary>

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
 
public class MexGridConstruction {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        StringBuilder sb = new StringBuilder();
 
        int[][] matrix = new int[n][n];
 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
 
                matrix[i][i] = 0;
                matrix[0][j] = j;
                matrix[i][0] = i;
 
                if (i > 0 && j > 0 && i < j) {
                    if (j % 2 == 0) {
                        matrix[i][j] = i + j;
                        matrix[j][i] = i + j;
                    } else {
                        matrix[i][j] = j - i;
                        matrix[j][i] = j - i;
                    }
                }
 
                sb.append(matrix[i][j]).append(" ");
 
            }
            sb.append("\n");
        }
 
        System.out.println(sb);
 
    }
}

```

</details>


- 실패.
n == 5일때는 성공 했으나, 케이스가 커지니 실패함. 
n == 6일때 2 3 0 1 6 3 이런식으로 3이 한줄에 2개씩 출력됨.

> **힌트 1**
> 정의를 그대로 옮기는 쪽으로 가볼까요? 그 방향이면, 한 칸 (i,j)의 값을 정하려면 무엇무엇을 먼저 모아야 하는지부터 한 문장으로 적어보시겠어요?
{: .prompt-tip }

그렇다면 지금 인덱스[i][j] 기준으로, 위쪽은 [i-1][j] 로 시작해서 i가 0까지 왼쪽은 [i][j - 1]로 시작해서  j가 0까지. 또 사용한 숫자를 담을 것은 boolean 배열을 활용해서, 1을 사용했으면 1번 인덱스를 true로 바꾸고, 칸이 바뀔때 마다 boolean을 초기화 해줌.

**2차 시도**
- 위 처럼 로직에 맞추어 코드를 수정후 성공함.


## 배운 점

- 처음 보는 문제에서 가장 먼저 꺼내야 할 것은 공식부터 추측하기 전에 정의대로 한 번 만들어보는 것. 문제 예제에 맞는 규칙에 짜맞추다 보니, n = 5 에서만 맞는 코드를 작성함. 작은 예제에서 통과한다고 규칙이 맞는 건 아니라는 것, 더 키워서 반례를 찾는 습관을 들여야함.
  
## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class MexGridConstruction {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());

        StringBuilder sb = new StringBuilder();

        int[][] matrix = new int[n][n];

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {

                boolean[] used = new boolean[n + n];

                for (int k = i - 1; k >= 0; k--) {
                    used[matrix[k][j]] = true;
                }

                for (int k = j - 1; k >= 0 ; k--) {
                    used[matrix[i][k]] = true;
                }

                for (int k = 0; k < n + n; k++) {
                    if (!used[k]) {
                        matrix[i][j] = k;
                        break;
                    }
                }

                sb.append(matrix[i][j]).append(" ");

            }
            sb.append("\n");
        }

        System.out.println(sb);

    }
}
```
