---
title: "macOS 업데이트 이후 발생하는 xcrun error"
date: 2022-10-25
categories: [Project, etc_Project]
tags: [mac, issue]
---

# Issue

macOS Ventura로 업데이트를 한 뒤 git을 실행하니 아래와 같은 오류 메시지가 출력됐다.

```bash
$git
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

또한 터미널의 다른 프로그램들에서도 여러 오류들이 발생했다.

![Untitled](/assets/img/project/xcrun-error/Untitled.png)

xcrun error에 대해 찾아보니 Xcode의 Command-line Tools가 업데이트되지 않아서 발생한 것이었다.

# Solution

이를 해결하기 위해 터미널에 다음과 같이 입력한다.

```bash
xcode-select --install
```

입력을 하게 되면 Xcode의 Command-line Tools의 설치가 진행된다.

![Untitled](/assets/img/project/xcrun-error/Untitled%201.png)

설치를 마치면 다시 정상적으로 실행되는 것을 확인할 수 있다.

![Untitled](/assets/img/project/xcrun-error/Untitled%202.png)

# Ref.

<https://stackoverflow.com/questions/52522565/git-is-not-working-after-macos-update-xcrun-error-invalid-active-developer-pa>