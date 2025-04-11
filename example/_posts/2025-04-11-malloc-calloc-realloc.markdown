---
layout: post
title:  "동적 메모리 할당(Dynamic memory allocation)"
date:   2025-04-11
hide_last_modified: true
---

* toc  
{:toc .large-only}

C언어에서의 동적 메모리 관리를 위해 사용하는 표준 라이브러리 함수 malloc, calloc, realloc, free에 대해서 알아보자. 이 함수들은 <stdio.h>에 정의 되어 있어. 

## 동적 메모리 할당을 사용할 때는 언제일까?

- 입력 크기를 미리 알 수 없을 때 (ex. 사용자 입력)

- 메모리를 효율적으로 쓰고 싶을 때

- 복잡한 자료구조를 다룰 때 (ex. 트리, 그래프, 연결 리스트)

먼저 동적 메모리 할당(dynamic memory allocation) 설명 전에 정적 메모리 할당(static memory allocation)에 대해 간략하게 소개할게

### C언어에서의 정적 할당과 동적 할당의 예시
~~~c
// 정적 할당
int arr[10];  // 크기 10으로 고정

// 동적 할당
int* ptr = (int*)malloc(10 * sizeof(int));  // 실행 중 필요한 만큼 할당
~~~

### 동적 메모리 할당 vs 정적 메모리 할당 차이점

| 구분 | 정적 메모리 할당 | 동적 메모리 할당 |
|:---:|:---:|:---:|
| 시점 | 컴파일 타임 | 런타임(실행 중) |
| 예시 | 배열처럼 크기가 고정된 변수 | 리스트처럼 크기가 유동적인 구조 |
| 메모리 해제 | 자동 해제 or 함수 종료 시 | 직접 해제 필요 (ex. free() in C) |

## 동적 메모리 할당(malloc, calloc, realloc, free)

### malloc(Memory Allocation)

지정한 바이트만큼의 메모리를 할당

~~~c
void* malloc(size_t size);
~~~
- 초기화: ❌ 초기화되지 않음. 메모리의 내용은 쓰레기값(garbage values)일 수 있음

- 반환값: 성공하면 void 포인터(void*)를 반환. 실패하면 NULL 반환

예제 : int 5개를 저장할 수 있는 메모리를 동적으로 할당함
~~~c
int* arr = (int*)malloc(5 * sizeof(int));
~~~

### calloc(Continguous Allocation)

num 개의 요소 각각 size 바이트 크기로 메모리를 할당.

~~~c
void* calloc(size_t num, size_t size);
~~~

- 초기화: ✅ 0으로 초기화됨

- 반환값: 성공하면 void* 반환. 실패하면 NULL

예제 : int 5개 크기의 메모리를 0으로 초기화한 채 할당.
~~~c
int* arr = (int*)calloc(5, sizeof(int));
~~~

### realloc(Reallocation)

이미 할당된 메모리 블록의 크기를 변경 (확장 또는 축소)

~~~c
void* realloc(void* ptr, size_t new_size);
~~~

- ptr이 NULL이면 malloc처럼 동작

- new_size가 0이면 free(ptr)처럼 동작

- 새 크기의 메모리를 새로 확보하고, 기존 데이터는 유지함. 새 영역으로 이동할 수도 있음

- 초기화: 새로 늘어난 부분은 초기화되지 않음

예제 : int 10개 크기로 확장함
~~~c
int* arr = (int*)malloc(5 * sizeof(int));
arr = (int*)realloc(arr, 10 * sizeof(int));
~~~

### free(ptr)

동적 메모리 할당 사용시 메모리 수동해제가 필수야! 아래처럼 사용이 끝났을 때 호출해야해

~~~c
int* ptr = (int*)malloc(sizeof(int) * 10);

// ... ptr을 사용해서 작업 수행 ...

free(ptr);  // 사용이 끝났으면 반드시 해제해야 함!
~~~

- 같은 포인터를 두 번 free()하면 안 돼! → "Double Free" 에러

- 보통 free() 한 뒤에는 NULL로 초기화하는 습관이 좋아
~~~c
int* data = (int*)malloc(100 * sizeof(int));
// ...
free(data);  // 꼭 해제!
data = NULL; // 안전을 위해 초기화
~~~

#### free() 사용하면 안 되는 경우

| 잘못된 예시 | 이유 |
|:---:|:---:|
| free() 없이 종료 | 메모리 누수 발생 |
| 스택에 할당한 변수에 free() | malloc 등으로 할당한 메모리만 해제 가능 |
| 같은 포인터를 두 번 free() | 프로그램이 비정상 종료될 수 있음 |
| 다른 포인터로 free() | 정확히 할당했던 포인터로만 해제해야 함 |

예제 : 스택 메모리에 free()
~~~c
int x = 10;
free(&x);  // ❌ 이렇게 하면 안 됨! (스택 메모리는 해제하지 마!)
~~~

## 주의할 점

- 반환값이 NULL인지 반드시 체크할 것! malloc이나 calloc은 실패할 수 있음

- 메모리 누수 방지를 위해 사용 후 free(ptr)를 꼭 호출해야 함

- malloc은 초기화를 안 해줘서 쓰기 전에 꼭 값 채워야 해

- realloc은 새로운 주소로 이동될 수 있으니, 반드시 반환값을 다시 변수에 할당해야 함

- free() 이후 그 포인터를 다시 쓰면 "댕글링 포인터(dangling pointer)" 문제가 생길 수 있어


## 마무리 

| 함수 | 역할 | 초기화 여부 | 사용 시기 |
|:---:|:---:|:---:|:---:|
| malloc | 지정한 크기만큼 할당 | ❌ 안 됨 | 빠르게 할당만 하고 싶을 때 |
| calloc | 여러 개 요소를 할당 + 초기화 | ✅ 0으로 초기화 | 배열 등 초기화가 필요한 경우 |
| realloc | 기존 메모리 크기 변경 | ❌ 안 됨 | 크기를 동적으로 바꿔야 할 때 |