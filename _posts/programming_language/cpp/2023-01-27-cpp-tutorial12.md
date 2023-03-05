---
title: "C++ 개념 정리 : 스마트 포인터"
date: 2023-01-27
categories: [Programming Language, CPP]
tags: [cpp]
---

# 스마트 포인터(smart pointer)

- 포인터처럼 동작하는 클래스 템플릿
- 사용을 마친 `메모리를 자동으로 해제`해준다.
- C++에서 메모리 누수를 방지하기 위해 제공

## 동작 방식

1. 기본 포인터(raw pointer)에 new 키워드로 메모리를 할당
2. 해당 포인터를 스마트 포인터에 대입하여 사용
3. 소멸자가 delete 키워드를 사용해 메모리를 자동으로 해제

## 종류

- unique_ptr
- shared_ptr
- weak_ptr

## unique_ptr

![ptr](/assets/img/programming_language/cpp/tutorial12/Untitled1.png)

- 개념
    - 포인터를 통해 **다른 객체를 소유**하고 관리하며 unique_ptr이 범위를 벗어날 때 해당 객체를 폐기하는 스마트 포인터
    - **객체**에 `소유권` 개념을 도입함
- 특징
    - unique_ptr이 소멸되면, 가리키던 객체도 소멸됨
    - 해당 객체의 소유권을 가지고 있을 때만, 소멸자가 해당 객체를 삭제할 수 있다.
    - `move()` 멤버 함수로 소유권을 이전 가능. 복사는 불가능
- 사용 예시
    
  ```cpp
  unique_ptr<int> ptr01(new int(5)); // int형 unique_ptr인 ptr01을 선언하고 초기화함.
  
  auto ptr02 = move(ptr01);          // ptr01에서 ptr02로 소유권을 이전함.
  
  // unique_ptr<int> ptr03 = ptr01;  // 대입 연산자를 이용한 복사는 오류를 발생시킴. 
  
  ptr02.reset();                     // ptr02가 가리키고 있는 메모리 영역을 삭제함.
  
  ptr01.reset();                     // ptr01가 가리키고 있는 메모리 영역을 삭제함.
  ```
    
- 기타
    - 함수에 포인터를 넘기는 방법
        - `get()` 함수 사용.
            
          ```cpp
          int main() {
            std::unique_ptr<A> pa(new A());
            do_something(pa.get());
          }
          ```

## shared_ptr

![ptr](/assets/img/programming_language/cpp/tutorial12/Untitled2.png)

- 개념
    - 포인터를 통해 객체의 공유 소유권을 유지하는 스마트 포인터
- 특징
    - 참조 횟수(reference count)
        - 참조하고 있는 스마트 포인터의 개수
        - 특정 객체에 새로운 shared_ptr이 추가될 때마다 1씩 증가
        - 수명이 다할 때마다 1씩 감소
    - 마지막 shared_ptr의 수명이 다하여, 참조 횟수가 0이 되면 delete 키워드를 사용하여 메모리를 자동으로 해제
- 정의 방법
    - `make_shared()` 사용
        
      ```cpp
      std::shared_ptr<A> p1 = std::make_shared<A>();
      ```
        
    - 예시
        
      ```cpp
      shared_ptr<Person> hong = make_shared<Person>("길동", 29);
      
      cout << "현재 소유자 수 : " << hong.use_count() << endl; // 1
      auto han = hong;
      cout << "현재 소유자 수 : " << hong.use_count() << endl; // 2
      han.reset(); // shared_ptr인 han을 해제함.
      cout << "현재 소유자 수 : " << hong.use_count() << endl; // 1
      ```
      
- 주의사항
    - 인자로 주소값을 전달하면, 자신이 해당 객체를 첫 번째로 소유한다고 착각하게 됨.
        - 따라서 아직 참조중인 객체를 소멸하여 문제 발생 가능.

## weak_ptr

![ptr](/assets/img/programming_language/cpp/tutorial12/Untitled3.png)

- 개념
    - 하나 이상의 **shared_ptr** 인스턴스가 소유하는 **객체에 대한 접근**을 제공하면서도 소유자의 수에는 포함되지 않는 스마트 포인터
- 특징
    - shared_ptr의 **순환 참조 문제**를 해결하기 위해 도입됨.
        - 순환 참조 문제 : 서로가 서로를 참조하기 때문에 영원히 해제되지 않는 문제
