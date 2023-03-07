---
title: "C++ 개념 정리 : 템플릿"
date: 2023-01-20
categories: [Programming Language, CPP]
tags: [cpp]
---

# 템플릿(template)

- 특징
    - **하나의 정의**로 **여러 타입에서 동작**할 수 있도록 해줌
        - 기존에는 자료형이 바뀔 때마다 대응되는 함수, 클래스를 새롭게 정의해줘야 했음.

## 함수 템플릿(function template)

- 개념
    - 함수의 **일반화**된 선언
    - 임의의 타입으로 작성된 함수에 특정 타입을 매개변수로 전달하면, **컴파일러가 알아서** 해당 타입에 맞는 **함수를 생성**함.
        - 함수 템플릿이 각각의 타입에 대해 처음으로 호출될 때, 컴파일러가 해당 타입의 인스턴스를 생성함.
        - 이후에 해당 타입의 함수 템플릿이 사용될 때마다 호출됨.
- 정의 방법
    
  ```cpp
  template <typename 타입이름>
  // 함수의 원형
  {
      // 함수의 본체
  }
  ```
    
- 예시
    
  ```cpp
  #include <iostream>
  
  template <typename T>
  void Swap(T &a, T &b)
  {
      T temp;
  
      temp = a;
      a = b;
      b = temp;
  }
  
  int main(void)
  {
      int c = 2, d = 3;
  
      std::cout << "c : " << c << ", d : " << d << std::endl;
      Swap(c, d);
      std::cout << "c : " << c << ", d : " << d << std::endl;
      std::string e = "hello", f = "world";
      std::cout << "e : " << e << ", f : " << f << std::endl;
      Swap(e, f);
      std::cout << "e : " << e << ", f : " << f << std::endl;
      return 0;
  }
  ```
    

## 클래스 템플릿 (class template)

- 개념
    - 클래스의 일반화된 선언
    - 클래스 템플릿에 전달되는 **템플릿 인수**에 따라 **별도의 클래스**를 만들 수 있음.
    - 함수 템플릿과 달리, 사용자가 사용하고자 하는 타입을 명시적으로 제공해야함.
- 정의 방법
    
  ```cpp
  template <typename 타입이름>
  
  class 클래스템플릿이름
  {
      // 클래스 멤버의 선언
  }
  ```
    
- 예시
    
  ```cpp
  template <typename T>
  
  class Data
  {
  private:
      T data_;
  
  public:
      Data(T dt);
      data(T dt);
      T get_data();
  };
  ```
    
- 특징
    1. 하나 이상의 템플릿 인수를 가지는 클래스 템플릿의 선언이 가능함        
        ```cpp
        template <typename T, int i> // 두 개의 템플릿 인수를 가지는 클래스 템플릿을 선언함.
        class X
        { ... }
        ```
        
    2. 클래스 템플릿에 디폴트 템플릿 인수를 명시할 수 있음.
        ```cpp
        template <typename T = int, int i> // 디폴트 템플릿 인수의 타입을 int형으로 명시함.
        class X
        { ... }
        ```
        
    3. 클래스 템플릿을 기초 클래스로 해서 상속할 수 있음.
        ```cpp
        template <typename Type>
        class Y : public X <Type> // 클래스 템플릿 X를 상속받음.
        { ... }
        ```
        

### 중첩 클래스 템플릿(nested class template)

- 개념
    - 클래스/클래스 템플릿 내에 또 다른 템플릿을 중첩해서 정의할 수 있음. **멤버 템플릿**이라고 함.
- 예시
    
  ```cpp
  template <typename T>
  
  class X
  {
      template <typename U>
  
      class Y
      {
          ...
      }
      ...
  }
  
  int main(void)
  {
      ...
  }
  
  template <typename T>
  template <typename U>
  X<T>::Y<U>::멤버함수이름()
  {
      ...
  }
  ```
    

### 명시적 특수화 (explicit specialization)

- **특정 타입**에 대해 **특별한 동작**을 정의할 수 있음.
- 예시
    - 임의의 템플릿 X가 있을 때, double형에 대해 특수화를 하면 다음과 같다.
  
  ```cpp
  template <> class X<double> { ... };
  ```
    

### 부분 특수화 (partial specialization)

- 템플릿 인수가 두 개 이상인 상황에서 **일부에 대해서만 특수화**를 해야할 때 사용.
- 사용 방법
    - template 키워드 직후의 꺾쇠에 특수화하지 않는 타입의 템플릿 인수를 명시하고,
    - 다음에 나오는 꺾쇠에 특수화하는 타입을 명시함.
- 예시
    
  ```cpp
  template <typename T1, typename T2>
  class X
  { ... };
  ```
    
    - 위와 같은 템플릿 X를 double형에 대해 부분 특수화를 하면 다음과 같다.
    
  ```cpp
  template <typename T1> class X<T1, double> { ... };
  ```
