---
layout: post
title:  "CS:APP 11장 네트워크 프로그래밍 & Proxy 서버 C언어로 구현"
description: >
 우리가 웹을 검색하고, 이메일 메시지를 보내고, 온라인 게임을 하는 등의 모든 경우 우리는 네트워크 응용을 사용한다
date:   2025-05-08
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

흥미로운 것은 모든 네트워크 응용들은 동일한 기본적인 프로그래밍 모델에 기초하고 있으며, 이들은 비슷한 전체 논리 구조를 가지며, 동일한 프로그래밍 인터페이스를 사용한다는 점이다. 개념들을 설명하고 난 후 개념들을 모두 연결해서 아주 작지만 실제로 동작하는 웹 서버를 개발할 것이다.

## 클라이언트 - 서버 프로그래밍 모델

** 모든 네트워크 응용 프로그램은 클라이언트-서버 모델에 기초하고 있다.** 아래와 같은 사진과 같이 한 개의 서버 프로세스와 한 개 이상의 클라이언트 프로세스로 구성된다.

![클라이언트 서버 모델](/assets/img/blog/computerscience/clientservertransaction.png)

1. 클라이언트가 한 개의 요청(request)을 서버에 보내는 것으로 트랜잭션을 개시한다. 예를 들어, 웹 브라우저가 파일을 필요로 할때, 웹 서버로 요청을 보낸다.

2. 서버는 요청을 받고, 해석하고, 자신의 자원들을 적절한 방법으로 조작한다. 예를 들어, 웹 서버가 브라우저로부터 요청을 받을 때, 디스크 파일을 읽는다.

3. 서버는 응답(response)을 클라이언트로 보내고, 그 후에 다음 요청을 기다린다. 예를 들어, 웹 서버는 이 파일을 다시 클라이언트로 돌려보낸다.

4. 클라이언트는 응답을 받고 이걸을 처리한다. 예를 들어, 웹 브라우저가 서버로부터 페이지를 한 개 받은 후, 이것을 스크린에 디스플레이한다.

## 네트워크

물리적으로 네트워크는 기하학적 위치로 구성된 **계층구조 시스템**이다 하위수준은 **LAN(Local Area Network)**으로 빌딩이나 캠퍼스에 설치된다. 가장 대중적인 LAN기술은 현재까지는 이더넷(Ehternet)이며, 시간에 따라서 엄청 발전되어 왔다. 이더넷(Ethernet)에 관련한 설명은 아래에 있다.

### NIC (Network Interface Card)

호스트에게 네트워크는 단지 또 다른 I/O 디바이스이다. 아래 사진과 같이 네트워크에서 수신한 데이터는 I/O와 메모리 버스를 거쳐서 어댑터에서 메모리로, 대개 DMA 전송으로 복사된다. 비슷하게 데이터는 또한 메모리에서 네트워크로 복사될 수 있다.

![호스트의 네트워크 구성](/assets/img/blog/computerscience/hosthardwareorganization.png)

### 이더넷 (Ethernet)

이더넷 세그먼트는 아래 사진과 같이 몇 개의 전선들과 허브라고 부르는 작은 상자로 구성된다. 한쪽 끝은 호스트의 어댑터에 연결되고, 다른 끝은 허브의 포트에 연결된다. 허브는 각 포트에서 수신한 모든 비트를 종속적으로 다른 모든 포트로 복사한다.

![이더넷 세그먼트](/assets/img/blog/computerscience/ethernetsegment.png)

- **이더넷 어댑터**는 어댑터의 비휘발성 메모리에 저장된 전체적으로 **고유한 48비트 주소**를 가진다.

- 호스트는 **프레임**이라고 부르는 비트들을 세그먼트의 다른 호스트에 보낼 수 있다.

- 각 **프레임**은 프레임의 **소스**와 **목적지**, 프레임의 **길이**를 식별할 수 있는 고정된 **헤더 비트**를 가지고 있으며, 그 뒤에 데이터 비트가 이어진다.

- 모든 호스트 어댑터는 이 프레임을 볼 수 있지만, 목적지 호스트만이 실제로 이것을 읽어들인다.

#### 더 큰 LAN - 브릿지의 등장

아래 사진에 나타난 것처럼 전선들과 브릿지라고 하는 작은 상자들을 사용해서 **다수의 이더넷 세그먼트가 연결되어 브릿지형 이더넷이라고 하는 더 큰 LAN을 구성할 수 있다.

![브릿지로 연결된 이더넷 세그먼트](/assets/img/blog/computerscience/bridgedethernetsegments.png)

- 브릿지형 이더넷에서 일부 선은 브릿지를 브릿지로 연결하고, 다른 선들은 브릿지를 허브로 연결한다.

- 각 선의 대역폭은 다를 수 있다. 우리의 예제에서 브릿지-브릿지선은 1Gb/s 대역폭을, 네 개의 허브-브릿지 선은 100Mb/s 대역폭을 가진다.

#### 브릿지와 허브의 차이점

- 브릿지는 허브보다 더 높은 전선의 대역폭을 가진다.

- 허브는 각 포트에서 수신한 모든 비트를 종속적으로 다른 모든 포트로 복사한다.

- 브릿지는 우수한 분산 알고리즘을 통해 필요한 경우 선택적으로 하나의 포트에서 다른 포트로 프레임을 복사한다.

### WAN (Wide Area Network)

계층구조의 상부에서 다수의 비호환성 LAN들은 라우터라고 부르는 특별한 컴퓨터에 의해서 연결될 수 있으며, **라우터는 네트워크 간 연결을 구성한다(상호연결 네트워크)**. 각 라우터는 이들이 연결되는 각 네트워크에 대해 어댑터(포트)를 가지고 있다. **라우터는 또한 고속의 point-to-point 전화 연결을 할 수 있으며, 이들은 WAN이라고 하는 네트워크의 사례다**. 이 이름은 이들이 LAN보다 지리적으로 더 넓은 지역에서 운용되기 때문에 불리게 되었다.

### internet

**internet의 중요한 특성은 이것이 매우 다르고 비호환적인 기술을 갖는 여러 가지 LAN과 WAN들로 이루어져 있다는 점이다**. internet 프로토콜은 두 가지 기본 기능을 제공해야 한다.

- 명명법(Naming Scheme) : internet 프로토콜은 호스트 주소를 위한 통일된 포맷을 정의해서 이 차이점들을 줄인다.

- 전달기법(Delivery Mechanism) : internet 프로토콜은 데이터 비트를 패킷(packet)이라고 부르는 비연속적인 단위로 묶는 통일된 방법을 정의해서 이 차이점을 줄인다. 패킷은 패킷 크기와 소스 및 목적지 호스트 주소를 포함하는 헤더와 소스 호스트가 보낸 데이터 비트를 포함하는 데이터로 구성된다. 이것을 Datagram이라고도 부른다.

전달기법 관련하여서 패킷화하는 과정은 아래 사진과 같다. 여기서 말하는 Datagram은 (1) → (2) 로 가는 방법이다.

![인터넷 패킷화 과정](/assets/img/blog/computerscience/internetpacket.png)

## 글로벌 IP 인터넷

인터넷 클라이언트 - 서버 응용의 기본적인 하드웨어 및 소프트웨어 구조는 아래 사진과 같으며, 이 구조는 1980년대 이후로 안정적이었다.

![클라이언트 서버 구조](/assets/img/blog/computerscience/clientserverstructure.png)

각 인터넷 호스트는 TCP/IP 프로토콜(Transmission Control Protocol/Internet Protocol)을 구현한 소프트웨어를 실행하며, 이것은 거의 모든 현대 컴퓨터 시스템에서 지원되고 있다.

관련한 프로토콜 HTTP/TCP/UDP 관련하여서는 [HTTP, TCP, UDP](../../computersystem/http-tcp-udp){:.heading.flip-title}을 참고하면 된다.

### IP 주소

IPv4 주소는 비부호형 32비트 정수다. 네트워크 프로그램은 IP주소를 아래와 같은 IP주소 구조체에 저장한다.

~~~c
/* IP address structure */
struct in_addr {
uint32_t s_addr; /* Address in network byte order (big-endian) */
};
~~~

TCP/IP는 네트워크 패킷 헤더에 포함되는 IP 주소 같은 모든 정수형 데이터 아이템에 대해서 통일된 **Network Byte Order(Big Endian 바이트 순서)**를 정의한다.

- Big Endian : 가장 큰 자릿값(최상위 바이트)을 가장 낮은 주소에, 그 다음 바이트를 뒤쪽 주소에 차례로 저장

- Little Endian : 반대로 최하위 바이트를 가장 낮은 주소에 두고, 큰 바이트로 갈수록 주소가 커짐. x86, ARM(Android/iOS) 같은 현대 CPU 대부분이 사용함

#### Dotted-decimal

IP 주소는 대게 사람들에게 dotted-decimal 표기라고 하는 형식으로 제시되며, 이것은 각 바이트가 십진수 값을 사용하고 다른 바이트들과는 점을 사용해서 구분된다. 아래와 같은 것이 dotted-decimal이다.

~~~linux
linux> hostname -i
128.2.210.175
~~~

### 인터넷 도메인 이름

**DNS(Domain Name System)** 데이터베이스는 수백만 개의 호스트 엔트리로 구성되어 있으며, 이들 각각은 도메인 이름의 집합과 IP 주소 집합 사이의 매핑을 정의한다.

4가지의 매핑을 아래에 나열하겠다.

1. 가장 간단한 경우로, 도메인 이름과 IP 주소 사이의 일대일 매핑

~~~linux
linux> nslookup whaleshark.ics.cs.cmu.edu
Address: 128.2.210.175
~~~

2. 다수의 도메인 이름이 동일한 IP 주소에 매핑

~~~linux
linux> nslookup cs.mit.edu
Address: 18.62.1.6

linux> nslookup eecs.mit.edu
Address: 18.62.1.6
~~~

3. 다수의 도메인 이름들은 다수의 IP 주소로 매핑

~~~linux
linux> nslookup www.twitter.com
Address: 199.16.156.6
Address: 199.16.156.70
Address: 199.16.156.102
Address: 199.16.156.230

linux> nslookup twitter.com
Address: 199.16.156.102
Address: 199.16.156.230
Address: 199.16.156.6
Address: 199.16.156.70
~~~

4. 마지막으로, 일부 유효한 도메인 이름들은 어떤 IP 주소에도 매핑되어 있지 않다.

~~~linux
linux> nslookup edu
*** Can’t find edu: No answer
linux> nslookup ics.cs.cmu.edu
*** Can’t find ics.cs.cmu.edu: No answer
~~~

### 인터넷 연결

**인터넷 client와 server는 연결(connection)을 통해서 바이트 스트림을 주고받는 방식으로 통신한다**. 이 연결은 두개의 프로세스를 연결한다는 점에서 **point-to-point 연결**이다. 데이터가 동시에 양방향으로 흐를 수 있다는 의미에서 이것은 **완전양방향(full-duplex)**이다.

- 소켓(Socket)은 연결의 종단점이다.

- 각 소켓은 인터넷 주소와 16비트 정수 포트로 이루어진 소켓 주소를 가지며, 이것은 address : port로 나타낸다.

- 클라이언트의 소켓 주소 내의 포트는 클라이언트가 연결 요청을 할 때 커널이 자동으로 할당하며, 이것은 단기(Ephemeral) 포트라고 한다.

- 서버는 서비스에 연관된 포트를 사용한다.

인터넷 연결같은 경우 두 개의 종단점의 소켓 주소에 의해 유일하게 식별된다. 이 두개의 소켓 주소는 소켓 쌍이라고 알려져 있으며, 아래와 같이 tuple로 나타낸다.

![인터넷 연결의 구조](/assets/img/blog/computerscience/internetconnection.png)

## 소켓 인터페이스

소켓 인터페이스는 네트워크 응용을 만들기 위한 Unix I/O 함수들과 함께 사용되는 함수들의 집합이다.

### 소켓 주소 구조체

**리눅스 커널의 관점에서 보면, 소켓은 통신을 위한 끝점이다. Unix 프로그램의 관점에서 보면 소켓은 해당 식별자를 가지는 열린 파일이다**.

- Unix 프로그램에서는 network도 파일처럼 여겨진다.

인터넷 소켓 주소는 아래 코드와 같이 

~~~c
/* IP socket address structure */
struct sockaddr_in {
uint16_t sin_family; /* Protocol family (always AF_INET) */
uint16_t sin_port; /* Port number(16bit) in network byte order */
struct in_addr sin_addr; /* IP address in network byte order */
unsigned char sin_zero[8]; /* Pad to sizeof(struct sockaddr) */
};

/* Generic socket address structure (for connect, bind, and accept) */
struct sockaddr {
uint16_t sa_family; /* Protocol family */
char sa_data[14]; /* Address data */
};
~~~