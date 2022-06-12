---
title: "[C언어] 함수 : 선언, 정의, 호출"
date: 2022-01-20
categories: [TIL]
tags: [c]
---

## 함수의 선언 (프로토타입)
컴퓨터는 왼쪽-> 오른쪽, 위->아래 방향으로 코드를 읽음.
따라서 아래에 정의된 함수를 위에서 사용할 때는 현재 함수 전에 프로토타입을 선언해야함.

```c
#include<stdio.h>

int main(void)
{
    int a = 10;
    int b = 20;
    test(a, b); //인자 a, b
}

void test(int c, int d) //매개변수 c, d
{
    printf("%d, %d", c, d);
}
```
위의 코드처럼 test함수 아래에 main함수가 나오는 형태가 아닌 main함수가 위에 오고 test함수를 호출하는 경우에는 아래와 같은 에러 메시지가 뜨는데 이는 프로토타입이 선언되지 않았기 때문.

## 함수의 정의
처음 함수를 공부하니 매개변수와 인자, 인수의 개념이 모호해서 혼용함. 그래서 자세히 찾아보니 아래와 같은 차이가 있었음.

### 매개변수(parameter), 인자
함수를 정의할 때 앞으로 들어올 인자의 형태. 변수.


### 인수(argument)
함수를 호출할 때 넣는 값. 이 값이 해당 함수에 전달됨.

```c
#include<stdio.h>

void test(int c, int d) //매개변수 c, d
{
    printf("%d, %d", c, d);
}

int main(void)
{
    int a = 10;
    int b = 20;
    test(a, b); //인자 a, b
}
```
## 함수의 호출
특정 함수를 사용하기 위함.
\*주의\* 함수를 호출할 때 함수이름 앞에 반환형(void 등)을 붙이지 않기!
예시)`void test(a, b);`

# Ref.
<https://www.tcpschool.com/javascript/js_function_parameterArgument>