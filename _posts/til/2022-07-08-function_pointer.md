---
title: "함수 포인터"
date: 2022-07-08
categories: [TIL]
tags: [c]
---

# 개요

**함수**를 가리키는 포인터  
함수를 배열 또는 구조체에 넣거나, 함수 자체를 함수의 매개변수로 넘겨주고, 반환값으로 가져오기 위해 사용됨.

# 특징

- 함수 포인터도 포인터이기 때문에, 일반적인 포인터와 마찬가지로 **메모리 주소**를 가리킴.
- 하지만 일반적인 포인터와 달리, 함수 포인터는 **데이터**가 아닌 **코드의 위치**를 가리킴.
- 함수 포인터는 **코드의 시작부분**을 가리킴. (배열을 가리키는 포인터가 첫번째 값을 가리키는 것과 같음)
- 함수 포인터를 통해 메모리를 **할당**하거나 **회수**하는 것은 불가능. → 함수 포인터를 대상으로 malloc(), free() 함수 사용 불가.

# 사용 예시

## 매개변수가 없는 경우

`반환값자료형 (*함수포인터이름)();`

- void hello() 함수가 있을 때, `void(*fp)();` 형태로 함수 포인터 선언  
- `fp = hello;` 형태로 포인터에 함수 주소 저장.(함수이름이 포인터이므로 이름만 쓰면 됨.)  
	- `fp = hello;` : hello 함수의 메모리 주소를 함수 포인터 fp에 저장  
- `fp();` 형태로 호출

## 매개변수가 있는 경우

`반환값자료형 (*함수포인터이름)(매개변수자료형1, 매개변수자료형2);`

```c
int add(int a, int b)    // int형 반환값, int형 매개변수 두 개
{
    return a + b;
}
```

`int (*fp)(int, int);` int형 반환값, int형 매개변수 두 개가 있는 함수 포인터 fp 선언


# Ref.
<https://dojang.io/mod/page/view.php?id=592>  
<https://aahc.tistory.com/17>