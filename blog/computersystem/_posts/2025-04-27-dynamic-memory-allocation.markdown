---
layout: post
title:  "수작업으로 Malloc 구현(9.9장)"
description: >
 C언어로 직접 Malloc을 구현해보자
date:   2025-04-27
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

말록 구현에 앞서, **명시적 할당자(Explicit allocator)**와 **묵시적 할당자(Implicit allocator)**에 대해 알아 보자 

| | Explicit allocator | Implicit allocator |
|:---:|:---:|:---:|
| 사용 예 | C언어에서 malloc 할당과 free | Java의 garbage collection, ML, and Lisp |

- free 수행에 관해서 직접적인지 또는 간접적인지에 따라서 달라진다

**Malloc**에 관하여서는 이전에 포스팅한 [동적 메모리 할당(Dynamic memory allocation)]{:.heading.flip-title} 을 참조하면 된다

여기서 다룰 내용은 **Explicit allocator**인 malloc 할당을 직접 구현할 예정이다

## Malloc 할당 

![말록 할당](/assets/img/blog/computerscience/mallocallocation.png)

위 사진의 순서 같은 경우 아래와 같다

1. p1(int사이즈X4) 할당

2. p2(int사이즈X5) 할당

3. p3(int사이즈X6) 할당

4. p2 free

5. p4(int사이즈X2) 할당

### 응용 프로그램 

- 응용프로그램 같은 경우 마음대로 malloc 할당과 free 요청이 가능하다

- free 할 시에는 malloc된 block이어야 한다 

### Allocator

명시적 할당기들은 아래와 같은 엄격한 제한사항 내에서 동작해야 함

- 할당된 블럭의 갯수와 크기에 대한 관리 권한이 없다

- malloc 요청에 즉각 반응해야 함

    - 이 뜻은 추후 할당에 대해 대기가 불가능하다는 것이다

- free된 메모리에서만 할당이 가능하다

- block의 크기에 대해서 정렬 요구사항에 맞추어야 한다

    - 리눅스 기준 아래와 같다
    
    -  x86 : 8byte, x86-64 : 16byte 

- free블록에 관해서만 조작이 가능하다

- 일단 malloc할당 되었으면, 움직일 수 없다

    - 이 말은 압축이 불가능하다는 이야기

### 용어 정리

![블록 용어 정리](/assets/img/blog/computerscience/blockdefinition.png)

- Throughput : 시간당 완료된 요청의 갯수 (5000의 malloc할당과 5000의 free요청을 10초 안에 완수하면, 1000operations/second)

- Payload : 블럭 안에서 실제로 사용할 Data

- Overhead : 블럭 안에서 payload 제외한 나머지(header, footer, padding)

- Aggregate payload : 할당된 블럭들의 Data들의 총합

- 최고 이용도(Peak utilization) : 할당기가 힙을 얼마나 효율적으로 사용하는지를 규정하는 많은 방법들 중 하나

    - Pk는 현재 할당된 블록들의 데이터들의 합, Hk는 현재 힙의 크기

    - 수식
$$
U_k = \frac{\max_{i \le k} P_i}{H_k}
$$

- 내부 단편화(Internal fragmentation) : Overhead나 padding에 의해서 발생되는데, 블록 내부에서 메모리를 효율적으로 관리하지 못하는 것

![외부 단편화](/assets/img/blog/computerscience/externalfragmentation.png)

- 외부 단편화(External fragmentation) : 가용 블록의 합들은 충분하지만, 가용 블록에 할당할 적당한 크기가 없게 관리가 된것

