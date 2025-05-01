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

    - 수식(높은것이 좋다) :
$$
U_k = \frac{\max_{i \le k} P_i}{H_k}
$$

- 내부 단편화(Internal fragmentation) : Overhead나 padding에 의해서 발생되는데, 블록 내부에서 메모리를 효율적으로 관리하지 못하는 것

![외부 단편화](/assets/img/blog/computerscience/externalfragmentation.png)

- 외부 단편화(External fragmentation) : 가용 블록의 합들은 충분하지만, 가용 블록에 할당할 적당한 크기가 없게 관리가 된것

## Malloc 할당 기초

### Header 탄생

우리가 포인터를 사용해서 블럭과 블럭을 건너뛰기 위해서는 어떻게 해야 할까? payload가 어디서부터 시작하지? 어디까지 읽어야 하지?

라는 의문에서 탄생한 개념이 '**header라는 overhead를 만들어서 거기에 block의 사이즈를 넣으면 어떨까?**' 이다

1. header를 사용해서 각 블럭의 처음으로 순차적으로 탐색이 가능해 졌다

    - 그러면 header를 어디에 할당을 할까? 
    
        - 규칙이야 만들면 그만이지만, 책에서는 특정 주소를 받으면 payload 시작 주소를 받기로 했다

        - payload 주소를 먼저 받게 되면, 그 앞에 overhead를 추가하기로 했다

    - 그러면 태초의 주소를 받게 되면, 앞이 없기 때문에 overhead를 못 넣지 않은가?

        - 그렇기 때문에 우리는 첫 주소에 항상 overhead용 padding을 해줘야 한다

![헤더](/assets/img/blog/computerscience/header.png)

여기서 추가적으로 우리는 가용 블록 or 비가용 블록인지 확인이 필요하다. 
그 이유는 할당 블럭에 다른 용도로 또 할당을 하면 안되기 때문.

32bit 기준으로 8byte기준, 64bit system으로는 16byte 정렬이 됨(여기서 정렬이라 하는것은 정렬만큼 최소크기로 블럭을 관리하겠다는 얘기이다)
그렇게 되면 3bit 또는 4bit는 항상 0으로 남게 된다.

2. 남게 되는 bit들을 이용해 다른 정보들을 저장하자

    - overhead를 읽을 때 첫 3bit or 4bit를 제외하고 읽고 3bit or 4bit를 따로 읽는것이다. 그렇게 되면 block size와 다른 정보들이 나오게 된다.

    - 책에서는 32bit system을 사용해서 3bit가 남고 가장 첫번째 bit에 가용/비가용(1/0)으로 구분하기로 했다

![헤더](/assets/img/blog/computerscience/headerwithalloc.png)

### Footer 탄생

할당을 하면서 Free도 하게 될텐데, 그렇게 되면 Free블록이 연속될 수 있다.
아래와 같은 사진을 보면 4사이즈 가용 블럭과 2사이즈 가용 블럭이 연속된다
이 상황에서 5사이즈 블럭을 할당하고자 할 때, 할당이 되지 않는 상황(외부 단편화)이 발생한다.

![연속된 가용 블럭](/assets/img/blog/computerscience/sequentialfreeblock.png)

할당이 되지 않는 이유는 코드상 '할당하려는 크기보다 같거나 큰 블럭을 찾게 되어 있어서'이다. 
여기서 다른 방법을 생각할 수도 있지만, 코드를 유지하면서 방법을 생각해보자.

1. 연속된 가용 블럭이 나오지 않게 하자

    - 만약 연속된 가용 블럭이 없으면 아쉽지도 않을 것이다

    - 그렇다면 어떻게 가용 블럭이 연속되지 않게 할까?

2. 할당된 블럭을 free 할때 앞과 뒤에 가용 블럭이 있다면 병합을 하자

    - 뒤에 블록 같은 경우 현재 header에서 사이즈를 더해 확인이 가능

    - 앞 블록 같은 경우 사이즈를 모름 → **footer 도입**해서 블럭뒤에 블럭의 크기와 가용/비가용 정보를 저장

    - 병합같은 경우 free하고 병합(coalesce)진행

extra. Footer같은 경우 overhead이기 때문에 없는게 좋다. 그렇다면 모든 블럭에 footer가 필요할까?

    - 아까 정렬에 따라 3 ~ 4bit가 남는다고 했다. 여기에는 현재 블록의 가용/비가용 정보가 들어가는데, 앞블럭의 가용/비가용 정보를 넣자

    - 앞블럭의 가용/비가용 정보가 있으면, 가용 블럭에만 footer가 필요하고 할당된 블럭에는 footer가 필요 없게 된다

### C 언어로 구현된 Malloc

내가 직접 구현하지 않고 CS:APP 교제를 통해 구할 수 있게 되었다. 묵시적 list에 first-fit 할당이다

~~~c
/*
 * mm-naive.c - The fastest, least memory-efficient malloc package.
 *
 * In this naive approach, a block is allocated by simply incrementing
 * the brk pointer.  A block is pure payload. There are no headers or
 * footers.  Blocks are never coalesced or reused. Realloc is
 * implemented directly using mm_malloc and mm_free.
 *
 * NOTE TO STUDENTS: Replace this header comment with your own header
 * comment that gives a high level description of your solution.
 */
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <string.h>

#include "mm.h"
#include "memlib.h"

/*********************************************************
 * NOTE TO STUDENTS: Before you do anything else, please
 * provide your team information in the following struct.
 ********************************************************/
team_t team = {
    /* Team name */
    "ateam",
    /* First member's full name */
    "Harry Bovik",
    /* First member's email address */
    "bovik@cs.cmu.edu",
    /* Second member's full name (leave blank if none) */
    "",
    /* Second member's email address (leave blank if none) */
    ""};

/* single word (4) or double word (8) alignment */
#define ALIGNMENT 8
#define WSIZE 4 /* header/footer size */
#define DSIZE 8 /* double word size */
#define CHUNKSIZE (1 << 12) /* Extend heap by this amount (bytes) */

#define MAX(x, y) ((x) > (y) ? (x) : (y))

/* Pack a size and allocated bit into a word */
#define PACK(size, alloc) ((size) | (alloc))

/* Read and write a word at address p */
#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p) = (val))

/* Read the size and allocated fields from address p */
#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)

/* Given block ptr bp, compute address of its header and footer */
#define HDRP(bp) ((char *)(bp) - WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)

/* Given block ptr bp, compute address of next and previous blocks */
#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp)-WSIZE)))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp)-DSIZE)))

/* rounds up to the nearest multiple of ALIGNMENT */
#define ALIGN(size) (((size) + (ALIGNMENT - 1)) & ~0x7)

#define SIZE_T_SIZE (ALIGN(sizeof(size_t)))

static char *heap_listp;
int mm_init(void);
static void *extend_heap(size_t words);
static void *find_fit(size_t asize);
static void place(void *bp, size_t asize);
static void *coalesce(void *bp);


/*
 * mm_init - initialize the malloc package.
 */
int mm_init(void)
{
    /* Create the initial empty heap */
    if ((heap_listp = mem_sbrk(4*WSIZE)) == (void *)-1)
        return -1;
    PUT(heap_listp, 0);
    PUT(heap_listp + (1*WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (2*WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (3*WSIZE), PACK(0, 1));
    heap_listp += (2*WSIZE);
    // next_ptr = heap_listp; /* Reset next_ptr to NULL */

    /* Extend the empty heap with a free block of CHUNKSIZE bytes */
    if (extend_heap(CHUNKSIZE/WSIZE) == NULL)
        return -1;
    return 0;
}

static void *extend_heap(size_t words)
{
    char *bp;
    size_t size;

    /* Allocate an even number of words to maintain alignment */
    size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE;
    if ((long)(bp = mem_sbrk(size)) == -1)
        return NULL;

    /* Initialize free block header/footer and the epilogue header */
    PUT(HDRP(bp), PACK(size, 0)); /* Free block header */
    PUT(FTRP(bp), PACK(size, 0)); /* Free block footer */
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1)); /* New epilogue header */


    /* Coalesce if the previous block was free */
    return coalesce(bp);
}

/*
 * mm_malloc - Allocate a block by incrementing the brk pointer.
 *     Always allocate a block whose size is a multiple of the alignment.
 */
void *mm_malloc(size_t size)
{
    size_t asize;
    size_t extendsize;
    char * bp;

    /* Ignore spurious requests */
    if (size == 0)
        return NULL;
    
    /* Adjust block size to include overhead and alignment reqs. */
    if (size <= DSIZE)
        asize = 2 * DSIZE;
    else
        asize = DSIZE * ((size + DSIZE + (DSIZE - 1)) / DSIZE);

    /* Search the free list for a fit */
    if ((bp = find_fit(asize)) != NULL)
    {
        place(bp, asize);
        return bp;
    }

    /* No fit found. Get more memory and place the block */
    extendsize = MAX(asize, CHUNKSIZE);
    if ((bp = extend_heap(extendsize/WSIZE)) == NULL)
        return NULL;
    place(bp, asize);
    return bp;
}

static void *find_fit(size_t asize)
{
    /* First-fit search */

    void *bp;

    for (bp = heap_listp; GET_SIZE(HDRP(bp)) > 0; bp = NEXT_BLKP(bp))
    {
        if (!GET_ALLOC(HDRP(bp)) && (asize <= GET_SIZE(HDRP(bp))))
        {
            return bp;
        }
    }
    return NULL;
}

static void place(void *bp, size_t asize)
{
    size_t csize = GET_SIZE(HDRP(bp));

    if ((csize - asize) >= (2*DSIZE))
    {
        PUT(HDRP(bp), PACK(asize, 1));
        PUT(FTRP(bp), PACK(asize, 1));
        bp = NEXT_BLKP(bp);
        PUT(HDRP(bp), PACK(csize-asize, 0));
        PUT(FTRP(bp), PACK(csize-asize, 0));
    }
    else
    {
        PUT(HDRP(bp), PACK(csize, 1));
        PUT(FTRP(bp), PACK(csize, 1));
    }
}

/*
 * mm_free - Freeing a block does nothing.
 */
void mm_free(void *ptr)
{
    size_t size = GET_SIZE(HDRP(ptr));

    PUT(HDRP(ptr), PACK(size, 0)); /* 헤더에 동일한 사이즈와 할당되지 않은 정보로 update */
    PUT(FTRP(ptr), PACK(size, 0)); /* 푸터에 동일한 사이즈와 할당되지 않은 정보로 update */
    coalesce(ptr); /* free 끼리 병합 */
}

static void *coalesce(void *bp)
{
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));
    size_t size = GET_SIZE(HDRP(bp));

    if (prev_alloc && next_alloc)
        return bp;

    else if (prev_alloc && !next_alloc)
    {
        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0));

    }
    else if (!prev_alloc && next_alloc)
    {
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);
    }
    else
    {
        size += GET_SIZE(HDRP(PREV_BLKP(bp))) + GET_SIZE(FTRP(NEXT_BLKP(bp)));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        PUT(FTRP(NEXT_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);
    }
    return bp;
}

/*
 * mm_realloc - Implemented simply in terms of mm_malloc and mm_free
 */
void *mm_realloc(void *ptr, size_t size)
{
    void *oldptr = ptr;
    void *newptr;
    size_t copySize;

    newptr = mm_malloc(size);
    if (newptr == NULL)
        return NULL;
    copySize = *(size_t *)((char *)oldptr - SIZE_T_SIZE);
    if (size < copySize)
        copySize = size;
    memcpy(newptr, oldptr, copySize);
    mm_free(oldptr);
    return newptr;
}
~~~

이 코드로 돌려보니 아래와 같은 결과가 나왔다
![묵시적 first-fit 결과](/assets/img/blog/computerscience/implicitfirst.png)

그 다음으로는 묵시적에서 제일 빠르다는 next-fit을 구현해 보았다

바뀐 부분은 아래에 정리했다
~~~c
static void *next_ptr = NULL; // 전역변수로 선언

static void *extend_heap(size_t words)
{
    char *bp;
    size_t size;

    /* Allocate an even number of words to maintain alignment */
    size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE;
    if ((long)(bp = mem_sbrk(size)) == -1)
        return NULL;

    /* Initialize free block header/footer and the epilogue header */
    PUT(HDRP(bp), PACK(size, 0)); /* Free block header */
    PUT(FTRP(bp), PACK(size, 0)); /* Free block footer */
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1)); /* New epilogue header */
    next_ptr = bp; /* Set next_ptr to the new block */

    /* Coalesce if the previous block was free */
    return coalesce(bp);
}

static void *find_fit(size_t asize)
{
    void *bp;

    for (bp = next_ptr; GET_SIZE(HDRP(bp)) > 0; bp = NEXT_BLKP(bp))
    {
        if (!GET_ALLOC(HDRP(bp)) && (asize <= GET_SIZE(HDRP(bp))))
        {
            next_ptr = bp;
            return bp;
        }
    }
    return NULL;
}

static void *coalesce(void *bp)
{
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));
    size_t size = GET_SIZE(HDRP(bp));

    if (prev_alloc && next_alloc)
        return bp;

    else if (prev_alloc && !next_alloc)
    {
        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0));

    }
    else if (!prev_alloc && next_alloc)
    {
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);
    }
    else
    {
        size += GET_SIZE(HDRP(PREV_BLKP(bp))) + GET_SIZE(FTRP(NEXT_BLKP(bp)));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        PUT(FTRP(NEXT_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);
    }
    next_ptr = bp;
    return bp;
}
~~~

바뀐 코드중에 보면 사실 coalesce에 next_ptr을 free한 블럭에 넣는것은 next-fit 개념에 어긋난다. 하지만, test case가 조금 많은 편이어서 이렇게 진행을 했다.

결과는 아래와 같이 21점이 올라간것을 확인할 수 있다

![묵시적 next-fit 결과](/assets/img/blog/computerscience/implicitnext.png)

마지막으로 명시적을 구현해 보았다 코드는 아래와 같다

~~~c
/*
 * mm-naive.c - The fastest, least memory-efficient malloc package.
 *
 * In this naive approach, a block is allocated by simply incrementing
 * the brk pointer.  A block is pure payload. There are no headers or
 * footers.  Blocks are never coalesced or reused. Realloc is
 * implemented directly using mm_malloc and mm_free.
 *
 * NOTE TO STUDENTS: Replace this header comment with your own header
 * comment that gives a high level description of your solution.
 */
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <string.h>

#include "mm.h"
#include "memlib.h"

/*********************************************************
 * NOTE TO STUDENTS: Before you do anything else, please
 * provide your team information in the following struct.
 ********************************************************/
team_t team = {
    /* Team name */
    "ateam",
    /* First member's full name */
    "Harry Bovik",
    /* First member's email address */
    "bovik@cs.cmu.edu",
    /* Second member's full name (leave blank if none) */
    "",
    /* Second member's email address (leave blank if none) */
    ""};

/* single word (4) or double word (8) alignment */
#define ALIGNMENT 16
#define WSIZE 4 /* header/footer size */
#define PTR_SIZE sizeof(void *) /* pointer size */
#define MIN_FREE_BLOCK_SIZE (WSIZE + PTR_SIZE + PTR_SIZE + WSIZE)
#define CHUNKSIZE (1 << 12) /* Extend heap by this amount (bytes) */

#define MAX(x, y) ((x) > (y) ? (x) : (y))

/* Pack a size and allocated bit into a word */
#define PACK(size, alloc) ((size) | (alloc))

/* Read and write a word at address p */
#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p) = (val))

/* Read the size and allocated fields from address p */
#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)

/* Given block ptr bp, compute address of its header and footer */
#define HDRP(bp) ((char *)(bp) - WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - (WSIZE*2))

/* Given block ptr bp, compute address of next and previous blocks */
#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp)-WSIZE)))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp)-(WSIZE*2))))

#define NEXT_FREE(bp) (*(void **)(bp)) // bp를 형변환 한거야 이중연결리스트로 그래서 bp가 가르키는 메모리공간에 다른블록을 가르키는 포인터가 생겨버리는거지 그걸 역참조해서 NEXT_FREE(bp)는 bp안에 다른블록을 가르키는 포인터의 값을 나타 내는 아이가 된거야.
#define PREV_FREE(bp) (*(void **)((char *)(bp) + PTR_SIZE)) // 그럼이것도 NEXT_FREE와 비슷하지만 한가지 다른게 bp의 시작주소 payload가 가르키는 메모리공간이 아니라 payload에다 4를 더한 주소에 prev_free를 저장하거나 읽는거야

#define SET_NEXT_FREE(bp, ptr) (NEXT_FREE(bp) = (ptr)) // 그럼 이건 위에는 포인터주소에 역참조한 상황이니까 거기에다가 ptr이라는 주소값을 넣은거지 즉 next인 주소를 넣은거지
#define SET_PREV_FREE(bp, ptr) (PREV_FREE(bp) = (ptr)) // 현재 가르키는 블록의 payload+wsize 위치에 이전 free블록의 주소를 저장하는거지

// #define NEXT_FRPTR(bp) (bp)
#define PREV_FRPTR(bp) ((char *)(bp) + GET_SIZE(((char *)(bp)+(WSIZE*2))))

/* rounds up to the nearest multiple of ALIGNMENT */
#define ALIGN(size) (((size) + (ALIGNMENT - 1)) & ~0xf)

#define SIZE_T_SIZE (ALIGN(sizeof(size_t)))

static void *free_root = NULL; /* Pointer to the first free block */
static char *heap_listp;

// Helper functions for free list manipulation
static void add_to_free_list(void *bp);
static void remove_from_free_list(void *bp);

int mm_init(void);
static void *extend_heap(size_t words);
static void *find_fit(size_t asize);
static void place(void *bp, size_t asize);
static void *coalesce(void *bp);


/*
 * mm_init - initialize the malloc package.
 */
int mm_init(void)
{
    /* Create the initial empty heap */
    if ((heap_listp = mem_sbrk(4*WSIZE)) == (void *)-1)
        return -1;
    PUT(heap_listp, 0);
    PUT(heap_listp + (1*WSIZE), PACK((WSIZE*2), 1));
    PUT(heap_listp + (2*WSIZE), PACK((WSIZE*2), 1));
    PUT(heap_listp + (3*WSIZE), PACK(0, 1));
    heap_listp += (2*WSIZE);
    free_root = NULL;
    // next_ptr = heap_listp; /* Reset next_ptr to NULL */

    /* Extend the empty heap with a free block of CHUNKSIZE bytes */
    if (extend_heap(CHUNKSIZE/WSIZE) == NULL)
        return -1;
    return 0;
}

static void *extend_heap(size_t words)
{
    char *bp;
    size_t size;

    /* Allocate an even number of words to maintain alignment */
    // size 계산은 바이트 단위로, ALIGNMENT(16)의 배수로 맞춤
    size = (words * WSIZE); // words는 WSIZE 단위이므로 총 바이트 계산
    size = ALIGN(size);     // 최종 크기를 16의 배수로 올림 (이게 더 안전)
    // size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE; // 원래 로직도 가능은 함

    if ((long)(bp = mem_sbrk(size)) == -1)
        return NULL;

    /* Initialize free block header/footer and the epilogue header */
    PUT(HDRP(bp), PACK(size, 0));           /* Free block header */
    PUT(FTRP(bp), PACK(size, 0));           /* Free block footer */
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1)); /* New epilogue header */

    /* Coalesce if the previous block was free and add block to free list */
    // free_root 설정 및 포인터 초기화는 coalesce에서 처리
    return coalesce(bp);
}

/*
 * mm_malloc - Allocate a block by incrementing the brk pointer.
 *     Always allocate a block whose size is a multiple of the alignment.
 */
void *mm_malloc(size_t size)
{
    size_t asize;
    size_t extendsize;
    char * bp;

    /* Ignore spurious requests */
    if (size == 0)
        return NULL;

    size_t needed = size + (WSIZE * 2);
    
    /* Adjust block size to include overhead and alignment reqs. */
    if (needed <= MIN_FREE_BLOCK_SIZE)
        asize = MIN_FREE_BLOCK_SIZE;
    else
        asize = ALIGN(needed);

    /* Search the free list for a fit */
    if ((bp = find_fit(asize)) != NULL)
    {
        place(bp, asize);
        return bp;
    }

    /* No fit found. Get more memory and place the block */
    extendsize = MAX(asize, CHUNKSIZE);
    if ((bp = extend_heap(extendsize/WSIZE)) == NULL)
        return NULL;
    place(bp, asize);
    return bp;
}



static void *find_fit(size_t asize)
{
    /* First-fit search */
    void *bp;

    // free_root (리스트 헤드) 부터 시작해서 NULL 만날 때까지 순회
    for (bp = free_root; bp != NULL; bp = NEXT_FREE(bp))
    {
        // GET_ALLOC은 필요 없을 수 있음 (free list에는 free 블록만 있어야 함)
        // 하지만 안전을 위해 남겨둘 수 있음
        // if (!GET_ALLOC(HDRP(bp)) && (asize <= GET_SIZE(HDRP(bp))))
        if (asize <= GET_SIZE(HDRP(bp))) // 크기만 비교
        {
            return bp; // 적합한 블록 찾음
        }
    }
    return NULL; /* No fit */
}

// place 함수 수정
static void place(void *bp, size_t asize)
{
    size_t csize = GET_SIZE(HDRP(bp));
    // void *cur_free_next = NEXT_FREE(bp); // Helper 사용 시 필요 없어짐
    // void *cur_free_prev = PREV_FREE(bp); // Helper 사용 시 필요 없어짐

    // 먼저 free list에서 현재 블록(bp)을 제거
    remove_from_free_list(bp);

    if ((csize - asize) >= (MIN_FREE_BLOCK_SIZE)) // 분할 가능한 경우
    {
        // 할당될 부분 설정
        PUT(HDRP(bp), PACK(asize, 1));
        PUT(FTRP(bp), PACK(asize, 1));

        // 남은 free 부분 설정
        void *next_bp = NEXT_BLKP(bp); // 남은 블록 포인터
        PUT(HDRP(next_bp), PACK(csize - asize, 0));
        PUT(FTRP(next_bp), PACK(csize - asize, 0));

        // 남은 free 블록을 free list에 다시 추가
        add_to_free_list(next_bp);
    }
    else // 분할 불가능한 경우 (전체 블록 사용)
    {
        PUT(HDRP(bp), PACK(csize, 1));
        PUT(FTRP(bp), PACK(csize, 1));
        // free list에서 이미 제거했으므로 추가 작업 필요 없음
    }
}

/*
 * mm_free - Freeing a block does nothing.
 */
void mm_free(void *ptr)
{
    size_t size = GET_SIZE(HDRP(ptr));

    PUT(HDRP(ptr), PACK(size, 0)); /* 헤더에 동일한 사이즈와 할당되지 않은 정보로 update */
    PUT(FTRP(ptr), PACK(size, 0)); /* 푸터에 동일한 사이즈와 할당되지 않은 정보로 update */
    coalesce(ptr); /* free 끼리 병합 */
}

// coalesce 함수 수정 (Helper 함수 사용)
static void *coalesce(void *bp)
{
    // 프롤로그 블록 바로 뒤 또는 에필로그 블록 바로 앞인지 확인하여 FTRP/HDRP 접근 보호
    void *prev_blk = PREV_BLKP(bp);
    void *next_blk = NEXT_BLKP(bp);

    // GET_ALLOC은 헤더/푸터에서 읽으므로 주소 유효성 먼저 체크
    // Prologue block's footer is always allocated.
    // Epilogue block's header is always allocated.
    size_t prev_alloc = GET_ALLOC(FTRP(prev_blk)); // Can read footer unless bp is first block
    size_t next_alloc = GET_ALLOC(HDRP(next_blk)); // Can read header unless bp is last block
    size_t size = GET_SIZE(HDRP(bp));

    // Case 1: No merge needed
    if (prev_alloc && next_alloc) {
        add_to_free_list(bp); // Add the newly freed block to the list
        return bp;
    }

    // Case 2: Merge with next block
    else if (prev_alloc && !next_alloc) {
        remove_from_free_list(next_blk); // Remove next block from list
        size += GET_SIZE(HDRP(next_blk));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0)); // Footer position is now FTRP(next_blk), but FTRP uses HDRP size, so this works
        add_to_free_list(bp); // Add the merged block to the list
        return bp;
    }

    // Case 3: Merge with previous block
    else if (!prev_alloc && next_alloc) {
        remove_from_free_list(prev_blk); // Remove previous block from list
        size += GET_SIZE(HDRP(prev_blk));
        bp = prev_blk; // Move bp to the beginning of the merged block
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0)); // Footer position is original FTRP(bp)
        add_to_free_list(bp); // Add the merged block to the list
        return bp;
    }

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
    // 마지막 free_root = bp; 제거
}

/*
 * mm_realloc - Implemented simply in terms of mm_malloc and mm_free
 */
void *mm_realloc(void *ptr, size_t size)
{
    void *oldptr = ptr;
    void *newptr;
    size_t copySize;

    newptr = mm_malloc(size);
    if (newptr == NULL)
        return NULL;
    copySize = GET_SIZE(HDRP(oldptr)) - (WSIZE*2);
    if (size < copySize)
        copySize = size;
    memcpy(newptr, oldptr, copySize);
    mm_free(oldptr);
    return newptr;
}

// --- Helper 함수 (place, coalesce 에서 사용) ---
// Free list 맨 앞에 블록 추가 (LIFO)
static void add_to_free_list(void *bp) {
    if (free_root == NULL) { // 리스트가 비어있을 때
        SET_NEXT_FREE(bp, NULL);
        SET_PREV_FREE(bp, NULL);
        free_root = bp;
    } else { // 리스트에 블록이 있을 때
        SET_NEXT_FREE(bp, free_root);
        SET_PREV_FREE(bp, NULL);
        SET_PREV_FREE(free_root, bp); // 기존 루트의 PREV를 새 블록으로
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

결과는 아래와 같다. 1점이 오른걸 확인할 수 있다. 명시적 list로는 best-fit이 메모리 효율이 제일 좋다. best-fit까지 구현을 할까 생각했는데, 주 마지막날이어서 다음날 발표를 준비해야 했다. 그래서 포기했다..

![명시적 first-fit 결과](/assets/img/blog/computerscience/implicitnext.png)

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

- add_to_free_list : 아래 사진의 ①, ②, ③, ④ 포인터의 변경/생성에 관여

- remove_from_free_list : 아래 사진의 ⑤, ⑥, ⑦, ⑧ 포인터의 변경/생성에 관여

![Case4 함수별 포인터](/assets/img/blog/computerscience/case4pointers.png)

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