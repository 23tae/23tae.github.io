---
title:  "42header 개인pc 설치방법"
categories: [42Seoul, etc_42]
tags: [42seoul]
 
date: 2022-03-16
---

> 42헤더를 클러스터 밖에서 적용할 수 있는 방법을 다룸

본 게시글은 linux, macOS를 기준으로 작성하였음.

# 1. 42header plugin 설치
클러스터 밖 개인 pc를 사용중이라면 pc에 우선 42헤더 플러그인을 설치해야한다. 아래와 같이 본인의 홈 디렉터리 임의의 디렉터리에서 git clone을 받는다.

```shell
git clone https://github.com/42Paris/42header.git
```
clone 받은 디렉터리에서 `stdheader.vim`(plugin에 위치)을 찾아`~/.vim/plugin`으로 복사한다. (해당 경로가 없으면 새로 만든다.)

# 2. configuration file 설정
아래 두가지 옵션 중 한가지로만 하면 된다.

## 옵션 1. zshrc
`~/.zshrc` 하단에 다음과 같이 추가한다. (intraID 부분만 본인의 ID로 변경)  
(파일이 없다면 새로 만든다.)

```shell
export USER=intraID
export MAIL=$USER@student.42seoul.kr
```

## 옵션 2. vimrc
`~/.vimrc` 하단에 다음과 같이 추가한다. (intraID 부분만 본인의 ID로 변경)  
(파일이 없다면 새로 만든다.)

```shell
let g:user42 = 'intraID'
let g:mail42 = 'intraID@student.42seoul.kr'
```

# 3. 테스트
터미널을 재시작한 후 테스트할 파일을 vim으로 열고 F1을 눌러서 42헤더가 제대로 입력되는지 확인한다.


# Ref.
[42Paris/42header](https://github.com/42Paris/42header)  
[[42Seoul] 윈도우10 wsl2에서 42 과제 편하게 하기](https://velog.io/@seomoon/42-윈도우10-wsl2에서-42서울-과제-편하게-하기norm-42헤더-git-config-등)