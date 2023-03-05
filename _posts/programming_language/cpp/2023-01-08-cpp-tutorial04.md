---
title: "C++ 개념 정리 : 객체지향, 생성/소멸자"
date: 2023-01-08
categories: [Programming Language, CPP]
tags: [cpp]
---

# 객체 지향

## 객체

![Untitled](/assets/img/programming_language/cpp/tutorial04/Untitled.png)

- **변수**들과 **참고 자료**들로 이루어진 소프트웨어 덩어리
- **추상화**가 필요함.
- 변수들이 외부로부터 보호됨.(**캡슐화**)
    
  ```cpp
  Animal animal;
  // 초기화 과정 생략
  
  animal.food += 100;         // --> 불가능
  animal.increase_food(100);  // --> 가능
  ```
  
  - **캡슐화** : 외부에서 직접 인스턴스 변수의 값을 바꿀 수 없고 항상 **인스턴스 메소드**를 통해서 간접적으로 조절하는 것
  - 객체의 내부를 완전하게 이해하지 않고 있어도 사용할 수 있음.

## 클래스

![Untitled](/assets/img/programming_language/cpp/tutorial04/Untitled%201.png)

- 클래스 : 객체가 어떻게 생성될지에 대한 정보를 담은 청사진.
- 인스턴스 : 클래스로 만든 객체
- 멤버 변수, 멤버 함수
- 접근 제어자
    - private : 객체 내에서 보호됨. 자기 객체 안에서만 접근 가능. 아무것도 명시하지 않으면 기본적으로 설정되는 제어자.
    - public : 외부에서 마음껏 이용 가능.

# 함수 오버로딩

- **같은 이름**의 함수를 중복해서 정의하는 것
    - 구분 방법
        - 전달되는 인자와 일치하는 함수를 찾음
    - 일치하는 인자가 없다면?
        - 자신과 최대로 근접한 함수를 찾음
- `[class_name]::[function_name]` 형태로 작성하여 함수를 클래스 밖에 정의할 수 있음

# 생성자

- 특징
    - 객체 생성 시 자동으로 호출되는 함수
    - 객체를 초기화 하는역할
    - 리턴값이 없다.
- 형식
    - `[class_name](parameter, ...)`
- 사용 예시
    
  ```cpp
  Date day(2011, 3, 1);         // 암시적 방법 (implicit)
  Date day = Date(2012, 3, 1);  // 명시적 방법 (explicit)
  ```
    

## 디폴트 생성자

- 개념
    - 사용자가 객체를 생성할 때 초기값을 명시하지 않으면 기본적으로 호출되는 생성자.
    - 사용자가 직접 정의할 수 있음
- 주의
    - 인자가 없는 생성자를 호출할 때는 `A a` 형식으로 써야함. `A a()` 로 쓰지 않도록 주의할 것.

## 생성자 오버로딩

- 생성자도 함수이므로 오버로딩 적용 가능.

## 복사 생성자

- 개념
    - 자신과 같은 클래스 타입의 **다른 객체에 대한 참조(reference)**를 인수로 전달받아, 그 참조를 가지고 **자신을 초기화**하는 방법
    - **깊은 복사**를 하도록 할 수 있음.
    - 객체를 처음 **생성 시**에만 호출됨.
    - 컴파일러가 **디폴트 복사 생성자**를 제공해줌.
- 복사의 종류
    - **얕은 복사**
        - 대입 연산자를 이용하여 객체에 객체를 대입하는 방법
          
          ```cpp
          Book web_book("HTML과 CSS", 350);
          
          Book html_book = web_book;
          ```
            
        - 실제 값을 복사하는 것이 아닌, **주소값을 복사**하는 방식.
        - 객체의 멤버가 heap 영역을 참조하는 경우 문제 발생 가능.
    - **깊은 복사**
        - **값을 복사**하는 방식.
- 형식
    
  ```cpp
  T(const T& a);
  ```
    
- 사용 예시
    - 복사 생성자 정의
        
      ```cpp
      Book::Book(const Book& origin) {
          title_ = origin.title_;
          total_page_ = origin.total_page_;
          current_page_ = origin.current_page_;
          percent_ = origin.percent_;
      }
        ```
        
        - 객체를 **상수** 레퍼런스로 받기 때문에 복사 생성자 내부에서 **데이터를 변경할수 없음.**
    - 복사 생성자 호출
        - 주요 방법
            
          ```cpp
          Book web_book("HTML과 CSS", 350);
          Book html_book(web_book);
          ```
            
        - 기타 방법
            
          ```cpp
          Book web_book("HTML과 CSS", 350);
          Book html_book = web_book;
          ```
            

### 디폴트 복사 생성자

- 개념
    - 컴파일러가 제공하는 기본 복사 생성자
    - 아무것도 하지 않는 디폴트 생성/소멸자와 다르게 **실제로 복사**해줌.
- 한계
    - `char *name`과 같은 포인터 형의 변수가 있을 때 아래와 같은 문제 발생.
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial04/Untitled%202.png)
        
    - 발생 원인
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial04/Untitled%203.png)
        
        - name에 대해 객체 pc1, pc2가 서로 같은 값(주소값)을 갖게 됨.
        - pc1을 소멸해도 pc2의 name은 해제된 pc1의 name과 같은 주소를 가리킨다.
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial04/Untitled%204.png)
        
    - 해결책
        - name에 대해 깊은 복사를 수행함. (이 경우 직접 복사 생성자를 만들어야 함.)

## 초기화 리스트 ****(Initializer list)****

- 생성자가 호출될 때 초기화를 해준다.

```cpp
Marine::Marine() : hp(50), coord_x(0), coord_y(0), damage(5), is_dead(false) {}
```

- 기존 방식과 다르게 멤버 변수의 **생성**과 **초기화**를 **동시에** 해준다.
    - 기존 방식 : 생성을 먼저 한 뒤에 대입을 해서 초기화를 한다.
- 클래스 내부에 **레퍼런스 변수**나 **상수**가 있는 경우 초기화 리스트를 통해 초기화해야한다.
    - 이들은 **생성과 동시에 초기화** 되어야 하기 때문.

# 소멸자

- 특징
    - 객체의 **수명이 끝나면** 컴파일러에 의해 **자동으로 호출**되며, 사용이 끝난 객체를 정리해준다.
    - 생성자의 반대 개념.
    - 소멸자가 필요없는 클래스라면 굳이 명시할 필요 없음.(디폴트 소멸자)
- 형식
    - `~[class_name]()`
- 사용 예시
    - 소멸자 정의
        
      ```cpp
      Marine::~Marine() {
        std::cout << name << " 의 소멸자 호출 ! " << std::endl;
        if (name != NULL) {
          delete[] name;
        }
      }
      ```
        
    - 소멸자 호출
        
      ```cpp
      ~Marine();
      ```
        

# Static 변수/함수

## Static 멤버 변수

- 개념
    - 클래스의 **모든 객체**가 **공유**하는 멤버 변수
    - 모든 객체가 하나의 static 멤버 변수를 사용한다.
- 형식
    
  ```cpp
  static int [variable_name];
  ```
    

## Static 멤버 함수

- 특징
    - 클래스 전체에 한 개만 존재하는 함수
    - 일반 함수와 달리 객체가 없어도 호출 가능.
    - 클래스의 static 변수만을 사용 가능.
- 사용 방법
    - `(클래스)::(static 함수)` 와 같이 사용.
        - `(객체).(멤버 함수)` 로 사용하지 않는 이유 : 객체가 아니라 클래스에 종속되기 때문이다.

# Const 멤버 함수

- 개념
    - **다른 변수의 값을 바꾸지 않는** 함수라고 명시할 수 있음.
    - 이 함수 안에서는 객체들의 읽기만 수행됨. 또한 여기서는 **다른 상수 함수만 호출**할 수 있음.
- 사용 방법
    
  ```cpp
  (함수의 정의) const;
  ```
    

# 주요 키워드

## this 포인터

- **객체 자신**을 가리키는 **포인터**
- 객체 자신의 주소를 리턴할 필요성이 있을 때 사용
- 클래스의 멤버 함수에서만 사용 가능.
- static 함수는 this 사용 불가능.

## explicit

- **암시적 변환이 되지 않도록** 해줌.
- 생성자를 다음과 같이 선언하면 됨.
  
  ```cpp
  explicit MyString(int capacity);
  ```
    
- explicit이 선언되어 있다면 생성자와의 타입이 다른 경우 암시적 변환이 되지 않고 다음과 같은 에러 발생
    
    ![Untitled](/assets/img/programming_language/cpp/tutorial04/Untitled%205.png)
    

## mutable

- 멤버 변수를 다음과 같이 선언하게 되면 const 함수 안에서도 해당 변수의 값을 변경할 수 있다.
    
  ```cpp
  mutable int data_;
  ```
    
- mutable을 사용하기 않고 const 함수 내에서 변수를 변경하게 되면 다음과 같은 에러가 발생한다.
    
    ![Untitled](/assets/img/programming_language/cpp/tutorial04/Untitled%206.png)
    

# 기타

- 검색 알고리즘
    - KMP 알고리즘
        - 불일치가 발생하기 직전까지 같았던 부분은 다시 비교하지 않고 패턴 매칭을 진행

# Ref.

[https://chanhuiseok.github.io/posts/algo-14/](https://chanhuiseok.github.io/posts/algo-14/)