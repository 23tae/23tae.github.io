---
title: "WSL2 시작 경로 변경하기"
date: 2022-03-14T13:03:28.099Z
categories: [Project]
tags: [wsl2, issue]
---
# Issue
윈도우 터미널에서 wsl2를 실행하게 되면 기본 시작 경로가 `C:\Users\<YourUsername>`으로 되어있다.
이것을 리눅스의 홈 디렉터리로 변경하려고 한다.

# Solution
윈도우 터미널을 실행한 후, 상단의 **설정**(또는 `Ctrl + ,`)-좌측 하단의 **Json 파일 열기**를 클릭하여 에디터에서 자신의 WSL 프로필을 찾는다.

```json
{
    "guid": "{c6eaf9f4-32a7-5fdc-b5cf-066e8a4b1e40}",
    "hidden": false,
    "name": "WSL",
    "source": "Windows.Terminal.Wsl",
}
```
source키-값 쌍 아래에 시작경로를 아래와 같이 추가한다.
`startingDirectory": "/home/<directory name>"`


# Ref.
<https://docs.microsoft.com/ko-kr/windows/terminal/troubleshooting#set-your-wsl-distribution-to-start-in-the-home--directory-when-launched>  
<https://jakupsil.tistory.com/45>