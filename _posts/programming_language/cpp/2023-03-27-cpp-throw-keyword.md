---
title: "[C++] throw 키워드"
date: 2023-03-27
categories: [Programming Language, CPP]
tags: [cpp, til]
---

# 개요

std::exception을 상속받아서 예외 클래스를 작성하는 경우 예외 문자열을 받기 위해 what() 함수를 다음과 같이 정의한다.

```cpp
const char* what() const throw();
```

또한 예외를 발생시키기 위해서는 다음과 같이 exception 클래스의 객체를 생성하여 throw한다.

```cpp
throw MyException();
```

# throw의 의미

여기서 사용하는 두 가지의 throw는 서로 다른 의미를 지닌다.

## 예외를 발생시키는 throw

```cpp
throw MyException();
```

여기서 사용되는 throw는 **예외를 던지는** 키워드이다. throw뒤에 예외 클래스의 생성자를 호출하는 방식으로 사용한다.

## 예외 발생을 방지하는 throw

```cpp
const char* what() const throw();
```

반면 여기서 사용되는 throw()는 위와는 다른 의미를 가진다. 여기서는 what이 호출될 때 **다른 예외가 발생하지 않는다**는 것을 알려준다. 여기서의 throw는 **예외 사양**으로, 해당 함수가 실행될 때 던져질 수 있는 예외를 설정하는데 쓰인다.

- **예외 사양(exception specification)**이란?
    - 개념
        - 예외 사양은 함수가 **던질 수 있는 예외를 선언**할 수 있는 기능이다.
        - 이를 통해 함수 호출 시 처리해야 할 예외를 명확히 할 수 있다.
    - 사용법
        - `throw()` 함수를 사용
        - 함수의 인자로 발생을 허용할 예외를 명시한다.
    - 예시
        - `void MyFunction() throw(int, char*);`
        - 이 경우 myFunction 함수에서 int나 char* 두 종류의 예외 발생이 허용된다.
    - 문제점
        - 함수에서 throw로 명시하지 않는 예외가 발생할 경우 예상치 못한 동작이 일어날 수 있다.
        - 컴파일러의 성능 최적화를 방해할 수 있다.

# noexcept

두 번째 용도로 쓰이는 throw 키워드는 C++11에서 폐기 예정(deprecated)이 되었고 C++17에 와서는 완전히 폐기됐다.

대신 해당 키워드를 대체하는 noexcept 키워드가 나왔다. 던져질 수 있는 예외의 종류를 설정할 수 있었던 throw 키워드와는 달리 noexcept는 **예외를 발생시킬지**, 말지에 대해서만 설정할 수 있다.

# 참고

- [c++ - What is the use of throw() after a function declaration? - Stack Overflow](https://stackoverflow.com/questions/73551110/what-is-the-use-of-throw-after-a-function-declaration)
- [throw () after function declaration in c++ exception struct? - Stack Overflow](https://stackoverflow.com/questions/22352927/throw-after-function-declaration-in-c-exception-struct)
- [noexcept specifier  (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/noexcept_spec)
- [Dynamic exception specification (until C++17) - cppreference.com](https://en.cppreference.com/w/cpp/language/except_spec)
- [예외 사양(throw, noexcept)(C++) \| Microsoft Learn](https://learn.microsoft.com/ko-kr/cpp/cpp/exception-specifications-throw-cpp?view=msvc-170)
