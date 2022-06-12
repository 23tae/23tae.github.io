---
title: "WSL2에서 vscode 실행 안됨 (code 명령어)"
date: 2022-02-05T14:51:03.704Z
categories: [Project]
tags: [wsl2, issue, vscode]
---
>WSL2에서 Ubuntu를 실행하여 `code .` 명령어를 입력했을 때 `not found`가 출력되는 문제에 관한 해결방법을 담고있음.

#  Issue
![code1](/assets/img/project/wsl-code1.jpeg)
평소처럼 terminal을 실행시키고 홈 디렉토리에서 `code .` 명령어를 쳤는데 `/mnt/c/Users/~/bin/code: not found` 라는 문구가 출력됐다. 윈도우에서 직접 vscode를 실행시키니 프로그램의 문제는 아니었다.
한참을 이 문제로 고생하다 구글링을 해보니 환경변수 설정이 해제되어 생긴 문제로 보였다.



# Solution
![code2](/assets/img/project/wsl-code2.jpeg)
해결방법은 의외로 간단하다. 홈디렉토리에서 `.profile`을 편집기로 실행시킨 후 맨 아래에 `PATH="$/mnt/c/Users/(유저이름)/AppData/Local/Programs/Microsoft\ VS\ Code/bin"` 다음과 같은 환경변수를 추가하면 된다.



# Reference
<https://jmcunst.tistory.com/152>
<http://blog.foundy.io/visual-studio-code-maeg-teomineoleseo-code-myeongryeongeo-path-seoljeonghagi/>