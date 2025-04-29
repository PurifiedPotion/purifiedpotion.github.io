---
layout: post
title:  "System Call"
description: >
 응용프로그램(User level code)이 Hardware를 조작하고 싶을때, 무엇을 할까?
date:   2025-04-28
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

응용 프로그램(유저 레벨 코드)이 직접 하드웨어를 만지면 안 되므로, 운영체제(OS)가 대신 해준다. 프로세스가 커널에게 무언가를 “요청”하는 공식 통로가 바로 **System Call**이다.

- "파일을 열어 줘(open)", "데이터를 읽어 줘(read)", "새 프로세스를 만들어 줘(fork)" ... 같은 요구 사항을 전달한다

- 커널은 트랩(trap)을 통해 특권 모드(커널 모드)로 올라가 작업을 수행한 뒤, 결과(보통 레지스터 값 또는 errno)를 돌려주고 사용자 모드로 복귀한다

## 호출 흐름(리눅스 x86-64 예)

| 단계 | 사용자 공간 | 커널 공간 |
|:---:|:---|:---|
| 1 | 라이브러리 wrapper 함수 (read(int fd, void *buf, size_t n)) | - |
| 2 | 시스템 콜 번호와 인수들을 레지스터에 적재하고 syscall(또는 int 0x80) 실행 | 트랩 발생→권한 레벨 전환, 커널 스택 진입 |
| 3 | 시스템 콜 디스패처가 번호를 해석해 해당 핸들러(sys_read) 호출 | 핸들러가 실제 I/O 수행, 파일 테이블 등 갱신 |
| 4 | 반환값을 레지스터에 넣고 sysret | - |
| 5 | 라이브러리 wrapper가 음수면 errno 설정 후 반환 | - |

여기서 **컨텍스트 스위치(레지스터·프로그램 카운터·스택 교체)**는 수십 ~ 수백 나노초가 소요되므로, 시스템 콜은 함수 호출보다 느리다

## 대표 범주

| 범주 | 대표 시스템 콜 | 설명·용도 |
|:---:|:---|:---|
| 프로세스 제어 | fork, execve, waitpid, exit | 프로세스 생성·변경·종료 |
| 파일 I/O | open/close, read/write, lseek, stat | 모든 디스크·파이프·소켓 I/O 통일 인터페이스("Everything is a file") |
| 장치 제어 | ioctl, mmap, munmap | 특수 장치 제어, 파일 매핑 등 |
| 메모리 관리  | brk, mmap | 힙 확장(malloc 내부) |
| 5 | 라이브러리 wrapper가 음수면 errno 설정 후 반환 | - |

## 언제 볼 수 있을까?

- Windows - MEM_COMMIT 없이 VirtualAlloc으로 예약만 한 뒤 접근할 때, 또는 일반 HeapAlloc/new 내부적으로

- Linux/Unix - 익명 매핑 mmap(..., MAP_ANONYMOUS)·스택·BSS·brk/sbrk 등

- 하이퍼바이저/가상머신 - 게스트가 처음 쓰는 시점까지 진짜 호스트 RAM을 배정하지 않는 ballooning 기법과 결합되기도 함

## 요약

Demand-zero memory = "필요할 때 0으로 초기화된 물리 페이지를 뒤늦게 붙이는 가상 메모리 기술"

→ 프로그램 입장에서는 '이미 0으로 초기화돼 있는 새 메모리'를 즉시 얻은 것처럼 보이지만, 실제 RAM은 첫 사용 시점까지 쓰이지 않아 메모리를 아끼고 보안을 높여 줌

### 나만의 요약

1. 물리 메모리 같은 경우 첫 사용 전까지는 메모리 할당이 되지 않음

2. 하지만 가상 메모리에서는 공간이 확보된 상태

3. 실제 사용시에는 page fault 되면서 물리 메모리 할당이 이루어짐, PTE의 Not-present → Present로 갱신