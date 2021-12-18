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

# Establishing a connection
three way handshake는 커넥션을 맺는 과정이다. 이를 통해 커넥션 실패 가능성을 줄일 수 있다. 이러한 체크 과정을 거치면서 메모리와 메시지의 트레이드 오프가 발생한다.

가장 단순한 three way handshake는 아래의 예시와 같다.

## Basic 3-Way Handshake for Connection Synchronization

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
      
                Basic 3-Way Handshake for Connection Synchronization
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
  
                  Simultaneous Connection Synchronization
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
  
                      Recovery from Old Duplicate SYN
```

위의 내용은 old duplicates를 복구하는 예시이다.
3번에서 old duplicate SYN 신호가 TCP B에 도착했지만, TCP B는 이것이 old duplicate라고 말할 수 없다 (4번).
TCP A는 ACK 필드가 유효하지 않다는 것을 발견하고 RST (reset)을 반환한다.
TCP B는 RST를 수신하여 LISTEN 상태로 돌아간다.
6번에서 마침내 original SYN이 도달하면, synchronization 절차가 정상적으로 진행된다.
만약 6번의 SYN 신호가 RST 이전에 도달하게 되면, 양방향으로 RST 신호가 오가면서 좀더 복잡한 신호교환 절차가 일어날 것이다.

## Half-Open Connections and Other Anomalies

TCP중 하나가 closed 또는 aborted 되거나, 두개의 커넥션이 메모리 손실의 결과로 동기화가 깨지게 되었을때는 half-open 커넥션 수립이 이루어진다.
이러한 커넥션은 양방향에서 데이터를 전송하려는 시도가 있으면 자동으로 reset된다.
half-open 커넥션은 이렇지 않은 비정상적인 상황에서 사용되고, 리커버리 절차도 포함된다.

만약 A의 커넥션이 더이상 존재하지 않을때, B에 있는 유저가 데이터를 보내려고 하면 B에서는 reset 메시지를 받게 된다. 이러한 메시지는 B TCP에게 뭔가 잘못되었다는 것을 알릴 수 있고, 이것으로 커넥션 중단을 예상할 수 있다. 

A의 TCP가 메모리 손실로 인해 crash가 발생한 상황에서 두개의 프로세서 A와 B가 서로 통신을 하고있다고 가정해보자.
이때 A의 TCP를 지원하는 운영체제는 오류 복구 메커니즘이 존재한다면, TCP가 OS에 의해 회복되었을때, A는 되돌려진 부분부터 다시 작업을 수행한다.

결과적으로, A는 다시 커넥션을 OPEN하려 하거나, 이미 연결되어있다고 믿고있어서 SEND를 하려 할 것이다.
후자의 경우, 로컬(A) TCP로부터 'connection not open' 에러 메시지를 받을 것이다.
커넥션을 수립하려고 시도하는 과정에서, A의 TCP는 SYN이 포함된 세그먼트를 보낼 것이다.

이러한 시나리오는 아래에 표시되어있다.

A가 crash된 이후, 유저는 커넥션을 re-open하려 시도한다.
이러한 과정동안 TCP B는 커넥션이 open되어있다고 생각할 것이다.

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

3번 과정에서 SYN이 TCP B에 도착하면, TCP B는 synchronized 상태가 되고 창(window)외부에서 들어오는 세그먼트는 다음에 들을 것으로 예상되는 시퀀스(ACK 100)을 나타내는 승인으로 응답한다.
TCP A는 이 세그먼트가 아무것도 승인하지 않음을 확인한다.
전송되고 동기화되지 않은 상태에서 half open을 감지했기 때문에 RST을 전송한다.

5번 과정에서 TCP B는 abort되있다. TCP A는 계속해서 커넥션을 수립하려고 한다. 이제 해당 과정은 위에 설명한 basic 3-way handshake의 과정을 통해 커넥션을 수립하게 된다.

흥미로운 대안 사례는 TCP A가 crash되었지만 TCP B가 sync되어있는 줄 알고 데이터를 보내려 할때 발생한다.
아래의 예시를 보자. 2번 과정에서 TCP B로부터 보낸 데이터가 TCP A에 도착한다. 하지만 이것은 받아들여질 수 없는데 connection이 존재하지 않기 때문이다. 그러므로 TCP A는 RST을 보내게 되고, TCP B는 해당 커넥션을 abort한다.

```
        TCP A                                              TCP B

  1.  (CRASH)                                   (send 300,receive 100)

  2.  (??)    <-- <SEQ=300><ACK=100><DATA=10><CTL=ACK> <-- ESTABLISHED

  3.          --> <SEQ=100><CTL=RST>                   --> (ABORT!!)

           Active Side Causes Half-Open Connection Discovery
```

아래의 과정은 두개의 TCP A와 B가 서로 SYN을 기다리는 수동적인 커넥션을 가지고 있는 상황이다. 2번 과정에서 old duplicate가 TCP B에 도달 했고, B는 3번 과정에서 SYN-ACK을 리턴했다. 하지만 A는 이를 수락하지 않고 RST을 보냈고, TCP B는 리셋처리를 하고 LISTEN 상태를 반환한다.

```
    TCP A                                         TCP B

  1.  LISTEN                                        LISTEN

  2.       ... <SEQ=Z><CTL=SYN>                -->  SYN-RECEIVED

  3.  (??) <-- <SEQ=X><ACK=Z+1><CTL=SYN,ACK>   <--  SYN-RECEIVED

  4.       --> <SEQ=Z+1><CTL=RST>              -->  (return to LISTEN!)

  5.  LISTEN                                        LISTEN

       Old Duplicate SYN Initiates a Reset on two Passive Sockets
```

이것 말고도 다양한 케이스가 가능하다. 그러한 모든 케이스는 아래의 RST 생성과 처리과정을 따른다.

## Reset Generation

현재의 커넥션과 상관이 없는 세그먼트가 도착했을때, reset(RST)은 꼭 보내져야한다.
위의 경우가 아니라면, reset은 보내지면 안된다.

상태와 관련된 아래의 세가지 그룹을 보자.

1. 만약 커넥션이 존재하지 않는다면 (CLOSED) 모든 incoming segment에 대해 응답으로 reset을 보낸다. 예외적으로, 존재하지 않는 커넥션으로 주소가 지정된 SYN은 이 방법으로 리젝된다. <br> 
incoming segment가 ACK 필드를 가진다면, reset은 이 segment가 가지고있던 시퀀스 넘버를 가진다. 그렇게 하지 않으면 reset은 시퀀스 넘버를 0으로 하고, ACK 필드는 incoming segment의 시퀀스 넘버와 incoming segment의 길이를 합친 값으로 설정된다.
2. 만약 커넥션이 non-synchronized 상태(LISTEN, SYN-SENT, SYN-RECEIVED)중 하나이고, incoming segment의 ACK가 지금까지 보내온 적 없는 값이거나 incoming segment가 요청된 보안레벨(security level) 및 구획(compartmnet)과 정확히 일치하지 않는 보안레벨 또는 구획이 있는 경우 reset이 전송된다. <br>
만약 우리가 보낸 SYN이 인식되지 않고, incoming segment의 우선순위가 요청된 우선순위보다 높다면, 로컬의 우선순위를 높이거나(만약 유저나 시스템이 허락하는 경우), reset을 보낸다. 만약 incoming segment의 우선순위가 요청된 우선순위보다 낮다면, 우선순위가 같아질때까지 계속한다 (만약 remote TCP가 우리의 우선순위가 맞추기 위해 우선순위를 올릴 수 없다면, 다음 segment 전송에서 이를 탐지하여 커넥션이 terminate 된다).<br>
만약 incoming segment가 ACK 필드를 가진다면, reset은 시퀀스 넘버는 incoming segment의 ACK 필드에 있는 값으로 한다. 그렇지 않으면 reset은 시퀀스 넘버를 0으로 하고 ACK 필드는 incoming segment의 시퀀스 넘버와 incoming segment의 길이를 합친 값으로 설정된다. 이 커넥션은 동일한 상태로 유지된다.
3. 만약 커넥션이 synchronized 상태라면(ESTABLISHED, FIN-WAIT-1, FIN-WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK, TIME-WAIT), 허용하지 않는 세그먼트(out of window sequence number, unacceptible acknowledgment number)는 현재 전송 시퀀스 넘버와 수신될 것으로 예상되는 다음 시퀀스 넘버를 나타내는 ACK를 포함하는 empty acknowledgment 세그먼트만 이끌어내야 하며 연결은 동일한 상태로 유지된다. <br>
만약 incoming segment가 요청된 커넥션의 보안레벨, 구획, 우선순위와 맞지 않는 보안레벨 또는 구획 또는 우선순위가 있는 경우, reset이 전송되고 커넥션은 CLOSED 상태가 된다. reset은 incoming segment의 ACK 필드 값을 시퀀스 넘버로 한다.

## Reset Processing

SYN-SENT상태를 제외한 모든 상태에서 모든 reset segment는 SEQ 필드를 확인하여 유효성 검사를 한다. 만약 시퀀스 넘버가 window안에 있다면 reset은 유효하다. SYN-SENT 상태에서 (initial SYN에 대해 RST가 수신되었을떄), ACK 필드가 SYN을 승인하면 RST이 허용된다.

RST 수신자는 먼저 이를 검증한 다음, 상태를 변경한다. 만약 수신자가 LISTEN 상태였다면, 이것을 무시합니다. 만약 수신자가 SYN-RECEIVED 상태였고, 이전에 LISTEN 상태였다면, 수신자는 LISTEN 상태로 돌아온다. 그렇지 않은 경우라면 수신자는 커넥션을 abort하고 CLOSED 상태가 된다. 만약 수신자가 이 외의 상태였다면, 커넥션을 abort하고 유저에게 이를 알리고 CLOSED 상태가 된다.

# Closing Connection

CLOSE는 "나는 더이상 보낼 데이터가 없어" 라는 뜻을 가지는 작업이다.

기본적으로 세가지 케이스가 존재한다:

1) 로컬 유저가 CLOSE하겠다고 말한다.
2) remote TCP가 FIN control signal을 보낸다.
3) 두 유저 모두 동시에 CLOSE 한다.

Case 1: Local user initiates the close

이 케이스에서, FIN segment가 구성되고 외부로 나가는 segment 큐에 들어간다. 이제 모든 SEND는 받아들여지지 않고, FIN-WAIT-1 상태가 된다. 해당 상태에서 RECEIVE는 허용된다. FIN을 포함하는 모든 segment는 확인될때까지 재전송된다. 다른 TCP가 FIN을 확인하고 자신의 FIN을 보낸 경우, 첫번째 TCP는 이 FIN을 ACK할 수 있다. FIN을 수신한 TCP는 이를 ACK 하지만, 사용자가 연결을 CLOSE 할 때까지 자신의 FIN을 보내지 않습니다.

Case 2: TCP receives a FIN from the network

요청하지 않은 FIN이 네트워크에 도착하면, 수신 TCp는 이를 ACK하고 사용자에게 커넥션이 닫히고있다는 것을 알릴 수 있다. 사용자는 CLOSE로 응답하고, TCP는 나머지 데이터를 보낸 후 FIN을 보낼 수 있다. 그 후 TCP는 자신의 FIN이 승인될 때 까지 기다렸다가 연결을 삭제한다. ACK이 오지 않으면, 시간초과 후 커넥션이 abort되고 사용자에게 알린다.

Case 3: both users close simultaneously

연결되어있는 두 사용자가 동시에 CLOSE하면 FIN segment가 교환된다. FIN 앞의 모든 세그먼트가 처리되고 승인되면 각 TCP는 수신한 FIN을 ACK 할 수 있다. 둘 다 이러한 ACK을 받으면 연결을 삭제한다.

```
      TCP A                                                TCP B

  1.  ESTABLISHED                                          ESTABLISHED

  2.  (Close)
      FIN-WAIT-1  --> <SEQ=100><ACK=300><CTL=FIN,ACK>  --> CLOSE-WAIT

  3.  FIN-WAIT-2  <-- <SEQ=300><ACK=101><CTL=ACK>      <-- CLOSE-WAIT

  4.                                                       (Close)
      TIME-WAIT   <-- <SEQ=300><ACK=101><CTL=FIN,ACK>  <-- LAST-ACK

  5.  TIME-WAIT   --> <SEQ=101><ACK=301><CTL=ACK>      --> CLOSED

  6.  (2 MSL)
      CLOSED

                         Normal Close Sequence
```

```

      TCP A                                                TCP B

  1.  ESTABLISHED                                          ESTABLISHED

  2.  (Close)                                              (Close)
      FIN-WAIT-1  --> <SEQ=100><ACK=300><CTL=FIN,ACK>  ... FIN-WAIT-1
                  <-- <SEQ=300><ACK=100><CTL=FIN,ACK>  <--
                  ... <SEQ=100><ACK=300><CTL=FIN,ACK>  -->

  3.  CLOSING     --> <SEQ=101><ACK=301><CTL=ACK>      ... CLOSING
                  <-- <SEQ=301><ACK=101><CTL=ACK>      <--
                  ... <SEQ=101><ACK=301><CTL=ACK>      -->

  4.  TIME-WAIT                                            TIME-WAIT
      (2 MSL)                                              (2 MSL)
      CLOSED                                               CLOSED

                      Simultaneous Close Sequence
```
