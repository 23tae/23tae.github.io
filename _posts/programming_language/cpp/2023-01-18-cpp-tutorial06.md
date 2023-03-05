---
title: "C++ 개념 정리 : 상속, 문자열"
date: 2023-01-18
categories: [Programming Language, CPP]
tags: [cpp]
---

# 표준 string 클래스

- 특징
    - 연산자 +, == 등이 오버로딩되어 있기 때문에 strcmp등의 함수 없이도 문자열 간의 비교, 붙이는 작업 등이 가능함.
    - 문자열을 하나의 타입처럼 표현할 수 있게 해줌
- 선언과 초기화
    
    ```cpp
    string str1;              // 문자열의 선언
    str1 = "C++ Programming"; // 문자열의 초기화
    string str2 = "C++";      // 문자열의 선언과 동시에 초기화
    ```
    

# 클래스 상속 (class inheritance)

- **기존에 정의**되어 있는 클래스의 모든 멤버 변수와 멤버 함수를 **물려받아**, **새로운 클래스**를 작성하는 것
- 물려주는 클래스와 받는 클래스를 각각 **기반 클래스(base class)**, **파생 클래스(derived class)**라고 함.
- 파생 클래스 정의 방법
    
    ```cpp
    class Derived : public Base
    ```
    
    ![Untitled](/assets/img/programming_language/cpp/tutorial06/Untitled.png)
    

## 멤버 함수 오버라이딩(overriding)

- 개념
    - 상속받은 클래스에서 멤버 함수의 동작을 재정의하는 것
    - 함수의 원형은 기존 멤버 함수의 원형과 같아야 함.
- 방법
    1. 파생 클래스에서 직접 오버라이딩
    2. 가상 함수를 이용해 오버라이딩

## protected 접근 제어자

![Untitled](/assets/img/programming_language/cpp/tutorial06/Untitled%201.png)

- 개념
    - **상속받는 클래스**에서는 접근 가능하도록 함.
    - **파생 클래스**에 대해서는 **public** 멤버처럼 취급되며, **외부**에서는 **private** 멤버처럼 취급됨.
- 접근 가능한 영역
    1. 이 멤버를 선언한 클래스의 멤버 함수
    2. 이 멤버를 선언한 클래스의 friend
    3. 이 멤버를 선언한 클래스에서 public 또는 protected 접근 제어로 파생된 클래스

## "is a" vs "has a"

- `is a`
    - 개발자라는 클래스가 사람 클래스를 상속받는다면, `개발자 is a 사람`이라고 할 수 있음.
        - 개발자는 사람의 모든 기능을 사용할 수 있기 때문.
    - 업캐스팅
        - 파생 클래스의 객체가 기반 클래스로 형변환 되는 것.
            
            ![Untitled](/assets/img/programming_language/cpp/tutorial06/Untitled%202.png)
            
        - `Derived is a Base` 이므로 다음과 같이 대입할 수 있음.
            
            ```cpp
            Base* p_c = &c;
            ```
            
        - 위와 같은 방식으로 Derived 객체를 Base로 형변환할 수는 없다.(`다운 캐스팅`)
            
            ```cpp
            Derived* p_c = dyanmic_cast<Derived*>(p_p); // 오류 발생
            ```
            
        - 이 상태에서 오버라이딩한 함수 what을 호출하게 되면, **Base의 what이 호출**됨.
- `has a`
    - 자동차라는 클래스에 엔진이라는 멤버 변수가 있다면, `자동차 has a 엔진`이라고 할 수 있음.

## 다중 상속(multiple inheritance)

- 개념
    - 두 개 이상의 클래스로부터 멤버를 상속받아 파생 클래스를 생성하는 것.
    - 상속하는 순서에 따라 생성자의 호출 순서가 달라짐.
- 사용 방법
    
    ```cpp
    class 파생클래스이름 : 접근제어지시자 기초클래스이름, 접근제어지시자 기초클래스이름[, 접근제어지시자 기초클래스이름, ...]
    {
        // 파생 클래스 멤버 리스트
    }
    ```
    
- 주의 사항
    - 멤버 변수가 중복될 수 있다.
    - **다이아몬드 상속**
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial06/Untitled%203.png)
        

# Virtual 키워드

## 가상 함수(virtual function)

- `파생 클래스에서 재정의`할 것으로 기대하는 멤버 함수.
- 선언 방법
    - `virtual 키워드`를 사용.
    
    ```cpp
    virtual 멤버함수원형;
    ```
    

### 다형성(polymorphism)

- **동일한 이름**의 함수가 서로 다르게 동작하는 것.
- polymorphism : 여러가지 형태

## 가상 소멸자 (virtual destructor)

- 필요성
    - **클래스를 상속**할 때는 반드시 **기반 클래스의 소멸자**를 `virtual`로 선언해주어야 함.
- 이유
    
    ```cpp
    Parent *p = new Child();
    delete p;
    ```
    
    - 위와 같이 Base 클래스 포인터가 Derived 클래스 객체를 가리킬 때, 해당 객체를 delete하여도 Base 클래스의 소멸자가 호출됨. 메모리 누수 발생 가능.
        
        ```bash
        $> ./a.out
        --- 평범한 Child 만들었을 때 ---
        Parent 생성자 호출
        Child 생성자 호출
        Child 소멸자 호출
        Parent 소멸자 호출
        --- Parent 포인터로 Child 가리켰을 때 ---
        Parent 생성자 호출
        Child 생성자 호출
        Parent 소멸자 호출
        ```
        
- 기반 클래스의 소멸자를 virtual로 선언해주면 다음과 같이 Child 소멸자가 정상적으로 호출됨.
    
    ```bash
    $> ./a.out
    --- 평범한 Child 만들었을 때 ---
    Parent 생성자 호출
    Child 생성자 호출
    Child 소멸자 호출
    Parent 소멸자 호출
    --- Parent 포인터로 Child 가리켰을 때 ---
    Parent 생성자 호출
    Child 생성자 호출
    Child 소멸자 호출
    Parent 소멸자 호출
    ```
    
    - 기반 클래스의 소멸자가 호출되는 이유? (”Parent 소멸자 호출” 부분)
        - Child 소멸자를 호출하면서 해당 소멸자가 스스로 Parent 소멸자를 호출하기 때문.

## 동적 바인딩

- **바인딩(binding)**
    - 개념
        - 함수를 호출하는 코드에서 **어느 블록에 있는 함수를 실행**해야하는지 해석하는 것.
    - 종류
        - **정적 바인딩(static binding)** (또는 초기 바인딩)
            - 함수를 호출하는 코드가 컴파일 타임에 고정된 메모리 주소로 변환하는 방식.
            - 대상 : 가상함수를 제외한 모든 멤버 함수
        - **동적 바인딩(dynamic binding)** (또는 지연 바인딩)
            - 컴파일 타임에 객체를 알 수 없어서 런타임에 함수를 실행하는 방식.
            - **가상 함수**에서 사용. (기초 클래스 타입의 포인터나 참조를 통하여 호출될 때)
            - 예시
                - `p_c->what();` 와 같은 코드가 있을 때, what을 Base에서 실행할 지, Derived에서 실행할지는 런타임에 결정됨

## 추상 클래스(abstract class)

- 개념
    - **순수 가상 함수**를 포함하는 클래스.
    - 반드시 상속되어야 한다.
    - 인스턴스를 생성할 수 없음.
        - 파생 클래스만 인스턴스 생성이 가능함.
- 사용이 불가능한 경우
    1. 변수 또는 멤버 변수
    2. 함수의 전달되는 인수 타입
    3. 함수의 반환 타입
    4. 명시적 타입 변환의 타입

### 순수 가상 함수(pure virtual function)

- 개념
    - **파생 클래스**에서 **반드시 재정의**해야하는 멤버 함수.
        - 가상 함수는 재정의가 필수가 아니다.
    - 기반 클래스에 함수의 동작을 정의하는 **본체가 없다.**
- 선언 방법
    
    ```cpp
    virtual 멤버함수의원형 = 0;
    ```
    

# Ref.

[https://www.geeksforgeeks.org/multiple-inheritance-in-c/](https://www.geeksforgeeks.org/multiple-inheritance-in-c/)
