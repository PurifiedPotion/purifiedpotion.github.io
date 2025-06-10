---
layout: post
title:  "PintOS 4~5주차 lazy loading & swap구현"
description: >
 이 글에서는 PintOS 4~5주차 lazy loading & swap 구현에 관련하여서 다루겠다.
date:   2025-06-08
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

이전에 만들었던 thread와 user program에 더해서 이번주 부터는 가상 메모리에 관한 함수를 구현해야 한다.

구현하기에 앞서서 사전에 알아두어야 하는 개념들에 대해서 먼저 설명하겠다.

## 구현에 필요한 개념들

[PML4](../../computersystem/pml4){:.heading.flip-title}, [Paging](../../computersystem/paging){:.heading.flip-title}, [Lazy Loading](../../computersystem/lazy-loading){:.heading.flip-title}, [Anonymous & File-backed Page](../../computersystem/anon-file){:.heading.flip-title}, 
