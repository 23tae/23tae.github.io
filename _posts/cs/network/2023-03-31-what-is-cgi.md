---
title: "CGI란?"
date: 2023-03-31
categories: [Computer Science, Computer Network]
tags: [cs, til, network]
---

# 개요

![Untitled](/assets/img/cs/network/what-is-cgi/Untitled.png)

CGI는 **Common Gateway Interface**의 약자로, Nginx와 같은 웹 서버가 **프로그램을 동적으로 실행**하고 **동적 콘텐츠를 생성**할 수 있도록 하는 표준 프로토콜이다. 주로 사용자의 **요청을 처리**하기 위해 사용된다.

- 실행되는 프로그램은 주로 **스크립트 언어**로 작성된다.
    - 스크립트 언어(scripting language)
        - 기존 시스템의 **기능을 조작**하거나 **사용자 정의** 및 **자동화**하는데 사용되는 프로그래밍 언어의 일종
        - 주로 컴파일 타임이 아닌 **런타임**에 해석된다.
    - 예시. CGI 스크립트
        
        ```python
        #!/usr/bin/env python
        
        import cgi
        
        # 응답의 콘텐츠 유형을 HTML로 설정한다
        print("Content-type: text/html\n")
        
        # 사용자의 입력 데이터를 STDIN에서 가져온다
        form = cgi.FieldStorage()
        
        # 사용자의 name을 가져온다. name이 없다면 기본값을 사용한다
        name = form.getvalue('name', 'taehoon')
        
        # HTML 응답을 생성한다
        print("<html>")
        print("<head><title>Greetings</title></head>")
        print("<body>")
        print("<h1>Hello, {}!</h1>".format(name))
        print("</body>")
        print("</html>")
        ```
        

# 동작 과정

1. 클라이언트가 **동적 콘텐츠**가 필요한 웹 페이지를 요청한다.
    - 예시. 페이지에 양식이 존재하는 경우, 데이터베이스에서 정보를 보여주는 경우
2. **웹 서버(Nginx, Apache 등)**는 요청을 받은 뒤 해당 요청이 동적 콘텐츠 실행을 필요로 한다는 것을 인식한다.
3. 서버가 프로그램(또는 CGI 스크립트)의 실행을 위해 **새로운 프로세스를 생성**한다.
4. 생성된 프로세스의 프로그램이 사용자가 보낸 요청을 **STDIN(표준 입력 스트림)**에서 받아와 데이터를 읽는다.
    - 예시. 사용자가 POST 요청으로 웹 양식을 보낸 경우, STDIN을 통해 해당 양식을 받는다.
5. 해당 프로그램은 데이터를 처리하여 웹 페이지에 필요한 **동적 콘텐츠를 생성**한다.
6. 프로그램이 동적 콘텐츠를 **STDOUT(표준 출력 스트림)**을 통해 웹 서버에게 되돌려준다.
7. 웹 서버는 동적 콘텐츠를 받아서 유저의 브라우저에 **웹 페이지**의 일부로 돌려준다.
8. CGI 프로세스가 **종료**된다.
9. **새로운 요청**이 발생하면 웹 서버는 **새 프로세스를 생성**하여 위의 과정을 반복한다.

# Nginx에서

## FastCGI

Nginx는 CGI를 직접적으로 지원하지 않는 대신 **FastCGI를 지원**한다. FastCGI의 프로세스들은 새로운 요청이 들어와도 계속 유지되기 때문에 새 프로세스를 매번 시작하는 CGI에 비해 오버헤드가 덜하다.

## 사용 방법

Nginx에서 CGI를 사용하기 위해서는 **요청을 FastCGI 서버로 넘겨주도록** 설정해야한다. 이러한 설정은  nginx configuration 파일의 `fastcgi_pass` 지시어를 통해 이루어진다. 또한 `fastcgi_param` 지시어를 통해 실행될 CGI 스크립트나 **프로그램의 경로**를 명시해야한다. 이러한 방식을 사용하면 Nginx가 프로그램이나 스크립트를 통해 동적 콘텐츠를 웹 페이지에 생성할 수 있게 된다.

- 예시
    
    ```
    location /cgi-bin/ {
        # CGI 스크립트의 경로
        fastcgi_pass 127.0.0.1:9000;
        # CGI 스크립트 파일의 타입과 경로
        fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html$fastcgi_script_name;
        # 다른 CGI 스크립트 파라미터도 넘겨준다
        include fastcgi_params;
    }
    ```
    

# 참고

- [Common Gateway Interface - Wikipedia](https://en.wikipedia.org/wiki/Common_Gateway_Interface)
- [Scripting language - Wikipedia](https://en.wikipedia.org/wiki/Scripting_language)
- [What Are Scripting Languages? (And Why Should I Learn One) \| Coursera](https://www.coursera.org/articles/scripting-language)
- [CGI \<cgi\> \| Microsoft Learn](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/cgi)
- [FastCGI Example \| NGINX](https://www.nginx.com/resources/wiki/start/topics/examples/fastcgiexample/)
- [perl - How to run CGI scripts on Nginx - Stack Overflow](https://stackoverflow.com/questions/11667489/how-to-run-cgi-scripts-on-nginx)
- [Understanding and Implementing FastCGI Proxying in Nginx  \| DigitalOcean](https://www.digitalocean.com/community/tutorials/understanding-and-implementing-fastcgi-proxying-in-nginx)
- [3. Output from the Common Gateway Interface - CGI Programming on the World Wide Web [Book]](https://www.oreilly.com/library/view/cgi-programming-on/9781565921689/06_chapter-03.html)
