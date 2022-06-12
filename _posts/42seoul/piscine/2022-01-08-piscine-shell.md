---
title: "Shell"
date: 2022-01-08
categories: [42Seoul, Piscine]
tags: [shell]
---
> 라피신 첫번째 과제인 Shell을 통해 배운 내용

## Shell
```shell
man
#명령어 별로 매뉴얼을 볼 수 있음

ctrl + a/e/u
#명령줄 앞/뒤 이동,삭제

echo [content] > [filename]
#파일을 생성해서 내용을 저장함

cat > [filename]
#파일 만들어서 바로 입력
#이후 종료는 ctrl+d 사용

mkdir ex{00..09}
#디렉토리 한번에 생성

cp -rf [original dir name]/* [new dir name]
#새 디렉토리에 복붙
#original 디렉토리의 하위 모든 디렉토리가 new 디렉토리 안에 복사됨.

cat */*
#디렉토리 내의 모든 파일 열기

tar -xf [filename]
#압축 해제

#옵션에 v를 추가하면 결과를 출력함

tar -cf [압축파일명] [내부파일명]
#파일 압축

#특수문자 파일명: 각 특수문자 앞에 \ 붙여야 함

chmod [per num] [filename]
#권한(permission) 변경

rm -rf [filename]
#강제 삭제 명령어(디렉토리 삭제에 이용)

ln [filename1] [filename2]
#하드링크 생성
ln -s [TARGET] [LINK_NAME]
#심볼릭 링크 생성

#!/bin/sh
#스크립트 첫줄에 붙이는 이유는 해당 경로에서 bash를 실행시킨다는 의미

kinit
#kerberos 티켓 발급
klist
#kerberos 티켓 현황 표시

file -m ft_magic 42f
#매직파일 확인
```
## vim

```shell
:se nu
:set number
#줄번호
```

