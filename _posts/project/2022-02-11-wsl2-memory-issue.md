---
title: "WSL2 종료 후 Vmmem이 Windows의 메모리를 차지하는 이슈"
date: 2022-02-11T10:59:49.941Z
categories: [Project]
tags: [wsl2, issue]
---
# Issue
WSL2를 통해 우분투 환경에서 작업을 하고 난 뒤 프로그램을 종료했는데 윈도우 상에서 작업 관리자를 실행했을 때 다음과 같이 Vmmem이 상당한 메모리를 차지하는 경우가 발생했다.
![config](/assets/img/project/wslconfig.jpeg)
해당 프로그램을 종료하려고 시도해도 아래와 같이 에러창이 뜨면서 종료가 되지 않는다.
![manager](/assets/img/project/task-manager.jpeg)
해당 이슈를 구글링 해보니 아직까지 해결되지 않은 문제인 듯 하다. 임시적인 해결방법은 윈도우 상에 wsl2가 차지하는 메모리의 양을 할당하는 것과 WSL2를 종료시키는 것 두 가지가 있는 것으로 보인다.
# Solution
**1. `.wslconfig` 파일 생성 (최초 1회만)**
undefined
윈도우 상에서 CMD를 실행한 후, `C:\Users\Username` 경로에 `copy con .wslconfig`와 같이 입력하여 `.wslconfig`라는 이름의 파일을 생성한다.
```shell
[wsl2]

memory=3GB

swap=0
```
파일 내부에 위와 같은 내용(memory의 값은 자신의 메모리 상황에 맞게 입력하면 됨)을 입력한 후 Ctrl+C를 눌러 저장한다.

<br>

**2. wsl 2 종료**
`wsl -l --running`를 입력해서 아래와 같이 현재 자신이 사용중인 Ubuntu의 배포판을 확인한다.

```shell
C:\Users\Username\>wsl -l --running
Linux용 Windows 하위 시스템 배포:
Ubuntu-18.04(기본값)
```
bash가 종료된 상태에서 자신의 우분투 버전을 다음과 같은 형태로 입력하여 wsl2를 종료한다.
`wsl -t Ubuntu-18.04`
undefined
그럼 위와 같이 문제가 해결된 것을 확인할 수 있다.

# Reference
<https://meaownworld.tistory.com/160>  
<https://github.com/microsoft/WSL/issues/4166>