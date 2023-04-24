---
title: "HTTP - 03.1 요청 메소드"
date: 2023-03-27
categories: [Computer Science, Computer Network]
tags: [cs, http, til]
---

# 메소드의 종류

- GET
- HEAD
- PUT
- POST
- TRACE
- OPTIONS
- DELETE

# 메소드별 의미

## GET

- 서버에서 **데이터를 가져오기** 위해 사용되는 메소드
- 요청을 보내면 서버가 응답 본문에 요청받은 데이터를 담아서 리턴한다.

## HEAD

- 개념
    - **GET** 메소드를 통해 특정 리소스를 요청했을 때 받을 **헤더를 요청**하는 메소드
    - 응답의 본문은 받지 않는다.
    - 리소스의 상태(마지막 수정 시간, content type 등)를 체크하기 위해 주로 사용된다.
- 예시
    ![head](/assets/img/cs/network/http-request-method/Untitled.png)

## PUT

- 서버 상에 **데이터를 생성**하거나 **업데이트**하는데 사용되는 메소드
- 새로운 데이터를 요청 본문에 담아 보내면 서버가 기존 리소스를 새로운 데이터로 업데이트한다.
    - 해당 리소스가 존재하지 않는다면 서버가 새롭게 생성한다.

## POST

- 개념
    - 리소스를 생성하거나 업데이트하기 위해 **서버에 데이터를 제출**하는 용도로 사용되는 메소드
    - 데이터는 요청 메시지의 본문에 담겨져 보내진다.
- 예시
    - 요청 메시지

        ```http
        POST /api/users HTTP/1.1
        Host: example.com
        Content-Type: application/json
        Content-Length: 54

        {"username": "taehooki", "password": "secretpassword"}
        ```

    - 응답 메시지

        ```http
        HTTP/1.1 201 Created
        Content-Type: application/json
        Location: /api/users/123
        Content-Length: 28

        {"message": "User created."}
        ```
        

## TRACE

- 개념
    - 요청과 응답이 클라이언트와 서버간에 교환되는 **과정을 진단**하기 위해 사용하는 메소드
    - 서버가 받은 **요청 헤더를 본문에** 담아 응답으로 보내는 방식으로 사용된다.
    - 쿠키나 기타 개인 정보가 노출될 수 있으므로 대부분의 웹 서버에서는 사용이 불가능하다.
- 예시
    - 요청 메시지
        
        ```http
        TRACE /index.html HTTP/1.1
        Host: example.com
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:90.0) Gecko/20100101 Firefox/90.0
        Accept: */*
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate, br
        Connection: keep-alive
        Referer: https://example.com/
        ```
        
    - 응답 메시지
        
        ```http
        HTTP/1.1 200 OK
        Content-Type: message/http
        Content-Length: 197
        Connection: keep-alive
        
        TRACE /index.html HTTP/1.1
        Host: example.com
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:90.0) Gecko/20100101 Firefox/90.0
        Accept: */*
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate, br
        Connection: keep-alive
        Referer: https://example.com/
        ```
        
    - 기타
        - 응답에서 요청 헤더를 반송하는 이유

## OPTIONS

- 개념
    - 특정 리소스에 대해 **사용가능한 HTTP 메소드**가 어떤 것이 있는지를 묻는 요청 메소드
- 예시
    - 다음 예시를 보면, `/example` 리소스에 대해 `GET`, `POST`, `OPTIONS` 메소드를 사용가능하다는 것을 알 수 있다.
    - 요청 메시지
        
        ```http
        OPTIONS /example HTTP/1.1
        Host: example.com
        Accept: */*
        ```
        
    - 응답 메시지
        
        ```http
        HTTP/1.1 200 OK
        Allow: GET, POST, OPTIONS
        Content-Length: 0
        ```
        

## DELETE

- 서버에서 리소스를 삭제하기 위해 사용되는 메소드

# 메소드별 처리 과정

## GET

1. Nginx가 configuration 파일상에서 요청 URL과 일치하는 URL이 포함된 location 블록이 존재하는지를 찾는다.
2. 일치하는 URL이 location 블록에 존재하는 경우
    1. 블록이 `proxy_pass` 지시문을 포함한 경우
        - 지시문에 지정된 백엔드 서버로 요청을 포워딩하고 응답 생성을 기다린다.
    2. 블록이 `root` 지시문을 포함한 경우
        1. 요청받은 파일을 `root` 지시문에 지정된 디렉토리에서 찾는다.
            1. 파일을 찾은 경우
                - 해당 내용을 응답 본문에 포함시킨다.
            2. 파일이 찾지 못찾은 경우
                - 클라이언트에게 `404 Not Found` 응답을 반환한다.
    3. 블록이 `fastcgi_pass` 지시어를 포함한 경우
        - Nginx는 지시문에 명시된 FastCGI 서버로 요청을 포워딩하고 FastCGI가 응답을 생성하기까지 기다린다.
    4. 블록이 `try_files` 지시문을 포함한 경우
        - 각각의 지정된 매개변수를 차례로 찾아보고 첫 번째로 찾은 것을 반환한다.
3. 일치하는 URL이 `location` 블록에 없는 경우
    - 클라이언트에게 404 에러 응답을 반환한다.
4. 응답 본문을 만든 뒤에 필요한 헤더를 추가하여 클라이언트에게 보낸다.
- 기타
    - `Content-Type`을 찾는 방법
        - 방법 1
            - 확장자를 보고 찾는다.
            - `mime.types`에 정의되어 있다.

        - 방법 2
            - Nginx configuration 파일 상의 `Content-Type`을 통해 찾는다.

## HEAD

- 전체적인 과정은 **GET**과 동일하다.
- 하지만 GET과 다르게 응답 메시지의 body는 보내지 않는다.

## PUT

1. configuration 파일에서 어떤 location 블록에 의해 요청이 처리될지를 확인한다.
2. 블록이 `try_files` 지시문을 포함한 경우
    - 요청받는 리소스가 서버에 존재하는지를 확인한다.
        - 이미 존재하는 경우 : 콘텐츠를 업데이트한다.
        - 존재하지 않는 경우 : 파일을 생성한다.
3. 요청의 본문에 있는 내용을 서버상의 경로(location 블록의 `root`나 `alias`에 주로 명시됨)에 저장한다.
4. 요청이 백엔드 서버나 애플리케이션에 의해 처리되어야 한다면 Nginx는 해당 요청을 업스트림 서버나 프록시로 전달한다. (이때 `proxy_pass` 지시어를 사용한다.)
5. 요청 처리에 성공한 경우
    - 업데이트한 경우
        - 해당 리소스 또는 교체된 부분이 담긴 응답을 반환한다. (`200 OK`)
        - 빈 응답을 반환한다. (`204 No Content`)
    - 파일을 생성한 경우
        - `201 Created` 반환한다. (생성된 파일의 경로가 담긴 Location 헤더 포함)
6. 요청 처리에 실패한 경우
    - 적절한 에러 응답을 반환한다. (ex. `400 Bad Request`, `500 Internal Server Error` 등)

## POST

1. Nginx가 HTTP 요청 헤더를 파싱하여 목적지 `URL`과 본문 `Content-Type`을 체크한다.
2. location 블록에서 일치하는 URL이 있는지 확인한다.
3. location 블록에 `client_max_body_size` 지시문이 설정되어있는 경우
    1. 요청 본문의 크기를 확인해서 크기가 초과되지 않는지 체크한다.
    2. 본문의 크기가 초과한다면 `413 Request Entity Too Large` 오류를 반환한다.
4. location 블록을 보고 적절한 업스트림 서버나 애플리케이션에 요청 본문을 넘겨준다.
    1. `proxy_pass` 지시문을 사용해 다른 서버로 요청을 포워딩하는 경우
    2. FastCGI나 uWSGI 서버를 사용하여 요청을 처리하는 경우
5. 해당 서버나 애플리케이션이 응답을 반환하는 경우
    - 해당 응답을 클라이언트에게 돌려준다.
6. 반환하지 않는 경우
    - 적합한 에러페이지를 생성해서 클라이언트에게 돌려준다.
7. 응답의 헤더를 수정하는데 `add_header`와 `expires` 지시문이 사용될 수 있다.
8. Nginx는 요청과 응답을 access log files에 기록한다.

## TRACE

1. Nginx가 configuration 서버 블록을 확인해서 어떤 location 블록으로 요청을 처리할지를 결정한다.
2. 요청 헤더와 본문을 클라이언트에게 반송(echo back)하도록 하여 요청을 처리한다.
3. 응답 상태 코드를 `200 OK`로 설정한다.
4. 응답의 `Content-Type` 헤더를 `message/http`로 설정한다.
5. Nginx는 응답 본문에 **요청 메시지의 모든 헤더**를 포함시킨다.
6. 설정된 응답을 클라이언트에게로 리턴한다.

## OPTIONS

1. 요청받은 URI가 유효하고 서버 블록 상의 일치하는 것이 존재하는지를 확인한다.
2. 존재하는 서버 블록을 찾았다면 해당 블록에 options 지시문이 있는지를 확인한다.
    1. 지시문이 없는 경우 : `405 Method Not Allowed` 응답을 반환한다.
    2. 지시문이 있는 경우 : options 지시문이 allowed_method 매개변수를 포함하는지를 확인한다.
        1. 포함하지 않는 경우 : 404 Not Found 에러를 반환한다.
        2. 포함한 경우 : **Allow 응답 헤더**에 해당 리소스에 대해 **허용된 HTTP 메소드**들을 포함하여 반환한다.
            - options 지시문에 명시된 추가적인 헤더들도 포함한다.(`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods` 등)
3. 응답을 클라이언트에게 보낸다.

## DELETE

1. Nginx가 configuration의 서버 블록을 확인해서 어떤 location 블록에서 요청을 처리해야할지를 결정한다.
2. 해당 유저가 파일을 삭제할 권한을 가졌는지를 확인한다.
    1. 권한이 없다면 : `403 Forbidden` 오류를 반환한다.
    2. 권한이 있다면
        - 해당 파일이 존재하는지를 확인한다.
            1. 파일이 존재하지 않는다면 : `404 Not Found` 오류를 반환한다.
            2. 파일이 존재한다면 : 서버의 파일 시스템에서 해당 파일을 삭제한다.
3. 요청을 성공적으로 처리한 경우 : `204 No Content` 응답을 반환한다.

# 참고
- [Nginx: handle location depending on HTTP method - Server Fault](https://serverfault.com/questions/754249/nginx-handle-location-depending-on-http-method)
- [How can I allow HTTP Methods on Nginx - Stack Overflow](https://stackoverflow.com/questions/59964033/how-can-i-allow-http-methods-on-nginx)
- [HTTP Methods GET vs POST](https://www.w3schools.com/tags/ref_httpmethods.asp)
