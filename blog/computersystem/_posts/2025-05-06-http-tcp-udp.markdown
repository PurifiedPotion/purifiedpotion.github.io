---
layout: post
title:  "HTTP, TCP, UDP"
description: >
 HTTP, TCP, UDP는 네트워크 통신에서 자주 등장하는 개념들이고, 이들은 OSI 7계층 모델 또는 TCP/IP 4계층 모델의 서로 다른 계층에 위치해 있어. 아래에 각 개념과 계층별 차이점을 정리해줄게
date:   2025-05-06
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

## 개념 요약

| 프로토콜 | 설명 | 위치하는 계층 (TCP/IP 기준) |
|:---:|:---:|:---:|
| HTTP (HyperText Transfer Protocol) | 웹 브라우저와 서버 간의 데이터 전송 (ex. HTML, 이미지 요청 등) | 애플리케이션 계층 |
| TCP (Transmission Control Protocol) | 신뢰성 있는 연결 지향형 데이터 전송 (패킷 손실 검사 및 재전송) | 전송 계층 |
| UDP (User Datagram Protocol) | 빠르지만 신뢰성 없는 연결less형 데이터 전송 | 전송 계층 |

TCP/ IP 게층 모델에 관하여서는 