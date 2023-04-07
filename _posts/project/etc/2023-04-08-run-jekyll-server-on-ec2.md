---
title: "[AWS EC2] Jekyll 서버 구동 방법"
date: 2023-04-08
categories: [Project, etc_Project]
tags: [aws, blog]
---

> 본 포스팅은 Ubuntu 22.04 LTS 를 기준으로 작성하였습니다.

# 개요

Jekyll 서버를 EC2에서 구동하기 위해서는 다음과 같은 과정을 거쳐야 한다.

1. 패키지 설치
2. 포트 포워딩
3. 서버 구동

# 전체 과정

## 패키지 설치

### ruby

```bash
sudo apt-get update &&\
sudo apt-get install -y ruby ruby-dev
```

### bundler

```bash
sudo gem install bundler
```

### Gemfile

- Gemfile이 존재하는 경로(보통 블로그의 루트 경로에 존재)에서 실행한다.

```bash
bundle install
```

## 포트 포워딩

로컬에서 브라우저로 서버에 접속하기 위해서는 `~.ssh/config` 파일에 `LocalForward` 지시어로 포워딩할 포트를 명시해주어야 한다.

- config 파일
    
    ```
    Host your_hostname
    		HostName <your_public_ip_address>
    		User <your_username>
    		IdentityFile <your_pem_file_path>
        LocalForward your_local_port server_address
    ```
    
- 예시
    - 로컬에서 5050포트로 접속하려면 아래와 같이 작성하면 된다. (Jekyll 서버는 4000번 포트를 사용)
    
    ```
    LocalForward 5050 localhost:4000
    ```
    

## 서버 구동

로컬에서와 동일하게 아래 명령어를 사용하여 구동한다.

```bash
bundle exec jekyll s
```

# 참고

- [Getting Started \| Chirpy](https://chirpy.cotes.page/posts/getting-started/)
- [Developing on Remote Machines using SSH and Visual Studio Code](https://code.visualstudio.com/docs/remote/ssh)
- [Using Jekyll with Bundler \| Jekyll • Simple, blog-aware, static sites](https://jekyllrb.com/tutorials/using-jekyll-with-bundler/)
