---
layout: post
title:  "Lazy Loading"
description: >
 메모리를 무한대로 사용할 수 있을까?
date:   2025-06-10
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

먼저 여기서 처음 들어보는 개념이 있다면, [Paging](../../computersystem/paging){:.heading.flip-title}글에서 개념을 참고해 본다.

## Lazy Loading 이란?

> "필요해질 때까지 실제 데이터를 읽어들이지 않고, 최소한의 메타데이터만 준비해 두었다가 첫 접근 시점에 비로소 불러오는 기법"
> (메모리·스토리지·웹 모두 같은 핵심 아이디어를 공유합니다.)

## 가상메모리에서의 Lazy Loading = Demand Paging

| 시점 | 커널이 하는 일 | 이득 |
|:---:|:---:|:---:|
| exec 시 | - 각 세그먼트의 page table entry를 P=0(not-present)로만 채우고<br/>- "어느 파일 오프셋에서 몇 바이트를 읽으면 되는지" 보조 정보를 보관 | - 프로세스 시작 시간이 빨라짐<br/>- 실제로 쓰이지 않는 코드는 끝까지 안 올려도 됨 |
| 첫 접근(Page Fault) | 1. HW가 페이지 폴트를 발생시켜 커널에 트랩<br/>2. 커널 lazy loader가<br/>- 파일에서 해당 범위만큼 읽어 오거나(File backed)<br/>- 0으로 초기화(Anon page)<br/>- `install_page()`→P=1로 바꿈<br/>3. 사용자 코드 재시작 | - 실제 사용량만큼만 메모리 소비<br/>- I/O 병목이 줄어듬 |

PintOS Project3에서 구현하는 `uninit_new()`/`vm_try_handle_fault()`**흐름**이 정확히 이 구조다. 실행 파일을 로드할 때에는 `lazy_load_segment()`를 등록해 두고, 페이지 폴트가 오면 `file_read_at()`으로 필요한 바이트만 채운 뒤 PTE를 present로 갱신한다.

##