---
title: "[C언어] 다양한 포인터"
date: 2022-03-08T01:13:21.899Z
categories: [TIL]
tags: [c]
---
# C
## void 포인터
`void *포인터이름;`

자료형이 정해지지 않은 포인터
역참조를 할 수 없음.  
[참고](https://velog.io/@23tae/void-%ED%8F%AC%EC%9D%B8%ED%84%B0)

## 이중 포인터
```c
#include <stdio.h>

int main()
{
    int *numPtr1;     // 단일 포인터 선언
    int **numPtr2;    // 이중 포인터 선언
    int num1 = 10;

    numPtr1 = &num1;    // num1의 메모리 주소 저장 

    numPtr2 = &numPtr1; // numPtr1의 메모리 주소 저장

    printf("%d\n", **numPtr2);    // 20: 포인터를 두 번 역참조하여 num1의 메모리 주소에 접근

    return 0;
}
```
![asterisk](/assets/img/til/pointer-asterisk.png)

## 함수 포인터
`반환값자료형 (*함수포인터이름)();`
함수 포인터와 저장될 함수의 반환값 자료형, 매개변수 자료형과 개수가 일치해야한다.
```c
//↓ 반환값 자료형
void (*fp)();    // 반환값과 매개변수가 없는 함수 포인터 fp 정의
//     ↑   ↖ 매개변수가 없음
// 함수 포인터 이름
```
# Ref.
<https://dojang.io/mod/page/view.php?id=279>  
<https://dojang.io/mod/page/view.php?id=592>