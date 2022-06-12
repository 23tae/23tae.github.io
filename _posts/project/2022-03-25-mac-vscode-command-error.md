---
title: "mac에서 vscode 실행 안됨 (code 명령어)"
date: 2022-03-25T07:32:21.250Z
categories: [Project]
tags: [issue, mac, vscode]
---
>macOS에서 `code .` 명령어를 입력했을 때 `not found`가 출력되는 문제에 관한 해결방법을 담고있음.

Ubuntu에서의 해결방법은 [다음](https://velog.io/@23tae/Ubuntu에서-vscode-실행-안됨-code-명령어)을 참고.

## ① .zshrc 파일 수정

홈 디렉터리에서 `.zshrc`파일을 수정하여 하단에 다음과 같은 라인을 추가한다.

```shell
code () { VSCODE_CWD="$PWD" open -n -b "com.microsoft.VSCode" --args $* ;}
```

## ② 수정된 파일 적용

다음 두가지 방법 중 한가지로 수정된 내용을 적용한다.

### option 1
shell 재시작

### option 2
`$ source ~/.zshrc` 입력