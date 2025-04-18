---
layout: post
title:  "동적 계획법(Dynamic programming)"
date:   2025-04-03
image: /assets/img/blog/postimage/Algorithm.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

똑같은 계산을 여러 번 반복해야할 때***(중복되는 부분 문제(Overlapping Subproblems))*** 또는 큰 문제의 최적해가, 작은 문제들의 최적해로 이루어질 때***(최적 부분 구조(Optimal Substructure))*** 사용하기 좋은 다이나믹 프로그래밍(DP)에 대해 설명해볼게

![다이나믹프로그래밍](/assets/img/blog/computerscience/다이나믹프로그래밍.png)

## Dynamic Programming 이란?

***최적화 이론***의 한 기술이며, 특정 범위까지의 값을 구하기 위해서 그것과 다른 범위까지의 값을 이용하여 효율적으로 값을 구하는 알고리즘 설계 기법이다.

쉽게 말해, 앞에서 구했던 답을 뒤에서도 이용하고 옆에서도 이용하는 문제해결 패러다임으로 분할 정복 알고리즘과 비슷하다. 다만, 분할정복과 차이가 생기는 부분은 원래의 문제를 부분 문제로 나누는 방식에 있다.

동적 계획법의 경우 주어진 문제를 나눌 때 부분 문제를 최대한 많이 이용하도록 나눈 다음, 주어진 부분 문제의 정답을 한 번만 계산하고 저장해둔 뒤 다시 한 번 이 부분 문제를 이용할 때에는 저장해둔 정답을 바로 산출하여 이용함으로써 속도를 향상시킨다.

## Top-Down 과 Bottum-Up 방식

다이나믹 프로그래밍에는 하향식 접근 방법과 상향식 접근 방법이 존재해

### Top-Down : 재귀 + 메모이제이션(Memoization)

- 위에서 아래로 내려가면서 필요한 값을 계산하고 저장

피보나치 수열을 통해 예시를 보여줄게

예제 : 재귀를 통한 피보나치 수열
~~~python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
~~~

일반 재귀 방식 같은 경우 중복 계산을 많이 해. fib(5)를 한다 했을 때, 아래 재귀에서 반복되는 연산이 있다는 얘기야

예제 : 다이나믹 프로그래밍(메모이제이션, Memoization)을 통한 피보나치 수열
~~~python
memo = {}

def fib(n):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib(n - 1) + fib(n - 2)
    return memo[n]
~~~

다이나믹 프로그래밍의 메모이제이션을 활용하면, 중복 계산 없이 한 번 구한 값을 저장해서 재사용할 수 있어

### Bottom-Up : 반복문 + 타불레이션(Tabulation)

- 아래에서 부터 차곡차곡 값을 쌓아올림

예제 : 다이나믹 프로그래밍(타뷸레이션, Tabulation)을 통한 피보나치 수열
~~~python
def fib(n):
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
~~~

타불레이션 같은 경우 상향식 접근 방법이야

### 상향식과 하향식 마무리

- 차이점

| 항목 | 하향식 (Top-Down) | 상향식 (Bottom-Up) |
|:---:|:---:|:---:|
| 구현 | 재귀 + 메모이제이션 | 반복문 + DP 테이블 |
| 계산 방식 | 큰 문제 → 작은 문제 | 작은 문제 → 큰 문제 |
| 효율성 | 필요한 부분만 계산 | 전부 계산 (필요 없어도) |
| 위험 | 재귀 깊이 초과 가능 | 없음 |
| 직관성 | 더 직관적 | 구현은 조금 더 복잡할 수 있음 |

| 상황 | 추천 방식 |
|:---:|:---:|
| 문제 구조가 재귀적으로 떠오를 때 | 하향식 (Top-down) |
| 입력 크기가 크고 재귀가 깊어질 위험 있을 때 | 상향식 (Bottom-up) |
| 전체 결과 테이블이 꼭 필요한 문제일 때 | 상향식 (Bottom-up) |
| 디버깅이나 점화식이 복잡해서 감 잡기 어려울 때 | 하향식으로 먼저 풀어보고 전환 가능 |

## 중복되는 부분 문제 (Overlapping Subproblems) 와 최적 부분 구조 (Optimal Substructure)

아까 다이나믹 프로그래밍 통해서 중복되는 부분 문제와 최적 부분 구조를 풀 수 있다고 했는데, 이거에 대해서 알려줄게

### 중복되는 부분 문제 (Overlapping Subproblems)

같은 부분 문제를 여러 번 반복해서 풀게 되는 상황을 말함

예제 : 피보나치 수열
~~~python
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)
~~~

fib(5)를 호출하면 내부적으로 다음과 같은 호출들이 생김:
~~~scss
fib(5)
├── fib(4)
│   ├── fib(3)
│   │   ├── fib(2)
│   │   └── fib(1)
│   └── fib(2)
└── fib(3)
    ├── fib(2)
    └── fib(1)
~~~

여기서 fib(2)는 세번이나 계산되고 있지? 이런 중복되는 계산을 피하려고 한 번 계산한 결과를 저장해서 재사용하는 게 DP의 핵심이야.

***중복되는 부분 문제가 있다 = 메모이제이션 하면 성능이 확 올라감***

### 최적 부분 구조 (Optimal Substructure)

문제의 최적해가, 그 하위 문제들의 최적해로 이루어질 수 있는 구조

예제 : 최단 경로 문제

A → B → C로 가는 경로 중 최단 경로를 찾고 싶다고 해보자

- 만약 A → B의 최단 경로를 알고 있고,

- B → C의 최단 경로도 알고 있다면,

A → C의 최단 경로는 A → B + B → C로 구할 수 있게 된다.

이런식으로 큰 문제의 답을, 작은 무제의 답으로부터 만들 수 있다면 최적 부분 구조가 성립이 돼

| 개념 | 의미 | 예시 |
|:---:|:---:|:---:|
| 중복되는 부분 문제 | 같은 계산을 여러 번 하게 됨 | 피보나치에서 fib(2) 여러 번 |
| 최적 부분 구조 | 큰 문제의 최적해 = 작은 문제들의 최적해의 조합 | 최단 경로, 배낭 문제 등 |

## DP 문제 해결 방법

DP 문제 해결 방법에 대해서 가이드 받은걸 공유해 보고자 해

### DP 배열(테이블)을 어떻게 정의할까?

Dynamic Pragramming의 핵심은 ***"DP 배열을 어떻게 정의하느냐"***야

예를 들어, dp[i]가 무엇을 의미할지를 먼저 정의하고 시작해야 해.

- 피보나치: dp[i] = i번째 피보나치 수

- LIS: dp[i] = i번째 원소를 마지막 원소로 하는 LIS의 길이

- 동전 문제: dp[i] = 금액 i를 만들 수 있는 경우의 수

### 상태(State)와 결정(Decision)을 나눠서 생각해보기

- 상태 : 문제의 '진행 상황' (예 : 현제 인덱스, 남은 용량, 남은 금액 등)

- 결정 : 그 상태에서 뭘 선택할 수 있는지 (예 : 물건을 넣을까 말까, 어떤 동전을 사용할까 등)

이렇게 사고해보면 점화식이 나오게 될텐데

### 점화식 세우기

점화식이란? dp[i] = dp[i-1] + dp[i-2] 같은 관계식

예를 들어, 계단 오르기 문제: 한 번에 1칸 또는 2칸 오를 수 있을 때, n칸을 오르는 경우의 수는?
~~~python
dp[n] = dp[n-1] + dp[n-2]
~~~

이런 식으로 문제의 규칙을 점화식으로 정리해줘야 해