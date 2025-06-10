---
layout: post
title:  "Page Replacement Policy"
description: >
 무한대로 보이게 하는 가상 메모리를 구현할때, 물리적인 한계에 부딛혀 메모리에서의 희생양을 찾아야 한다. 희생양은 어떻게 찾을까?
date:   2025-06-10
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

## 왜 Page Replacement Policy가 필요한가?

가상 메모리는 **프로세스마다 "거의 무한대"**로 보이는 주소 공간을 제공하지만, 실제 물리 프레임 수는 제한적이다.

1. **페이지 폴트**가 발생해 새 페이지를 적재해야 할 때

2. **여유 프레임이 없으면** 커널은 한 프레임을 비워야 합니다.

3. 어떤 페이지를 희생(victim) 으로 내보낼지를 정하는 규칙이 **Page Replacement Policy**입니다. 목표는 **디스크 I/O 횟수(=페이지 폴트율)** 를 최소화해 전체 성능을 높이는 것


## 정책 설계 시 고려 요소

| 관점 | 설명 |
|:---|:---|
| 효율성 | 낮은 페이지 폴트율, 적은 CPU 오버헤드 |
| 공정성 | 여러 프로세스가 공유할 때 특정 프로세스만 희생되지 않도록 |
| Stack Property | 물리 프레임 수를 늘리면 절대 페이지 폴트가 늘지 않는 성질 (OPT·LRU가 가짐) |
| 실현 가능성 | 하드웨어 지원(Reference/Dirty 비트 등) 유무, 구현 복잡도 |
| 전역 vs 로컬 | 	전역(global) – 전체 시스템에서 victim 선정,<br/>로컬(local) – fault를 낸 프로세스의 working set 안에서만 선정 |

## 대표 알고리즘

### 이론적 최적 (OPT/MIN)

가장 **먼 미래**에 참조될 페이지를 제거.

- 실현 불가능 → 벤치마킹 용 lower bound.

### FIFO (First-In First-Out)

들어온 순서대로 제거. 구현 간단하지만 **Belady’s Anomaly** 발생(프레임을 늘려도 폴트가 증가 가능).

### Second-Chance / Clock

FIFO 큐＋“최근 참조(return bit=A)” 검사

1. 큐 헤드 검사 → A=1이면 0으로 clear 하고 뒤로 회전

2. A=0인 페이지를 victim

- 간단·저렴, LRU 근사

### LRU (Least Recently Used)

가장 오래 사용되지 않은 페이지 제거 — **Stack Property** 보장

- 정확 LRU는 하드웨어 타임스탬프가 필요해 현실적으로 **근사(Approx. LRU)** 를 사용 :

    - **N-비트 Aging**: 주기마다 R 비트를 상위로 쉬프트

    - **카운터 기반**: 하드웨어가 마지막 접근 시각 기록(32/64 비트)

    - **인접 시계(Enhanced Clock/Clock-Pro)** 등

### LFU / NFU (Least Frequently Used / Not Frequently Used)

누적 참조 횟수 기반. 오래 살아남은 “cold yet used” 페이지가 묶여서 교체되지 않는 문제(aging 필요).

### Working-Set / WSClock

- **Working Set (Δ)**: 최근 Δ 참조 내에 사용된 페이지 집합을 유지

- **WSClock**: Clock 큐에 timestamp를 더해 Working-Set을 근사 → 디스크 쓰기 I/O 분산

### Random

무작위 victim. LRU 추적 비용이 극도로 부담될 때 간단히 사용(예: 일부 GPU MMU)

## Belady’s Anomaly

스택 속성을 갖지 않는 FIFO 계열에서 보이는 현상

- 3 프레임보다 4 프레임이 더 많은 페이지 폴트를 낼 수 있음 → **정책 선택 시 주의**

## 실제 OS 사례

| OS | 구현 요지 |
|:---|:---|
| Linux (5.x) | “Multi-gen LRU”<br/>- Active/Inactive 목록, 페이지 hot/cold 구분<br/>- `kswapd` 가 백그라운드로 스캔, cold & dirty 우선 정리 |
| Windows | **Working-Set + Clock-type** 아키텍처. 프로세스마다 WS 크기 자동 조절, 전역 밸런서가 과도 점유 시 회수 |
| PintOS Project 3 | Clock 알고리즘을 직접 구현하도록 유도. `struct frame` 리스트 ＋ 핸드 포인터 유지, `pml4_is_accessed()` 로 R 비트를 확인해 second-chance 부여 |

## 성능 분석 방법

1. **Trace 재생**: 학습용 메모리 참조 시퀀스를 돌려 폴트 카운트 비교

2. **실험 변수**: 프레임 수, Δ(working-set 윈도), R/Dirty 비트 리셋 주기

3. **Thrashing 탐지**: 폴트 율 급증 + CPU 사용률 ↓ → 정책·resident set 튜닝 필요

## 요약

- **Page Replacement Policy** = 희생 페이지 선택 규칙

- 이상적 OPT는 불가 → LRU 근사 또는 Clock 계열이 실용적

- 정책마다 **복잡도·메모리 오버헤드·성능·공정성** trade-off 존재

- 실제 커널은 **다단계 큐 + 참조/더티 비트** 로 LRU 근사를 구현하고, background daemon(kswapd, balance-set) 으로 지속 튜닝

- PintOS 같은 교육용 OS에서는 Clock (Second-Chance) 구현이 가장 흔하며, 정확한 R/Dirty 비트 처리·프레임 테이블 관리·swap I/O 동기화가 핵심 포인트