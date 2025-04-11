---
layout: post
title:  "malloc, calloc, realloc"
date:   2025-04-11
hide_last_modified: true
---

* toc  
{:toc .large-only}

C언어에서의 동적 메모리 관리를 위해 사용하는 표준 라이브러리 함수 malloc, calloc, realloc에 대해서 알아보자. 이 함수들은 <stdio.h>에 정의 되어 있어


## malloc(Memory Allocation)

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

## calloc(Continguous Allocation)

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

## realloc(Reallocation)

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

## 주의할 점

- 반환값이 NULL인지 반드시 체크할 것!

- 메모리 누수 방지를 위해 사용 후 free(ptr)를 꼭 호출해야 함

- realloc은 새로운 주소로 이동될 수 있으니, 반드시 반환값을 다시 변수에 할당해야 함


## 마무리 

| 함수 | 역할 | 초기화 여부 | 사용 시기 |
|:---:|:---:|:---:|:---:|
| malloc | 지정한 크기만큼 할당 | ❌ 안 됨 | 빠르게 할당만 하고 싶을 때 |
| calloc | 여러 개 요소를 할당 + 초기화 | ✅ 0으로 초기화 | 배열 등 초기화가 필요한 경우 |
| realloc | 기존 메모리 크기 변경 | ❌ 안 됨 | 크기를 동적으로 바꿔야 할 때 |