---
title: "네이글 알고리즘(Nagle's algorithm)이란?"
date: 2023-04-01
categories: [Computer Science, Computer Network]
tags: [cs, til, network]
---

# 개요

네이글 알고리즘은 네트워크를 통해 **전송되는 패킷의 수를 줄여** TCP/IP **네트워크의 효율성을 향상**시키는 방법이다. 이 알고리즘은 존 네이글(John Nagle)이 정의한 것으로, 네이글이 Ford Aerospace에서 일하던 시기인 1984년에 [RFC 896](https://www.rfc-editor.org/rfc/rfc896)에 **TCP/IP 인터네트워크 혼잡 제어**라는 이름으로 소개되었다.

- 인터네트워크(internetwork) : 통신 프로토콜이 다르거나 같은 복수의 통신망을 상호 접속하여 형성한 통신망의 집합 또는 광역 통신망
- RFC (Request for Comments)
    - 인터넷의 주요 기술 개발, 표준 등을 기술한 메모
    - IETF (Internet Engineering Task Force, 국제 인터넷 표준화 기구)에서 문서에 번호를 매겨 관리한다.

# 작동 방식

이 알고리즘은 여러 개의 **작은 발신 메시지를 결합**하여 한 번에 보내는 방식으로 작동한다.

1. 발신자는 데이터의 **첫 번째 패킷**을 수신자에게 보낸다.
2. 발신자는 수신자의 **acknowledgement(ACK, 확인 응답)**를 기다린다.
3. 발신자가 첫 번째 패킷에 대한 **ACK**를 받으면 **전송 대기 중인 데이터**가 있는지 확인한다.
    - 있는 경우 : 발신자는 **데이터를 버퍼링**하고 더 많은 데이터가 도착할 때까지 기다렸다가 버퍼링된 데이터와 새 데이터를 모두 포함하는 **더 큰 패킷을 전송**한다.
    - 없는 경우 : 발신자는 버퍼링된 데이터가 포함된 **다음 패킷**을 보낸다.
4. 모든 데이터가 전송될 때까지 위의 과정들을 반복한다.

# 장단점

## 장점

네이글 알고리즘은 네트워크 레이턴시가 길고 패킷이 작아서 데이터 전송이 크게 지연될 수 있는 상황에서 특히 유용하다. 크기가 작은 패킷의 개수를 줄임으로서 네이글 알고리즘은 **네트워크 대역폭을 효율**적으로 사용하며 웹 서버의 전체적인 성능을 향상시킨다.

## 단점

네이글 알고리즘은 발신자쪽 종단에서 작은 패킷들을 하나의 큰 패킷이 되도록 버퍼링한다. 이로 인해 **추가적인 딜레이**가 발생할 수 있다. 따라서 온라인 게임이나 화상 회의와 같이 매우 짧은 지연도 눈에 띄는 **실시간(real-time) 애플리케이션**에서는 잘 사용되지 않는다.

이러한 애플리케이션에서는 네이글 알고리즘의 지연을 피하기 위해 `setsockopt()`함수에 `TCP_NODELAY` 옵션을 사용하거나 **UDP**를 사용한다.

# 참고

- [Nagle's algorithm - Wikipedia](https://en.wikipedia.org/wiki/Nagle%27s_algorithm)
- [\[네트워크\] TCP Nagle 알고리즘](https://devjh.tistory.com/106)
- [Nagle's algorithm - Network Encyclopedia](https://networkencyclopedia.com/nagles-algorithm/)
- [TTA정보통신용어사전](http://terms.tta.or.kr/dictionary/dictionaryView.do?subject=internetwork)
- [Request for Comments - Wikipedia](https://en.wikipedia.org/wiki/Request_for_Comments)
- [TCP Networking: Understanding the Nagle Algorithm](https://www.lifewire.com/nagle-algorithm-for-tcp-network-communication-817932)
