---
title: "전처리기 지시어 (include, define, ...)"
date: 2022-03-21T16:20:41.247Z
categories: [TIL]
tags: [c]
---
전처리기 지시어 (Preprocessor Directive)는 프로그램을 실행시킬 때가 아닌, 컴파일할 때 동작함

## #include
헤더파일 삽입 지시자

- 컴파일러에서 기본 제공된 파일을 포함할 때
`#include <filename>`

- 직접 만든 파일을 포함할 때
`#include "filename"`

## #define
기호상수를 정의하는 지시자
보통 변수와 구분하기 위해 **대문자**로 작성함

예시)
```c
#define NAME "thk"
#define PI 3.14
```


# Ref.
<https://dev-alice.tistory.com/5>