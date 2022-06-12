---
title: "[C언어] gcc 컴파일 명령어"
date: 2022-01-10
categories: [TIL]
tags: [c, gcc]
---
```shell
gcc [filename]
#기본 컴파일 명령어. 실행파일 이름은 a.out으로 생성

gcc -o [output name] [filename]
#실행파일 이름을 지정
gcc -o [output name] [f1] [f2] [f3]
#파일 여러개 하나로 컴파일

gcc -Wall
#모든 모호한 코드에 대해 경고
```