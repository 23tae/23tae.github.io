---
title: "C++ 개념 정리 : namespace"
date: 2023-01-04
categories: [Programming Language, CPP]
tags: [cpp]
---

# namespace

## 개념
- 연관된 식별자(변수, 함수, 클래스 등)들을 하나의 이름으로 묶는데 사용
- 이름이 겹쳐서 발생하는 문제를 예방한다.
- 코드를 더욱 논리적으로 조직할 수 있게 해준다.

## using
- namespace의 name들을 현재 스코프에 보이도록 해준다.
- 예시
    
  ```cpp
  MyNamespace::myFunction();
  ```
  
  ```cpp
  using MyNamespace::myFunction;
  
  // 이제 myFunction을 바로 사용할 수 있다.
  myFunction();
  ```

## using namespace
- namespace 전체를 현재의 스코프로 가져온다.
- 예시
    
  ```cpp
  using namespace MyNamespace;
  
  // 이제 MyNamespace 내의 모든 name들을 바로 사용할 수 있다.
  myFunction();
  ```
        
##  예시
  - main 함수에서는 using namespace를 사용해서 cout, endl와 같이 사용 가능
  - print 함수에서는 using std::cout, using std::endl를 각각 선언해서 사용

```cpp
#include <iostream>

void print() {
  using std::cout;
  using std::endl;
  cout << "world!" << endl
}

int main() {
  using namespace std;
  cout << "hello" << endl;
  print();
  return 0;
}
```