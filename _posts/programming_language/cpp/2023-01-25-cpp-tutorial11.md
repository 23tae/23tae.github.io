---
title: "C++ 개념 정리 : 좌측값(lvalue), 우측값(rvalue)"
date: 2023-01-25
categories: [Programming Language, CPP]
tags: [cpp]
---

# 복사 생략 (copy elision)

```cpp
int main() {
  A a(1);  // 일반 생성자 호출
  A b(a);  // 복사 생성자 호출

  A c(A(2)); // 일반 생성자 호출 -> 복사 생략
}
```

위와 같은 경우에는 복사 생성자가 호출되지  않는다.

이를 **복사 생략(copy elision)**이라고 한다.

# 좌측값(lvalue), 우측값(rvalue)

```cpp
int a = 1;
```

위의 식에서 a를 좌측값, 1을 우측값이라고 한다.

- **좌측값**
    - 주소값을 취할 수 있는 값
    - 식의 양쪽 모두에 올 수 있다.
- **우측값**
    - 주소값을 취할 수 없는 값
    - 식의 오른쪽에만 올 수 있다.

```cpp
int a;         // a 는 좌측값
int& l_a = a;  // 좌측값 레퍼런스

int& r_b = 1;  // 오류 발생 (rvalue이기 때문)
```
