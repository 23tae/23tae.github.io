---
title: "Undefined ・ unspecified ・ implementation-defined behavior"
date: 2023-03-12
categories: [Programming Language, C]
tags: [cs, til, c]
---

# 개요

Undefined behavior, unspecified behavior, implementation-defined behavior는 C/C++ 언어에서 프로그램이 특정 상황에 어떻게 동작해야하는지를 나타내는 용어이다.

# Undefined behavior

## 개념

- 언어 명세가 프로그램의 **동작을 명시하지 않은 상황**을 가리킨다.
- 프로그램의 동작은 **예측할 수 없으며** 컴파일러, 하드웨어 및 기타 요소에 따라 달라질 수 있다.
    - 언어 명세 (language specification)
        - 프로그래밍 언어의 구문과 의미를 설명하는 문서이다.
        - 언어의 문법, 구문 규칙, 다양한 상황에서의 행동에 대한 공식적인 설명을 담고있다.

## 대표적인 사례
- null 포인터를 역참조하는 것
    ![Untitled](/assets/img/programming_language/c/undefined-behavior/Untitled.png)
    
- 배열의 한계를 넘는 인덱스에 write하는 것
    
    ![Untitled](/assets/img/programming_language/c/undefined-behavior/Untitled%201.png)
    
- 초기화되지 않은 변수를 사용하는 것
    - 아래는 long형 변수를 초기화하지 않고 출력한 것
        
        ![Untitled](/assets/img/programming_language/c/undefined-behavior/Untitled%202.png)
        
- 0으로 나누는 것
    
    ![Untitled](/assets/img/programming_language/c/undefined-behavior/Untitled%203.png)
    

# Unspecified behavior

## 개념

- 언어 명세에서 **여러 가지 유효한 동작을 허용**하지만 어떤 동작이 나타날지는 **지정하지 않은** 상황을 말한다.
- 프로그램의 동작은 **구현에 맡겨진다.**

## 대표적인 사례

- 초기화되지 않은 변수의 값
- 다수의 스레드가 공유 리소스에 액세스하는 순서
- 함수의 인자가 evaluated되는 순서
    - 예시. `func(a(), b())` 라는 함수가 있을 때, `a()`와 `b()` 중 어떤 것이 먼저 계산될 지에 대한 순서

# Implementation-defined behavior

## 개념

- 언어 명세가 **여러 가지 유효한 동작을 허용**하지만 구현이 **어떤 동작을 나타낼 것인지 문서화**해야 하는 상황을 의미한다.
- 프로그램의 동작은 **구현에 따르지만**, 프로그래머가 **사전에 예측**할 수 있도록 문서화되어야 한다.

## 대표적인 사례

- 자료형의 크기
    - int형, long형의 크기
- 부동 소수점 타입이 나타낼 수 있는 값의 범위
- `printf`가 형식 지정자(format specifier)와 일치하지 않는 인자를 넘겨받았을 때의 동작

# 참고

- [Undefined behavior - Wikipedia](https://en.wikipedia.org/wiki/Undefined_behavior)
- [Unspecified behavior - Wikipedia](https://en.wikipedia.org/wiki/Unspecified_behavior)
- [c++ - Undefined, unspecified and implementation-defined behavior - Stack Overflow](https://stackoverflow.com/questions/2397984/undefined-unspecified-and-implementation-defined-behavior)
- [What are 'Unspecified' and 'Implementation-Defined' behavior? \| C++ FAQ](https://64.github.io/cpp-faq/unspecified-impldefined/)