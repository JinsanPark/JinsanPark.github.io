---
title: "CSES - Apple Division"
date: 2026-05-21 15:00:00 +0900
categories: [CSES, Introductory Problems]
tags: [java, cses, 문제풀이]
math: true
---

## 문제 정보

| 항목 | 내용 |
|------|------|
| 시간 제한 | 1.00s |
| 메모리 제한 | 512MB |
| 제출 일자 | 2026-05-20 |
| 소요 시간 | 0.13s |

## 요약

- 문제 유형: 두 그룹으로 나눴을 때 합의 최소 차를 구한다.
- 핵심 아이디어: 한쪽만 채우면 안되고 두 그룹 모두 채워 모든 경우의 수를 비교해야한다
- 포인트: 문제 조건에서 n은 최대 20. 즉, 2^20가지. 이 크기는 재귀로 충분히 커버가 가능하다.

## 문제 설명

[문제 링크](https://cses.fi/problemset/task/1623)
출처: CSES Problem Set by Antti Laaksonen (https://cses.fi/problemset)
라이선스: Creative Commons BY-NC-SA 4.0

---

## Apple Division
There are $n$ apples with known weights. Your task is to divide the apples into two groups so that the difference between the weights of the groups is minimal.

### Input
The first input line has an integer $n$: the number of apples.  
The next line has $n$ integers $p_1, p_2, \dots, p_n$: the weight of each apple.

### Output
Print one integer: the minimum difference between the weights of the groups.

### Constraints
* $1 \le n \le 20$
* $1 \le p_i \le 10^9$

### Example
**Input:**
```text
5
3 2 7 4 1
```

**Output:**
```text
1
```

<details markdown="1">
<summary><b>🇰🇷 한국어 번역 (클릭하여 펼치기)</b></summary>

무게를 아는 $n$개의 사과가 있습니다. 당신의 임무는 두 그룹의 무게 차이가 최소가 되도록 사과를 두 그룹으로 나누는 것입니다.

### 입력
첫 번째 입력 줄에는 사과의 개수를 나타내는 정수 $n$이 주어집니다.  
다음 줄에는 각 사과의 무게를 나타내는 $n$개의 정수 $p_1, p_2, \dots, p_n$이 주어집니다.

### 출력
정수 하나를 출력합니다: 두 그룹의 무게 차이의 최솟값.

### 제한 사항
* $1 \le n \le 20$
* $1 \le p_i \le 10^9$

### 예제
**입력:**
```text
5
3 2 7 4 1
```

**출력:**
```text
1
```

</details>

---

## 초기 접근 및 로직 구상 방식

처음에는 투 포인터로 생각함.

전부 합친 수를 개수 / 2로 나누고, 포인터 2개 넣고 2개의 포인터가 가리키는 두 수의 합이 가까우면? 두 포인터가 만나는 지점에서 합쳐서 작은 쪽에 넣기로 구상

1 2 3 4 5. 같은 수의 배열이나, 3 2 7 4 1 같은 배열에는 짠 코드가 맞게 동작했으나,
1 2 4 8 같은 배열에서는 위에 방식대로 사용할 수 없었음.

## 시도

**1차 제출**

- 실패


<details markdown="1">
<summary><b>투포인터 실패 코드 (클릭하여 펼치기)</b></summary>

```java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.StringTokenizer;
 
public class AppleDivision {
    public static void main(String[] args) throws IOException {
 
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int[] line = new int[n];
        boolean[] isUsed = new boolean[n];
        StringTokenizer st = new StringTokenizer(br.readLine());
        int sum = 0;
 
        for (int i = 0; i < n; i++) {
            line[i] = Integer.parseInt(st.nextToken());
            sum += line[i];
        }
 
        br.close();
 
        int pointLeft = 0;
        int pointRight = n - 1;
        int sumDiv = sum / 2;
        int groupA = 0;
        int groupB = 0;
 
        Arrays.sort(line);
 
        while (pointLeft != pointRight) {
 
            if (!isUsed[pointLeft] && !isUsed[pointRight] && sumDiv == line[pointLeft] + line[pointRight]) {
 
                if (groupA < groupB) {
                    groupA = line[pointLeft] + line[pointRight];
                } else {
                    groupB = line[pointLeft] + line[pointRight];
                }
 
                isUsed[pointLeft] = true;
                isUsed[pointRight] = true;
                pointLeft = 0;
                pointRight--;
 
            } else if (!isUsed[pointLeft] && !isUsed[pointRight] && pointRight - pointLeft == 1) {
 
                if (groupA <= groupB) {
                    groupA = line[pointLeft] + line[pointRight];
                } else {
                    groupB = line[pointLeft] + line[pointRight];
                }
 
                isUsed[pointLeft] = true;
                isUsed[pointRight] = true;
                pointLeft = 0;
                pointRight--;
 
            } else {
                pointLeft++;
            }
 
        }
 
        pointLeft = 0;
 
        while (pointLeft < n) {
 
            if (!isUsed[pointLeft]) {
                if (groupB > groupA) {
                    groupA += line[pointLeft];
                    isUsed[pointLeft] = true;
                } else {
                    groupB += line[pointLeft];
                    isUsed[pointLeft] = true;
                }
            }
 
            pointLeft++;
 
        }
 
        System.out.println(Math.abs(groupA - groupB));
 
    }
}

```
</details>


> **힌트 1**
> "사과 하나하나가 A냐 B냐를 20번 결정한다"를 코드로 어떻게 펼치느냐인데 — 사과 i를 결정하고 나면, 그 다음에 해야 할 일이 사과 i+1에 대해 똑같은 결정을 하는 거잖아요. 같은 모양의 일이 반복되죠.
{: .prompt-tip }

- 같은 일을 반복하는건 재귀함수인데, 이 코드 문제를 읽고는 모든 경우를 빠짐없이 만들어내라고 느끼지 못했음. 결정적인 단서가 n ≤ 20에서, 2^20은 충분히 재귀로 가능하니깐 재귀로 풀면 된다고함. 저번 StringCreating에서는 문자열을 붙히는 거였고, 이번 문제는 문자열에 붙히는게 아닌 그냥 숫자만 계속 더해주면 된다고 생각이 들었음. 차이점은 저번 문제는 문자를 땟다가 붙히는것이였고, 이번 문제는 두갈래로 나눠주는것.


**2차 제출(최종) **

- 성공.

재귀 함수를 A그룹, B그룹으로 나누어 각각 A에 배열의 숫자를 넣으며 계산, 두번째 함수에서는 B에 배열의 숫자를 넣으며 계산했음.


## 배운 점

- 문제 조건의 크기를 보고 재귀를 판단 할 수 있어야함.
- 투포인터를 사용했는데,[1, 2, 4, 8] 같은 배열을 1차 제출 코드로 돌려보면 A에 [4,8] B에 [1,2]가 들어가 차가 9가 나옴. 실제로는 A에 [1,2,4] B에 [8] 이 들어가(혹은 반대로) 그룹의 최소 차가 1이 나와야함.
- 재귀가 머리속으로 안그려지면 손으로 따라가 보는게 좋음.

## 최종 풀이 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class AppleDivision {

    static int min = Integer.MAX_VALUE;

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int[] line = new int[n];
        StringTokenizer st = new StringTokenizer(br.readLine());
        long groupA = 0;
        long groupB = 0;

        for (int i = 0; i < n; i++) {
            line[i] = Integer.parseInt(st.nextToken());
        }

        br.close();

        int i = 0;
        division(groupA,groupB,line, i);
        System.out.println(min);

    }

    public static void division(long A, long B, int[] line, int i) {

        if (i == line.length) {
            if (min > Math.abs(A - B)) {
                min = (int) Math.abs(A - B);
            }
            return;
        }

        division(A + line[i],B,line,i + 1);
        division(A,B + line[i] , line,i + 1);

    }
}


```
