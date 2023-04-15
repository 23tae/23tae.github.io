---
title: "[AWS EC2] Jekyll 서버 구동 방법"
date: 2023-04-08
categories: [Project, etc_Project]
tags: [aws, ec2, blog]
---

> 본 게시글은 Ubuntu 22.04 LTS를 기준으로 작성하였습니다.

# 개요

Jekyll 서버를 EC2에서 구동하기 위해서는 다음과 같은 과정을 거쳐야 한다.

1. 포트 포워딩
2. 패키지 설치
3. Jekyll 서버 구동

# 전체 과정

## 포트 포워딩

로컬에서 브라우저로 서버에 접속하기 위해서는 로컬의 `~/.ssh/config` 파일의 `LocalForward` 지시어에 포워딩할 포트를 명시해주어야 한다.

- 설정 파일
    
    ```
    Host <your_hostname>
        HostName <your_public_ip_address>
        User <your_username>
        IdentityFile <your_pem_file_path>
        LocalForward <your_local_port> <server_address>
    ```
    
- 예시
    - 로컬에서 5050포트로 접속하려면 아래와 같이 작성한다. (Jekyll 서버는 4000번 포트를 사용)
    
    ```
    LocalForward 5050 localhost:4000
    ```

## 패키지 설치

EC2 인스턴스에 접속하여 다음과 같은 패키지를 설치해야 한다.

- Ruby
- Bundler
- Gems

### 패키지 리스트 업데이트

```bash
sudo apt-get update
```

### Ruby

```bash
sudo apt-get install -y ruby-full build-essential zlib1g-dev
```

### Bundler

```bash
gem install bundler
```

### Gems

- Gemfile이 존재하는 경로(보통 블로그의 루트 경로에 존재)에서 실행한다.
    - Gemfile
        - Ruby용 의존성 관리자인 Bundler가 사용하는 설정 파일
        - Ruby 애플리케이션 구동에 필요한 gems와 버전을 명시한다.

```bash
bundle install
```

- 위 명령어로 홈에 Gems가 설치되고 Gemfile.lock 파일이 생성된다.
    - Gemfile.lock : 기기에 설치된 gems의 버전을 기록한 파일


## Jekyll 서버 구동

위 과정을 모두 마쳤다면 EC2 터미널에서 아래 명령어를 사용하여 Jekyll 서버를 구동한다.

```bash
bundle exec jekyll serve
```

포트 포워딩을 하였기 때문에 로컬에서 `localhost:<위에서 설정한 포트>` 로 접속할 수 있다.

# 참고

- [Getting Started \| Chirpy](https://chirpy.cotes.page/posts/getting-started/)
- [Developing on Remote Machines using SSH and Visual Studio Code](https://code.visualstudio.com/docs/remote/ssh)
- [Jekyll on Ubuntu \| Jekyll • Simple, blog-aware, static sites](https://jekyllrb.com/docs/installation/ubuntu/)
- [What is the difference between Gemfile and Gemfile.lock in Ruby on Rails - Stack Overflow](https://stackoverflow.com/questions/6927442/what-is-the-difference-between-gemfile-and-gemfile-lock-in-ruby-on-rails)
