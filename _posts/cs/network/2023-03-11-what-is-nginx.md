---
title: "Nginx란?"
date: 2023-03-11
categories: [Computer Science, Computer Network]
tags: [cs, til, network]
---

![Untitled](/assets/img/cs/network/what-is-nginx/nginx-logo.svg)

# 개요

Nginx는 리버스 프록시, 로드 밸런서, HTTP 캐시로도 작동하는 인기 있는 오픈 소스 **웹 서버** 소프트웨어이다. 2002년 이고르 시소예프(Igor Sysoev)가 만들었으며 2004년에 처음 출시되었다.
높은 성능, 안정성, 낮은 리소스 사용으로 유명하며 넷플릭스, 드롭박스, 에어비앤비, 깃허브, Wordpress 등의 웹사이트와 웹 애플리케이션에서 사용중이다.

# Nginx를 왜 사용할까?

- 높은 성능
    - nginx는 **다수의 동시 연결 및 요청**을 효율적으로 처리하도록 설계되었다. 이를 통해 초당 수천 개의 요청을 처리할 수 있어 트래픽이 많은 웹 사이트에 자주 쓰인다.
- 리버스 프록시 서버 (reverse proxy server)
    - nginx는 클라이언트로부터 요청을 수신하여 다른 서버로 전달할 수 있다. 이를 통해 여러 서버의 부하를 분산시키고 시스템의 전반적인 성능을 향상시킬 수 있다.
- 로드 밸런싱 (load balancing)
    - nginx는 라운드 로빈, IP 해시, 최소 연결 등 다양한 로드 밸런싱 알고리즘을 지원합니다. 여러 서버 간에 **요청을 분산**시켜 단일 서버의 과부하를 방지하고 고가용성을 보장한다.
        - 고가용성 (high availability) : 서버와 네트워크, 프로그램 등의 정보 시스템이 상당히 오랜 기간 동안 지속적으로 정상 운영이 가능한 성질
- 컨텐츠 캐싱 (content caching)
    - nginx는 자주 액세스하는 컨텐츠를 메모리에 캐싱할 수 있어 백엔드 서버의 부하를 줄이고 클라이언트의 응답 시간을 향상시킬 수 있다.
        - 캐싱 (caching) : 오랜시간이 걸리는 작업의 결과를 저장해서 시간과 비용을 필요로 회피하는 기법
- 보안
    - nginx에는 SSL/TLS 암호화, IP 화이트리스트/블랙리스트, DDoS 공격에 대한 보호 등의 보안 기능이 내장되어 있다.
- 간편한 설정(configuration)
    - nginx는 이해하기 쉬운 간단한 configuration 문법을 가진다. 또한 동적 설정 업데이트를 지원해서 서버를 재시작하지 않고도 설정을 변경할 수 있다.

# 요청 처리 방식

Nginx는 모듈 시스템을 사용하여 각 요청을 처리한다. 요청은 일련의 단계를 거치며, 각 모듈은 특정 단계에서 요청을 처리한다.

1. 클라이언트가 서버에게 보낸 **요청(request)**은 운영체제의 **네트워크 스택**에 도달한 뒤에 **Nginx로 전달**된다.
2. 우선 Nginx는 **요청 헤더를 읽고 파싱(parsing)**한다. 그 뒤에 어떤 **서버 블록**과 **위치 블록**이 요청을 처리할지를 결정한다.
    - 서버 블록 (server block)
        - 특정 가상 서버에 대한 설정을 정의한 configuration 블록
        - 각 블록이 특정 도메인명이나 IP 주소에 대한 설정을 담고있다.
    - 위치 블록 (location block)
        - **특정 위치로의 요청**이 어떻게 처리될 지에 대해 정의한 configuration 블록.
        - **서버 블록 내부**에 위치한다.
        - 정규 표현식으로 작성된다.
    - 예시
        
        ```
        server {
            listen 80;
            server_name example.com;
            
            location / {
                # configuration for handling requests to the root directory
            }
        }
        ```
        
3. Nginx는 요청된 리소스가 **캐시**에 있는지 확인한다.
    - 리소스가 캐시에 없는 경우 :  요청을 **업스트림 서버** 또는 **백엔드 어플리케이션 서버**로 보낸다.
4. 업스트림 서버가 사용 불가능한 경우
    - Nginx는 정적 파일을 제공하거나 클라이언트에 오류를 반환한다.
5. 업스트림 서버를 사용 가능한 경우
    - Nginx는 요청을 **백엔드 서버로 전달**하고 응답을 기다린다.
6. 백엔드 서버로부터 **응답(response)을 수신**하면 Nginx는 응답 헤더를 처리한 뒤에 **응답 본문(body)**를 클라이언트에게 다시 전송한다.
7. 설정에 따라 로드 밸런싱, SSL 종료, 캐싱과 같은 다른 기능도 수행할 수 있다.
    - SSL 종료 : SSL로 암호화된 데이터 트래픽이 해독(또는 오프로드)되는 프로세스
8. 마지막으로 Nginx는 클라이언트 IP 주소, 요청 방법 및 응답 상태 코드와 같은 **요청 세부 정보**를 **액세스 로그**에 기록합니다.

# 참고

- contents
    - [What is NGINX? - NGINX](https://www.nginx.com/resources/glossary/nginx/)
    - [How nginx processes a request](http://nginx.org/en/docs/http/request_processing.html)
    - [고가용성 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EA%B3%A0%EA%B0%80%EC%9A%A9%EC%84%B1)
    - [Caching - 생활코딩](https://www.opentutorials.org/course/697/3839)
    - [SSL Termination이란 무엇인가? :: 작은 거인의 블로그](https://sepiros.tistory.com/50)
- images
    - [Nginx - 위키백과, 우리 모두의 백과사전](https://ko.m.wikipedia.org/wiki/Nginx)
