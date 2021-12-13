---
title: "IPv4의 네트워크 패킷 주소와 각 필드의 역할"
subtitle: "TCP/IP를 정리하며"
tags:
    - network
date: 2021-12-13
---
패킷은 제어정보와 사용자 데이터로 이루어지며 이를 페이로드라고도 한다.
패킷은 정보기술에서 컴퓨터 네트워크가 전달하는 데이터의 형식화된 블록이다.
패킷을 지원하지 않는 컴퓨터통신 연결은 단순히 바이트 문자열 비트를 독립적으로 연속하여 데이터를 전송한다.
데이터가 패킷으로 형식이 바뀔때 네트워크는 장문메시지를 더 효과적이고 신뢰성있게 보낼수 있다.

<img src="https://user-images.githubusercontent.com/34048253/145793779-2da05d3e-8fe2-4803-b593-cd03b82453d4.jpeg" width=300  alt="ipv4의 구조" />

- version (4 bits): 현재로는 버전 4를 사용
- Header Length (4 bits): 헤더의 길이
- Type of Service (Tos) Flag (8 bits): 요구되는 서비스 품질을 나타냄
  - 우선순위 설정용 (0~2 bit): Precedence (우선순위 8단계)
  - TOS 설정용
    - Bit 3: Delay (0: 보통의 지연, 1: 높은 지연)
    - Bit 4: Throughput (0: 보통 처리율, 1: 높은 처리율)
    - Bit 5: Reliability (0: 보통 신뢰성, 1: 높은 신뢰성)
    - Bit 6: Minimum Cost (최소 비용)
  - Bit 7: 항상 0
- Total Packet Length (16 bits): IP 헤더 및 데이터를 포함한 IP 패킷 전체 길이를 바이트 단위로 길이를 표시. (최대값은 65,535 = 2^16 - 1)
- Fragment Identifier (16 bits): 각 조각이 동일한 데이터그램에 속하면 같은 일련번호를 공유함
  - Datagram: 패킷교환에서 각각 독립적으로 췩브되는 각각의 패킷
- Fragmentation Flag (3 bits) 분열의 특성을 나타내는 플래그
  - Bit 1: 미사용 (항상 0)
  - Bit 2: D F bit (Don't Fragment)
    - 0: 분열
      - 라우터에서도 분열이 가능함을 뜻함
    - 1: 미분열
      - 목적지 컴퓨터가 조각들을 다시 모을 능력이 없기 때문에 중간에 라우터로 하여금 데이터그램을 단편화 하지 말라는 뜻
  - Bit 3: M F bit (More Fragment)
    - 현재의 조각이 마지막이면 0
    - 더 많은 조각이 뒤에 계속 있으면 1
- Fragmentation Offset (13bits): 8바이트 단위(2워드)로 최초의 분열조각으로부터 어떤 곳에 붙여야하는지 위치를 나타냄.
  - 각 조각들이 순서가 바뀌어 도착할 수도 있기 때문에 이 필드가 중요함.
- TTL, Time To Live (8 bits): IP 패킷 전달, DNS 캐싱 등에서 생존 시간
- Protocol Identifier (8 bits): 어느 상위계층 프로토콜이 데이터 내에 포함되었는가를 보여줌
  - 1: ICMP, 2: IGMP, 6: TCP, 8: EGP, 17: UDP, 89: OSPF 등
- Header Checksum (16 bits): 헤더 오류 검출
- Source IP Address (32 bits): 송신처 IP 주소
- Destination IP Address (32 bits): 목적지 IP 주소
- Padding: 기본 블록 크기/길이를 맞추기 위한 처리
  - 마지막 블록이 블록 길이에 미치지 못할 경우에, 적당한 비트열을 채워넣음
  - 단순히 0을 붙이는 것은 아님
