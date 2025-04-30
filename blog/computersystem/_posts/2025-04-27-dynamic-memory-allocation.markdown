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

    -
$$
U_k = \frac{\max_{i \le k} P_i}{H_k}
$$

- 내부 단편화(Internal fragmentation) : Overhead나 padding에 의해서 발생되는데, 블록 내부에서 메모리를 효율적으로 관리하지 못하는 것

![외부 단편화](/assets/img/blog/computerscience/externalfragmentation.png)

- 외부 단편화(External fragmentation) : 가용 블록의 합들은 충분하지만, 가용 블록에 할당할 적당한 크기가 없게 관리가 된것

## 명시적 리스트(Explicit list)에서의 refactoring

명시적 리스트를 구현하기 위해서 조금 복잡하게 구현했던 내용 개선에 대해서 얘기하려고 해

명시적 리스트를 구현하기 위해서는 가용 블록의 포인터 생성/변경 고려가 필요, 포인터 생성/변경 같은 경우 아래 사진과 같이 구현해야 함

![Case 4](/assets/img/blog/computerscience/case4.png)

Case 4 같은 경우에 포인터 8개의 생성/변경이 필요하기 때문에 아래와 같은 코드가 나옴

~~~c
static void *coalesce(void *bp)
{
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));
    size_t size = GET_SIZE(HDRP(bp));
    void *prev_free_root;
    void *prev_free_root_prev;
    void *next_free_blk_next;
    void *next_free_blk_prev;
    void *prev_free_blk_next;
    void *prev_free_blk_prev;

    if (prev_alloc && next_alloc)               // Case 1
    {
        if (free_root == NULL)
        {
            free_root = bp;
            SET_NEXT_FREE(bp, NULL);
            SET_PREV_FREE(bp, NULL);
        }
        else
        {
            prev_free_root = free_root;
            SET_PREV_FREE(prev_free_root, bp);
            SET_NEXT_FREE(bp, prev_free_root);
            SET_PREV_FREE(bp, NULL);
            free_root = bp;
        }
    }
    else if (prev_alloc && !next_alloc)         // Case 2
    {
        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0));

        if (free_root == NULL)
        {
            free_root = bp;
            SET_NEXT_FREE(bp, NULL);
            SET_PREV_FREE(bp, NULL);
        }
        else
        {
            prev_free_root = free_root;
            prev_free_root_prev = PREV_FREE(free_root);
            next_free_blk_next = NEXT_FREE(NEXT_BLKP(bp));
            next_free_blk_prev = PREV_FREE(NEXT_BLKP(bp));
            SET_NEXT_FREE(next_free_blk_prev, next_free_blk_next);
            SET_PREV_FREE(next_free_blk_next, next_free_blk_prev);
            SET_NEXT_FREE(bp, prev_free_root);
            SET_PREV_FREE(bp, NULL);
            SET_PREV_FREE(prev_free_root_prev, bp);
            free_root = bp;
        }

    }
    else if (!prev_alloc && next_alloc)         // Case 3
    {
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);

        if (free_root == NULL)
        {
            free_root = bp;
            SET_NEXT_FREE(bp, NULL);
            SET_PREV_FREE(bp, NULL);
        }
        else
        {
            prev_free_root = free_root;
            prev_free_root_prev = PREV_FREE(free_root);
            prev_free_blk_next = NEXT_FREE(PREV_BLKP(bp));
            prev_free_blk_prev = PREV_FREE(PREV_BLKP(bp));
            SET_NEXT_FREE(prev_free_blk_prev, prev_free_blk_next);
            SET_PREV_FREE(prev_free_blk_next, prev_free_blk_prev);
            SET_NEXT_FREE(bp, prev_free_root);
            SET_PREV_FREE(bp, NULL);
            SET_PREV_FREE(prev_free_root_prev, bp);
            free_root = bp;
        }

    }
    else                                        // Case 4
    {
        size += GET_SIZE(HDRP(PREV_BLKP(bp))) + GET_SIZE(FTRP(NEXT_BLKP(bp)));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        PUT(FTRP(NEXT_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);

        prev_free_root = free_root;
        prev_free_root_prev = PREV_FREE(free_root);
        prev_free_blk_next = NEXT_FREE(PREV_BLKP(bp));
        prev_free_blk_prev = PREV_FREE(PREV_BLKP(bp));
        next_free_blk_next = NEXT_FREE(NEXT_BLKP(bp));
        next_free_blk_prev = PREV_FREE(NEXT_BLKP(bp));
        SET_NEXT_FREE(prev_free_blk_prev, prev_free_blk_next);
        SET_PREV_FREE(prev_free_blk_next, prev_free_blk_prev);
        SET_NEXT_FREE(next_free_blk_prev, next_free_blk_next);
        SET_PREV_FREE(next_free_blk_next, next_free_blk_prev);
        SET_NEXT_FREE(bp, prev_free_root);
        SET_PREV_FREE(bp, NULL);
        SET_PREV_FREE(prev_free_root_prev, bp);
        free_root = bp;
    }
    free_root = bp;
    return bp;
}
~~~

동료의 의견도 있었고 Google AI Studio의 의견에 따라서 함수 구현함

![Google AI Studio](/assets/img/blog/computerscience/googleaistudio.png)

### Refactoring 함수

~~~c
// --- Helper 함수 (place, coalesce 에서 사용) ---
// Free list 맨 앞에 블록 추가 (LIFO)
static void add_to_free_list(void *bp) {
    if (free_root == NULL) { // 리스트가 비어있을 때
        SET_NEXT_FREE(bp, NULL);
        SET_PREV_FREE(bp, NULL);
        free_root = bp;
    } else { // 리스트에 블록이 있을 때
        SET_NEXT_FREE(bp, free_root);   // 현재 위치의 next를 기존 첫번째 free block
        SET_PREV_FREE(bp, NULL);        // 현재 위치의 prev를 NULL
        SET_PREV_FREE(free_root, bp);   // 기존 첫번째 free block의 PREV를 현재 블록으로
        free_root = bp; // 루트를 새 블록으로 업데이트
    }
}

// Free list에서 블록 제거
static void remove_from_free_list(void *bp) {
    void *prev_free = PREV_FREE(bp);
    void *next_free = NEXT_FREE(bp);

    if (prev_free == NULL) { // bp가 리스트의 첫 번째 블록일 때
        free_root = next_free; // 다음 블록을 루트로 설정
    } else { // bp가 중간 또는 마지막 블록일 때
        SET_NEXT_FREE(prev_free, next_free); // 이전 블록의 NEXT를 다음 블록으로
    }

    if (next_free != NULL) { // bp가 마지막 블록이 아닐 때
        SET_PREV_FREE(next_free, prev_free); // 다음 블록의 PREV를 이전 블록으로
    }
    // bp의 포인터는 초기화할 필요 없음 (어차피 할당되거나 병합될 것임)
}
~~~

### 개선된 Case 4

~~~c
    // Case 4: Merge with both previous and next blocks
    else { // (!prev_alloc && !next_alloc)
        remove_from_free_list(prev_blk); // Remove previous block
        remove_from_free_list(next_blk); // Remove next block
        size += GET_SIZE(HDRP(prev_blk)) + GET_SIZE(HDRP(next_blk)); // Use HDRP for next block size
        bp = prev_blk; // Move bp to the beginning of the merged block
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0)); // Footer position is FTRP(next_blk)
        add_to_free_list(bp); // Add the final merged block to the list
        return bp;
    }
~~~