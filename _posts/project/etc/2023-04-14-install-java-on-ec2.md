---
title: "[AWS EC2] Java 설치 방법"
date: 2023-04-14
categories: [Project, etc_Project]
tags: [til, ec2, java]
---

> 본 게시글은 Ubuntu 22.04 LTS를 기준으로 작성하였습니다.

# 개요

Java를 EC2 인스턴스에 설치하는 방법은 패키지 매니저 사용, 직접 다운로드하여 설치 등 다양하다. 이 글에서는 그 중 직접 파일을 다운로드하여 설치하는 방법에 대해 다루었다.

# 설치 과정

## 설치 파일 확인

Oracle 사이트의 [Java 설치페이지](https://www.oracle.com/java/technologies/downloads/#java17)로 이동하여 자신의 시스템에 맞는 Java17 설치 링크를 복사한다.

다음 명령어를 통해 프로세스 아키텍처를 찾은 후, 설치할 파일을 찾는다.

```bash
uname -p
```

위 명령어를 입력했을 때 `arm64` 이 나온다면 ARM64를, `x86_64` 라면 x64를 설치한다.

내가 사용하는 인스턴스의 경우 OS는 Ubuntu, 아키텍처는 `x86_64`를 사용하였기에 `[Java17]-[Linux]-[x64 Compressed Archive]` 링크를 복사하였다. 아래 내용은 해당 파일로 설치하는 방법이다.

## 설치 경로 생성

Java를 설치하고 싶은 경로를 생성한 뒤, 해당 경로로 이동한다.

```bash
sudo mkdir <path_to_install>
cd <path_to_install>
```

- 예시
    
    ```bash
    sudo mkdir /usr/java && \
    cd /usr/java
    ```
    

## 파일 다운로드

기존에 복사해둔 링크를 생성한 경로에 다운로드한다.

```bash
wget <file_link>
```

## Java 설치

링크로 다운로드 받은 파일의 압축을 해제한다.

```bash
tar -zxvf <filename.tar.gz> <your-directory-name>
```

- 예시
    
    ```bash
    tar -zxvf jdk-17_linux-x64_bin.tar.gz jdk-17.0.6
    ```
    

압축을 해제하고 난 뒤, 사용을 마친 압축 파일은 삭제한다.

## PATH 설정

`java` 명령어로 Java의 실행파일을 실행하도록 설정하는 과정이다.

### JAVA_HOME 변수 설정

PATH를 설정을 위한 변수를 **default shell**의 startup 스크립트에 추가한다.

- default shell
    
    ```bash
    echo $SHELL
    ```
    

default shell로 **bash**를 사용하는 경우 `~/.bashrc`, **zsh**를 사용한다면 `~/.zshrc` 하단에 다음 내용을 추가한다. JAVA_HOME 변수는 이전에 압축 해제한 디렉토리의 위치로 설정한다.

```bash
export JAVA_HOME=/usr/java/jdk-17.0.6
export PATH=$JAVA_HOME/bin:$PATH
```

- export 명령어는 현재 shell 세션에만 적용되기 때문에 위와 같이 `.bashrc` 또는 `.zshrc`에 명령을 추가해주어야 한다.
    - 이렇게 작성하면 shell을 새롭게 시작할 때마다 export가 적용된다.

## 설치 확인

다음 명령어를 입력하여 Java가 정상적으로 설치되었는지를 확인한다.

```bash
java -version
```

정상적으로 설치가 되었다면 다음과 같은 메시지가 출력된다.

```
java version "17.0.6" 2023-01-17 LTS
Java(TM) SE Runtime Environment (build 17.0.6+9-LTS-190)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.6+9-LTS-190, mixed mode, sharing)
```

# 참고

- [Java Downloads \| Oracle](https://www.oracle.com/java/technologies/downloads/#java17)
- [Installation of the JDK on Linux Platforms](https://docs.oracle.com/en/java/javase/17/install/installation-jdk-linux-platforms.html)
