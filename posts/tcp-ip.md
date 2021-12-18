---
title:  "TCP/IP 정리"
subtitle: "3 way handshake의 상황별 처리방법에 대해 알아보자 "
tags:
    - network
date:   2021-12-18
---

> https://datatracker.ietf.org/doc/html/rfc793

TCP/IP는 현재 인터넷 프로토콜 중 가장 많이 쓰이고있다.

TCP/IP는 패킷 통신 방식의 인터넷 프로토콜인 IP(internet protocol)와 전송 조절 프로토콜인 TCP (transmission control protocol)

IP는 패킷 전달 여부를 보증하지 않고, 패킷을 보낸 순서와 받는 순서가 다를 수 있다.
TCP는 IP 위에서 동작하는 프로토콜로, 데이터의 전달을 보증하고 보낸 순서대로 받게 해준다.

HTTP, FTP, SMTP 등 TCP를 기반으로 한 수많은 애플리케이션 프로토콜들이 IP 위에서 동작한다.

# internet protocol
송신 호스트와 수신 호스트가 패킷 교환 네트워크(패킷 스위칭 네트워크)에서 정보를 주고받는 데 사용하는 정보 위주 규약이며, OSI 네트워크 계층에서 호스트의 주소지정과 패킷 분할 및 조립 기능을 담당한다.

IP의 정보는 패킷 혹은 데이터그램이라고 하는 덩어리로 나뉘어 전송된다. IP에서는 이전에 통신한 적 없는 호스트에 패킷을 보낼 때 경로 설정이 필요없다.

IP는 비신뢰성과 비연결성이 특징이다. 비신뢰성은 흐름에 관여하지 않기 때문에 보낸 정보가 제대로 갔는지 보장하지 않는다는 뜻이다. 예를 들어 전송 과정에서 패킷이 손상될 수도 있고, 같은 호스트에서 전송한 패킷의 순서가 뒤죽박죽이 될 수도 있고, 같은 패킷이 두번 전송될 수도 있으며, 아예 패킷이 사라질 수도 있다. 패킷 전송과 정확한 순서를 보장하려면 TCP 프로토콜과 같은 IP의 상위 프로토콜을 이용해야 한다.

<img src="https://upload.wikimedia.org/wikipedia/commons/5/5b/InternetProtocolStack.png" width=300 />

# TCP (전송 제어 프로토콜)

TCP는 인터넷에 연결된 컴퓨터에서 실행되는 프로그램간에 데이터를 안정적으로, 순서대로, 에러없이 교환할 수 있게 한다.

# three way handshake
three way handshake는 커넥션을 맺는 과정이다. 이를 통해 커넥션 실패 가능성을 줄일 수 있다. 이러한 체크 과정을 거치면서 메모리와 메시지의 트레이드 오프가 발생한다.

## Basic 3-Way Handshake for Connection Synchronization
가장 단순한 three way handshake는 아래의 예시와 같다.

```
      TCP A                                                TCP B

  1.  CLOSED                                               LISTEN
      connection이 없는 상태

  2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED
      A가 B에게 100번 sequence number와 함께 SYN 신호를 보냄

  3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED
      B가 A에게 SYN, ACK 신호를 보냄 (이때 A의 커넥션이 맺어짐)

  4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED
      A가 B에게 101번 sequence number와 함께 ACK 신호를 보냄 (이때 B의 커넥션이 맺어짐)

  5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED
      A가 B에게 ACK 신호와 데이터를 보냄
```

이때 2,3,4번 단계를 three way handshake라고 한다.

## Simultaneous Connection Synchronization
동시에 진행되는 커넥션 수립은 약간 더 복잡하다. 아래의 그림처럼, 각각의 TCP 사이클은 CLOSED 상태에서 SYN-SENT 상태가 되었다가 SYN-RECEIVED를 거쳐 ESTABLISHED 된다.

```
      TCP A                                            TCP B

  1.  CLOSED                                           CLOSED

  2.  SYN-SENT     --> <SEQ=100><CTL=SYN>              ...

  3.  SYN-RECEIVED <-- <SEQ=300><CTL=SYN>              <-- SYN-SENT

  4.               ... <SEQ=100><CTL=SYN>              --> SYN-RECEIVED

  5.  SYN-RECEIVED --> <SEQ=100><ACK=301><CTL=SYN,ACK> ...

  6.  ESTABLISHED  <-- <SEQ=300><ACK=101><CTL=SYN,ACK> <-- SYN-RECEIVED

  7.               ... <SEQ=101><ACK=301><CTL=ACK>     --> ESTABLISHED
```

three way handshake의 주된 목적은 old duplicate connections을 방지하는 것이다. 이것을 해결하기 위해, special control message, reset이 고안되었다고 한다. 만약 TCP가 non-synchronized 상태일때(SYN-SENT, SYN-RECEIVED) 메시지를 받았다면(receiving), LISTEN을 반환하여 RESET 할 수 있도록 한다. 만약 TCP가 synchronized상태라면 (ESTABLISHED, FIN-WAIT-1, FIN-WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK, TIME-WAIT) 커넥션을 중단시키고 유저에게 알린다. 이것은 half-open 커넥션이라고 한다.

## Recovery from Old Duplicate SYN

```
      TCP A                                                TCP B

  1.  CLOSED                                               LISTEN

  2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               ...

  3.  (duplicate) ... <SEQ=90><CTL=SYN>               --> SYN-RECEIVED

  4.  SYN-SENT    <-- <SEQ=300><ACK=91><CTL=SYN,ACK>  <-- SYN-RECEIVED

  5.  SYN-SENT    --> <SEQ=91><CTL=RST>               --> LISTEN


  6.              ... <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

  7.  SYN-SENT    <-- <SEQ=400><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

  8.  ESTABLISHED --> <SEQ=101><ACK=401><CTL=ACK>      --> ESTABLISHED
```

위의 내용은 old duplicates를 복구하는 예시이다.
3번에서 old duplicate SYN 신호가 TCP B에 도착했지만, TCP B는 이것이 old duplicate라고 말할 수 없다 (4번).
TCP A는 ACK 필드가 유효하지 않다는 것을 발견하고 RST (reset)을 반환한다.
TCP B는 RST를 수신하여 LISTEN 상태로 돌아갑니다.
6번에서 마침내 original SYN이 도달하면, synchronization 절차가 정상적으로 진행됩니다.
만약 6번의 SYN 신호가 RST 이전에 도달하게 되면, 양방향으로 RST 신호가 오가면서 좀더 복잡한 신호 교환 절차가 일어날 것입니다.

##   Half-Open Connections and Other Anomalies

TCP중 하나가 closed 또는 aborted 되거나, 두개의 커넥션이 메모리 손실의 결과로 동기화가 깨지게 되었을때는 half-open 커넥션 수립이 이루어집니다.
이러한 커넥션은 양방향에서 데이터를 전송하려는 시도가 있으면 자동으로 reset됩니다.
half-open 커넥션은 이렇지 않은 비정상적인 상황에서 사용되고, 리커버리 절차도 포함됩니다.

만약 A의 커넥션이 더이상 존재하지 않을때, B에 있는 유저가 데이터를 보내려고 하면 B에서는 reset 메시지를 받게 됩니다. 이러한 메시지는 B TCP에게 뭔가 잘못되었다는 것을 알릴 수 있고, 이것으로 커넥션 중단을 예상할 수 있습니다. 


A의 TCP가 메모리 손실로 인해 crash가 발생한 상황에서 두개의 프로세서 A와 B가 서로 통신을 하고있다고 가정해봅시다.
이때 A의 TCP를 지원하는 운영체제는 오류 복구 메커니즘이 존재한다고 가정합니다.
TCP가 OS에 의해 회복되었을때, A는 되돌려진 부분부터 다시 작업을 수행합니다.

결고적으로, A는 다시 커넥션을 OPEN하려 하거나, 이미 연결되어있다고 믿고있어서 SEND를 하려 할 것입니다.

후자의 경우, 로컬(A) TCP로부터 'connection not open' 에러 메시지를 받을 것입니다.

커넥션을 수립하려고 시도하는 과정에서, A의 TCP는 SYN이 포함된 세그먼트를 보낼 것입니다.

이러한 시나리오는 아래에 표시되어있습니다.

A가 crash된 이후, 유저는 커넥션을 re-open하려 시도합니다.
이러한 과정동안 TCP B는 커넥션이 open되어있다고 생각할 것입니다.


```
      TCP A                                           TCP B

  1.  (CRASH)                               (send 300,receive 100)

  2.  CLOSED                                           ESTABLISHED

  3.  SYN-SENT --> <SEQ=400><CTL=SYN>              --> (??)

  4.  (!!)     <-- <SEQ=300><ACK=100><CTL=ACK>     <-- ESTABLISHED

  5.  SYN-SENT --> <SEQ=100><CTL=RST>              --> (Abort!!)

  6.  SYN-SENT                                         CLOSED

  7.  SYN-SENT --> <SEQ=400><CTL=SYN>              -->

                     Half-Open Connection Discovery
```
