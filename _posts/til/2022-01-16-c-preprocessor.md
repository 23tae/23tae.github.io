---
title: "[C언어] 전처리기"
date: 2022-01-16
categories: [TIL]
tags: [c]
---
## 전처리기 (Preprocessor)

전처리기는 번역의 첫 번째 단계의 일부로 소스 파일의 텍스트를 조작하는 텍스트 프로세서이다. 전처리는 소스 텍스트를 구문 분석하지 않지만 매크로 호출을 찾기 위해 토큰으로 나눈다. 비록 컴파일러가 일반적으로 첫 번째 패스에서 전처리를 호출하지만, 전처리는 컴파일 없이 텍스트를 처리하기 위해 별도로 호출될 수도 있다.

## 전처리기 지시문 (Preprocessor directives)

#define 및 #ifdef와 같은 전처리기 지시어는 일반적으로 소스 프로그램을 다른 실행 환경에서 쉽게 변경하고 컴파일할 수 있도록 하기 위해 사용된다. 소스 파일의 지시어는 전처리기에게 특정 작업을 수행하도록 지시한다.

예를 들어, 프리프로세서는 텍스트의 토큰을 바꾸거나, 다른 파일의 내용을 원본 파일에 삽입하거나, 텍스트 섹션을 제거하여 파일의 일부를 컴파일하지 못하도록 할 수 있다. 전처리 라인은 매크로 확장 전에 인식되고 수행된다. 따라서 매크로가 전처리기 명령처럼 보이는 것으로 확장되면 전처리기에서 인식되지 않는다.

전처리기 문은 이스케이프 시퀀스가 지원되지 않는다는 점을 제외하고 소스 파일 문과 동일한 문자 집합을 사용한다. 전처리기 문에 사용되는 문자 집합은 실행 문자 집합과 동일하다. 전처리기는 음수 문자 값도 인식한다.

### 전처리기가 인식하는 지시문의 종류
```
#define
#elif
#else
#endif
#error
#if
#ifdef
#ifndef
#import
#include
#line
#import
#include
#line
#pragma
#undef
#using
```

### #define 지시문(#define directive)

#define은 식별자 또는 매개 변수화된 식별자와 토큰 문자열의 연결인 매크로를 생성한다.  
매크로가 정의된 후 컴파일러는 소스 파일에서 식별자가 나타날 때마다 토큰 문자열을 대체할 수 있다.

**syntax**
```c
#define identifier token-stringopt
#define identifier ( identifieropt, ... , identifieropt ) token-stringopt
```

### #include 지시문(#include directive)

지시문이 나타나는 지점에 지정된 파일의 내용을 포함하도록 전처리에 지시한다.

**syntax**
```c
#include " path-spec "
#include < path-spec >
```

## Reference

[전처리기](https://docs.microsoft.com/ko-kr/cpp/preprocessor/preprocessor?view=msvc-170)  
[#define 지시문](https://docs.microsoft.com/ko-kr/cpp/preprocessor/preprocessor?view=msvc-170)  
[#include 지시문](https://docs.microsoft.com/ko-kr/cpp/preprocessor/hash-include-directive-c-cpp?view=msvc-170)