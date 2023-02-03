---
title: "[C언어] 파일 입출력 함수"
categories: [Programming Language, C]
tags: [til, c, io]
date: 2022-10-06
---

# fopen
- **함수 원형**
	- `#include <stdio.h>`
    - `FILE *fopen(const char * restrict path, const char * restrict mode);`
- **설명**
    - path에 담긴 문자열을 이름으로 갖는 파일을 열어서 스트림에 연결시킨다.
- **매개변수**
    - path : 파일의 경로가 담긴 문자열
    - mode : 파일을 여는 모드
        - r
            - reading을 위해 여는 경우
            - 스트림이 파일의 시작 부분에 위치한다.
        - w
            - writing을 위해 여는 경우
            - 스트림이 파일의 시작 부분에 위치한다.
                - 파일이 존재 여부와 상관없이 처음부터 쓰기 시작한다.
        - a
            - writing을 위해 여는 경우.
            - 스트림이 파일의 끝 부분에 위치한다.
                - 파일이 존재하면 끝에서부터 내용을 추가한다. (w와의 차이점)
- **리턴값**
    - 성공적으로 완료된 경우 : **파일 포인터**
    - 그 외의 경우 : NULL, errno가 해당 오류로 설정된다.
- **open()과의 차이점**
    - open은 파일을 열어서 파일 디스크립터라는 정수를 얻는다.
    - fopen은 파일을 열어서 파일 스트림 정보를 얻는다.

# fread

- **함수 원형**
	- `#include <stdio.h>`
    - `size_t fread(void *restrict ptr, size_t size, size_t nitems, FILE *restrict stream);`
- **설명**
    - nitems개의 객체를, stream이 가리키는 스트림에서, 각각 size개의 바이트만큼 읽어서, ptr이 가르키는 위치에 저장한다.
- **리턴값**
    - 일반적인 경우 : 읽어들인 객체의 바이트 수
    - 오류가 발생하거나 EOF에 도달한 경우 : 0 또는 읽은 바이트 수(size보다는 작음)

# fclose

- **함수 원형**
	- `#include <stdio.h>`
    - `int fclose(FILE *stream);`
- **설명**
    - stream을 파일이나 함수와 분리한다.
- **리턴값**
    - 성공적으로 완료된 경우 : 0 반환
    - 그 외의 경우 : EOF 반환, errno가 해당 에러로 설정된다.

# fscanf

- **함수 원형**
	- `#include <stdio.h>`
    - `int fscanf(FILE *restrict stream, const char *restrict format, ...);`
- **설명**
    - 입력을 format에 맞게 scan한다.
    - 입력은 스트림 포인터인 stream으로 받는다.
    - scan의 결과는 포인터 인자에 저장된다.
- **리턴값**
    - 성공한 경우 : 읽어들인 데이터의 수
    - 형식 지정자와 일치하는 것이 없는 경우 : 0 반환
    - 아무것도 못읽고 입력에 실패한 경우 : EOF 반환