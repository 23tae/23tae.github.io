---
title: "C++ 개념 정리 : 입출력"
date: 2023-01-19
categories: [Programming Language, CPP]
tags: [cpp]
---

# C++ 입출력 라이브러리

![Untitled](/assets/img/programming_language/cpp/tutorial07/Untitled.png)

- C++의 모든 입출력 클래스는 `ios_base`를 기반으로 한다.
- ios
    - 스트림 버퍼를 초기화 한다.
    - 데이터를 읽을 때 큰 덩어리로 불러와서 저장하는 곳.

## istream

- 입력을 수행하는 클래스
    
  ```cpp
  std::cin >> a;
  ```
    
- `>>` 연산자가 다양한 자료형으로 오버로딩 되어있다.
- **스트림의 상태**를 나타내는 **플래그**가 정의되어 있음.
    - `goodbit` : 스트림에 **입출력 작업이 가능**할 때
    - `badbit` : 스트림에 **복구 불가능한 오류** 발생시
    - `failbit` : 스트림에 **복구 가능한 오류** 발생시
    - `eofbit` : 입력 작업시에 **EOF 도달** 시
- 입력받는 진수는 기본적으로 10진수
    - 진수를 설정하는 방법
        1. 형식 플래그 (format flag)
        2. 조작자 (manipulator)

### 스트림 버퍼

- 스트림이란?
    - 문자들의 순차적인 나열
- `std::streambuf`
    - 문자열을 스트림처럼 이용할 수 있음.
    - 스트림의 상태를 나타내기 위해 세 개의 포인터를 정의함.
        - 시작 포인터 : 버퍼의 시작 부분을 가리킴.
        - 스트림 위치 지정자 : 읽을 문자를 가리킴.
        - 끝 포인터 : 버퍼의 끝 부분을 가리킴.

## fstream

- 파일 스트림
- istream, ostream을 각각 상속받아서 ifstream, ofstream이 됨.
- 소멸자에서 자동으로 close해주기 때문에 닫아줄 필요가 없음.
    - 한 객체가 여러 개의 파일을 읽어야 할 경우에는 close와 open이 필요함.
