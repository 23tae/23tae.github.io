---
title: "NetPractice ② 배경 지식"
categories: [42Seoul, NetPractice]
tags: [cs]
date: 2023-01-15
---

# 네트워크 구조 (OSI 7계층)

![Untitled](/assets/img/42seoul/netpractice/netpractice2/Untitled.png)

- 네트워크 프로토콜이 통신하는 구조를 7개의 계층으로 분리한 것

## 1계층 : 물리 계층 (Physical layer)

- 전기적, 기계적, 기능적인 특성을 이용해서 **통신 케이블로** 데이터를 전송
- 전송 단위(Protocol Data Unit, PDU) : 비트
- 데이터를 전달만 하는 역할
- 주요 장비 : 통신 케이블, 리피터, 허브

## 2계층 : 데이터링크 계층 (Datalink layer)

- 송수신되는 정보의 오류와 흐름을 관리
- **MAC 주소**로 통신
- 전송 단위 : **프레임**
    - 이더넷에서 주고받는 데이터의 최소 단위
    - MAC 프레임, 이더넷 프레임 등으로 불림
- 주요 장비 : 브리지, 스위치

## 3계층 : 네트워크 계층 (Network layer)

- **라우팅**을 통해 데이터를 목적지까지 안전하고 빠르게 전달
- 역할 : 라우팅, 흐름 제어, 세그멘테이션(segmentation/desegmentation), 오류 제어, 인터네트워킹(Internetworking) 등
- 전송 단위 : **패킷**
    - 모든 프로토콜 스택에서 네트워크를 오가는 데이터의 단위로 사용되는 일반 용어
- 주요 장비 : 라우터, layer3 스위치(layer2 스위치에 라우팅 기능 포함됨)

## 4계층 : 전송 계층 (Transport layer)

- **End-to-End Reliable Delivery** : 종단 간 신뢰성 있는 데이터 전송을 담당.
    - 이를 위해 분할과 재조합, 연결제어, 흐름제어, 오류제어, 혼잡제어를 수행함.
- **Process-To-Process Communication** : 종단(Host)의 구체적인 목적지(Process)까지 데이터가 도달할 수 있도록 함.
- 목적지를 특정하기 위해 **포트 번호**를 사용.
- 전송 단위 : **세그먼트**
- 주요 프로토콜 : TCP, UDP
- 주요 장비 : L4 스위치

## 5계층 : 세션 계층 (Session layer)

- 응용 프로그램간의 논리적인 연결(세션) 생성 및 제어를 담당.
    - 두 컴퓨터 간의 대화나 세션을 관리.
- 전송 단위 : 데이터 또는 메시지

## 6계층 : 표현 계층 (Presentation layer)

- 데이터 표현 방식, 상이한 부호 체계 간의 변화에 대해 규정.
- 인코딩/디코딩, 압축/해제, 암호화/복호화 등의 역할을 수행함.
    - 응용 계층으로부터 전달받은 데이터를 읽을 수 있는 형식으로 변환.
- 전송 단위 : 데이터

## 7계층 : 응용 계층 (Application layer)

- 사용자가 네트워크 자원에 접근하는 방법을 제공.
- 전송 단위 : 데이터
- 주요 프로토콜 : TELNET, FTP, SMTP, HTTP 등

# 인터넷 프로토콜 (IP, Internet Protocol)

- 개념
    - TCP/IP 기반의 인터넷 망을 통하여 **데이터그램의 전달**을 담당하는 프로토콜
- 주요 기능
    - IP 주소 지정 (**Addressing**)
    - IP 계층에서 IP 패킷의 라우팅 대상이 됨 (**Routing**)
- 특징
    - 신뢰성(에러제어) 및 흐름제어 기능이 전혀 없음(Best-Effort Service)
    - **비연결성** 데이터그램 방식(Connectionless)
    - 패킷의 완전한 전달(소실,중복,지연,순서바뀜 등이 없게함)을 보장 않음(Unreliable)
    - 경우에 따라 **단편화**가 필요함.
    - 모든 **상위 계층 프로토콜**(TCP,UDP,ICMP,IGMP 등)이 **IP 데이터그램**에 실려서 전송됨
- IP 헤더
    - 개념 : IP 데이터그램(패킷)의 앞부분에서 주소 등 각종 제어 정보를 담고 있는 부분.
        - IP 데이터그램 = IP 헤더 + 데이터
    - 특징
        - 수신 및 발신 주소를 포함함.
        - 바이트 전달 순서 : 최상위 바이트(MSB)를 먼저 보냄(Big-endian)

## IP 주소

- 숫자로 이루어지며 **인터넷에 연결된 장치를 식별**하는 역할을 함.
    
    ![Untitled](/assets/img/42seoul/netpractice/netpractice2/Untitled%202.png)
    

### IPv4, IPv6

- 프로토콜 버전에 따라 IPv4와 IPv6로 구분됨.
    - IPv4 : 각 부분이 0~255 사이의 10진수로 구성됨.
    - IPv6 : 숫자와 알파벳이 혼합된 16진법으로 구성됨.
        - IPv4와 달리 암호화와 인증 기능을 제공.

### 공인 / 사설 IP

- IP 주소에는 공인 IP 주소, 사설 IP 주소, 동적 IP 주소, 정적 IP 주소 등이 있음.
- **공인 IP(public IP) 주소**
    - 인터넷 업체가 사용자에게 할당한 외부 IP 주소
    - 공유기가 인터넷과 통신하도록 함.
    - 동일한 인터넷에 연결된 장치들은 IP 주소를 공유함.
    - 정적/동적 IP 주소, 전용/공유 IP 주소로 구분됨.
        - 동적 IP 주소
            - 시간이 지남에 따라 변화하는 IP 주소
            - 인터넷 업체가 할당
            - 장치를 재부팅하거나, 새로운 장치를 네트워크에 추가하거나, 네트워크 설정을 수정할 때마다 변경됨.
        - 정적 IP 주소
            - 변화하지 않는 IP 주소
            - 웹사이트를 호스팅하거나 이메일 및 FTP 서비스를 제공하는 서버에서 할당
            - 안정적인 인터넷 연결과 웹 주소의 일관성을 유지해야 하는 공공 기관에서 사용
- **사설 IP(private IP) 주소**
    - **공유기**가 **홈 네트워크**에 연결된 장치에 할당한 내부 IP 주소
    - 로컬 네트워크에서 할당되며 다른 네트워크의 IP 주소와 중복될 수 있음.
        - 내부 네트워크에서만 사용돼서 문제가 없기 떄문.
    - 다음 사설 IP 주소 대역 내의 주소가 할당됨.
        - 클래스 A: 10.0.0.0~10.255.255.255
        - 클래스 B: 172.16.0.0~172.31.255.255
        - 클래스 C: 192.168.0.0~192.168.255.255
- 인터넷에 연결된 장치에는 공인 IP 주소와 사설 IP 주소 모두 할당됨.
    - IP 주소가 장치 수에 비해 부족하기 때문.

# 서브넷 (Subnet)

- IP 주소에서 네트워크 영역을 부분적으로 나눈 **부분 네트워크**
- 참고 : [[네트워크] 04. The Network Layer: Data Plane](https://23tae.github.io/posts/network04/#ipv4-%EC%A3%BC%EC%86%8C-%EC%A7%80%EC%A0%95ipv4-addressing)

## 서브넷 마스크 (Subnet Mask)

- IP 주소의 **네트워크 ID**와 **호스트 ID**를 **분리**하는 역할
- 각 클래스별 **기본 서브넷 마스크**
    
    ![Untitled](/assets/img/42seoul/netpractice/netpractice2/Untitled%203.png)
    
- `IP주소 & 서브넷 마스크` = **네트워크 ID**
    
    ![Untitled](/assets/img/42seoul/netpractice/netpractice2/Untitled%204.png)
    
- `192.168.32.0/24` 에서 `/24`
    - 서브넷 마스크의 bit 개수
    - 서브넷 마스크의 왼쪽부터 1이 24개임을 의미.

## 서브넷팅 (Subnetting)

- IP 주소 낭비를 방지하기 위해 원본 네트워크를 여러개의 서브넷으로 분리하는 과정
    - 이렇게 해서 분리된 네트워크 단위가 **서브넷**.

## 네트워크 / 브로드캐스트 주소

### 네트워크 주소

- 하나의 **네트워크를 통칭**하기 위한 주소
- 해당 네트워크의 첫 번째 IP 주소
- 계산 방법 : IP 주소와 서브넷 마스크를 AND 연산

### 브로드캐스트 주소

- 네트워크에 있는 모든 클라이언트들에게 데이터를 보내기(**브로드캐스팅**) 위한 주소
- 해당 네트워크의 맨 마지막 주소
- 계산 방법 : 호스트 IP(서브넷 마스크의 오른쪽의 0인 비트)를 모두 1로 변경

# 라우터 / 스위치

## 라우터

![Untitled](/assets/img/42seoul/netpractice/netpractice2/Untitled%205.png)

- 컴퓨터 네트워크 간에 데이터 패킷을 전송하는 네트워크 장치
    - **서로 다른 네트워크**를 연결한다.
- 패킷의 위치를 추출, 해당 위치로의 최적 경로를 지정.(**라우팅**)
- 데이터 패킷을 다음 장치로 전달.(**포워딩**)
- 3계층인 **네트워크 계층**에서 동작함.

## 스위치

![Untitled](/assets/img/42seoul/netpractice/netpractice2/Untitled%206.png)

- 한 네트워크 상에 있는 여러 장치를 연결한다.
- **데이터 링크 계층**에서 동작함.
- 다수의 장비를 연결 및 통신을 중재함.
- 다수의 장비를 연결하고 케이블을 한곳으로 집중시킴.
- 통신을 중재함.
- 스위칭 **허브**라고도 불림.
- **목적지 MAC 주소**를 통해 위치 파악 후, 해당 위치까지 연결된 포트로만 전기신호를 보냄.

## 비교

| 기준 | 라우터 | 스위치 |
| --- | --- | --- |
| 목적 | 서로 다른 네트워크를 연결함 | 한 네트워크 상의 여러 장치를 연결함 |
| 계층 | 네트워크 계층에서 작동 | 데이터 링크 계층 및 네트워크 계층에서 작동 |
| 작업 | 패킷이 대상 컴퓨터에 도달하기 위해<br>따라야 할 최상의 경로를 결정함 (**라우팅**) | 프로세스를 수신하여 패킷을<br>적절한 출력 포트로 전달함 (**스위칭**) |
| 유형 | 적응형 라우팅, 비적응형 라우팅 | 회로 스위칭, 패킷 스위칭, 메시지 스위칭 |

# 기타

## 프레임 / 패킷

![Untitled](/assets/img/42seoul/netpractice/netpractice2/Untitled%201.png)

### 주요 개념

| 기준 | 프레임 | 패킷 |
| --- | --- | --- |
| 기본 개념 | 데이터 링크 계층 프로토콜의 데이터 단위 | 네트워크 계층 프로토콜의 데이터 단위 |
| OSI 계층 | 데이터 링크 계층 | 네트워크 계층 |
| 포함된 정보 | 출발지와 도착지의 MAC 주소 | 출발지와 도착지의 IP 주소 |
| 상관관계 | 프레임 내부에 패킷을 캡슐화 함 | 패킷 내부에 세그먼트를 캡슐화 함 |

## 네트워크 인터페이스
- 두 가지 네트워크 장비 또는 프로토콜 계층이 연결되는 지점

# Ref.

[OSI 7계층 - IT위키](https://itwiki.kr/w/OSI_7계층)  
[OSI 7 계층이란?, OSI 7 계층을 나눈 이유 :: effortDev](https://shlee0882.tistory.com/110)  
[OSI 7 계층 - 해시넷](http://wiki.hash.kr/index.php/OSI_7_%EA%B3%84%EC%B8%B5)  
[The relationship between the Maximum Transmission Unit (MTU) and the Maximum Segment Size (MSS)](https://crnetpackets.com/2016/01/27/the-relation-between-maximum-transmission-unit-mtu-and-the-maximum-segment-size-mss/)  
[패킷과 프레임의 차이](https://jacking75.github.io/network_packet_frame/)  
[공인 IP와 사설 IP... 다양한 IP 유형의 차이는?](https://nordvpn.com/ko/blog/public-ip-and-private-ip/)  
[[네트워크] 서브넷, 서브넷마스크, 서브넷팅이란?](https://code-lab1.tistory.com/34)  
[서브넷과 Broadcast 주소](https://velog.io/@dnstlr2933/%EC%84%9C%EB%B8%8C%EB%84%B7%EA%B3%BC-Broadcast-%EC%A3%BC%EC%86%8C)  
[라우터와 스위치의 차이점](https://ko.gadget-info.com/difference-between-router)  
[라우터 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EB%9D%BC%EC%9A%B0%ED%84%B0)  
[네트워크 통신하기](https://catsbi.oopy.io/eec728e7-0a31-4c96-9d33-20421bd5e6b3)  
[네트워크 연결과 구성요소](https://catsbi.oopy.io/75315c4f-20d7-4196-a239-363f374f7727)  
[https://infinity-cable-products.com/blogs/hardware/what-is-a-network-switch](https://infinity-cable-products.com/blogs/hardware/what-is-a-network-switch)  
[https://www.lifewire.com/what-is-a-router-2618162](https://www.lifewire.com/what-is-a-router-2618162)
