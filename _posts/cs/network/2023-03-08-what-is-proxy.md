---
title: "프록시 (Proxy)란?"
date: 2023-03-08
categories: [Computer Science, Computer Network]
tags: [cs, til, network]
---

# 프록시 (Proxy)

![Untitled](/assets/img/cs/network/what-is-proxy/Untitled.png)

## 개념

- 클라이언트와 서버 사이에서 **대리로 통신을 수행**하는 것
    - 이러한 중개 기능을 하는 서버를 **프록시 서버(proxy server)**라고 한다.
- 주로 **보안**을 강화하고 **성능**을 향상시키거나 **익명성**을 제공하기 위해 사용된다.

## 주요 기능

- 캐싱 (caching)
- 필터링 (filering)
    - 예시) 바이러스 백신 스캔, 유해 컨텐츠 차단(자녀 보호) 기능 등
- 로드 밸런싱 (load balancing)
    - 여러 서버들이 서로 다른 요청을 처리하게 해준다.
- 인증 (authentication)
    - 다양한 리소스에 대한 접근을 제어한다.
- 로깅 (logging)
    - 이력 정보를 저장한다.

## 작동 과정

1. 유저가 자원을 **요청**한다.
2. 요청이 프록시 서버로 전달된다.
3. 프록시가 원본 서버에서 **자원을 가져다가 유저에게 리턴**한다.

## 사용 예시

- HTTP 프록시
- SSL 프록시
- FTP 프록시
- SOCKS 프록시 등

## 종류

작동 방향에 따라 포워드 프록시, 리버스 프록시로 나뉜다.

### 포워드 프록시 (forward proxy)

![Untitled](/assets/img/cs/network/what-is-proxy/Untitled%201.png)

- 개념
    - **클라이언트**와 **인터넷** 사이에 있는 프록시 서버
    - **클라이언트의 신원을 숨기고** 익명성을 제공하기 위해 사용된다.
- 작동 과정
    1. 클라이언트가 인터넷에서 리소스를 요청한다.
    2. 요청이 전달 프록시 서버로 전송된다.
    3. 프록시 서버는 **클라이언트를 대신하여 요청을 수행**하고 클라이언트에 응답을 반환한다.
    4. 이 과정에서 서버는 **클라이언트의 요청**을 인지하지만 **인터넷은 인지하지 못한다.**

### 리버스 프록시 (reverse proxy)

![Untitled](/assets/img/cs/network/what-is-proxy/Untitled%202.png)

- 개념
    - **인터넷**과 **서버** 사이에 있는 프록시 서버
    - 수신 트래픽을 여러 서버 간에 분산하고 로드 밸런싱을 제공한다.
    - 서버의 신원을 감춰 추가적인 보안 계층을 제공한다.
- 작동 과정
    1. 클라이언트가 서버에 요청할 때, 요청은 먼저 리버스 프록시 서버로 전송된다.
    2. 프록시 서버는 요청을 처리하는 데 가장 적합한 서버를 결정하고 해당 서버로 요청을 전달한다.
    3. 서버는 **프록시 서버**를 인지하지만 **클라이언트는 인지하지 못한다.**
- 기타
    - 서버가 여러 웹 사이트나 서비스를 **호스팅**할 때 유용하다.
        - 요청된 도메인 또는 URL을 기반으로 **적절한 서버로 트래픽을 라우팅**할 수 있기 때문

# 캐시, 데이터베이스와의 차이

## 주요 개념

### 캐시 (cache)

![cache.png](/assets/img/cs/network/what-is-proxy/cache.png)

- 개념
    - **자주 접근되는 데이터**나 자원을 **임시 저장**하여 시스템의 성능을 향상시키는 **저장소** 메커니즘
- 작동 과정
    1. 유저가 자원을 **요청**한다.
    2. 시스템은 자원이 캐시에 존재하는지를 확인한다.
    3. 존재한다면, **캐시에 있는 자원을 제공**한다.

### 데이터베이스 (database)

- 개념
    - **데이터를 저장**, 구조화, 관리하기위해 디자인된 소프트웨어 시스템
    - 데이터를 구조화된 형식으로 저장한다.
        - 이후에 어플리케이션을 통해 해당 데이터에 접근하여 조작한다.
- 주요 기능
    - 효율적인 데이터 저장, 검색 및 조작 기능
    - 트랜잭션, 인덱싱, 백업 및 복구 기능
- 사용
    - 웹 어플리케이션에서 널리 쓰인다.
    - 주로 유저 데이터, 상품 정보나 그 밖에 중요 정보를 저장하는데 사용된다.

## 주요 차이점

### 프록시, 캐시 vs 데이터베이스

- 프록시와 캐시는 웹 애플리케이션의 **성능과 확장성을 향상**시키는 데 사용된다.
- 반면 데이터베이스는 **데이터를 저장, 구성 및 관리**하는 데 사용된다.

### 프록시 vs 캐시

- 프록시는 **클라이언트와 서버 사이**에 존재한다.
- 캐시는 컴퓨터 시스템 또는 **기기 내부**에 존재한다.

# Ref.

- [https://developer.mozilla.org/ko/docs/Web/HTTP/Overview](https://developer.mozilla.org/ko/docs/Web/HTTP/Overview)
- [https://www.upguard.com/blog/what-is-proxy](https://www.upguard.com/blog/what-is-proxy)
- [https://docs.oracle.com/cd/E19575-01/821-0053/adymk/index.html](https://docs.oracle.com/cd/E19575-01/821-0053/adymk/index.html)
- [https://witscad.com/course/computer-architecture/chapter/cache-memory](https://witscad.com/course/computer-architecture/chapter/cache-memory)