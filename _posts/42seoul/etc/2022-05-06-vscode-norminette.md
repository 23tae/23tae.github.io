---
title:  "VSCode에서 Norminette 확장 오류"
categories: [42Seoul, etc]
tags: [42seoul]
 
date: 2022-05-06
---

![42norminette](/assets/img/42seoul/etc/42norminette.png)

본 게시글은 linux, macOS를 기준으로 작성하였음.


# Issue

norminette을 Global에서 설치한 상태에서 vscode extenstion 중 **42 Norminette Highlighter (3.x)** 를 설치하게 되면 norminette을 찾지 못했다는 문구가 뜨면서 확장이 적용되지 않는다.

# Solution

클러스터에서 정상 작동하던 확장이 개인 pc에서는 제대로 동작하지 않는 이유는 두 환경에서의 norminette 패키지의 설치 경로가 다르고 path가 설정되어 있지 않기 때문인 것으로 보인다.

## 1. alias 삭제

기존에 `~/.zshrc`에 `alias norminette="python3 -m norminette"` 와 같은 명령어 별칭을 지정해 두었다면 우선 별칭을 삭제한다.

## 2. PATH 추가

1. `/etc` 로 이동
2. `sudo vim paths` 로 path 목록 수정
3. 목록의 제일 아래에 본인의 홈디렉토리 이름에 맞게 `/Users/$HOME/Library/Python/3.8/bin` 을 추가
(이때의 `3.8`로 된 Python 버전은 사용자에 따라 다를 수 있으니 사전에 해당 경로에 norminette이 존재하는지 확인)


## 3. vscode 재시작 후 확장 설치

# Ref.
https://jinnynote.com/learn/how-to-add-paths-on-mac/