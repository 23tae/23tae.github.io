---
title:  "norminette 개인pc 설치방법"
categories: [42Seoul, etc]
tags: [42seoul]
 
date: 2022-05-05
---

> norminette을 클러스터 외부에서 설치하는 방법을 다룸

본 게시글은 linux, macOS를 기준으로 작성하였음.

# 설치 과정

### 옵션 1. Global에서 설치
1. setuptools 업그레이드
```bash
python3 -m pip install --upgrade pip setuptools
```
2. norminette 설치
```bash
python3 -m pip install norminette
```

3. 명령어 별칭 추가 및 적용
```bash
echo 'alias norminette="python3 -m norminette"' >> ~/.zshrc
```
```bash
source ~/.zshrc
```

- 추후 버전 업그레이드는 아래의 명령을 통해 진행
```bash
python3 -m pip install --upgrade norminette
```


### 옵션 2. 가상환경에서 설치
1. 가상환경 생성
```bash
python3 -m venv venv
```
2. 가상환경 활성화
```bash
source venv/bin/activate
```
3. norminette 설치
```bash
pip install norminette
```

- norminette 사용이 끝나면 `deactivate` 명령어로 가상환경 비활성화


# Ref.
[42School/norminette](https://github.com/42School/norminette)  
[42Seoul Norminette 설치하기](https://velog.io/@pearpearb/42Seoul-Norminette-설치하기)