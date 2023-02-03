---
title: "[C언어] 메모리 관련 함수 (할당/해제)"
categories: [Programming Language, C]
tags: [til, c, malloc]
date: 2022-10-07
---

# 메모리 할당

## malloc

- **함수 원형**
	- `#include <stdlib.h>`
	- `void * malloc(size_t size);`
- **설명**
	- size 바이트의 메모리를 할당한다.
- **리턴값**
	- 성공 시 : 할당된 메모리에 대한 포인터
	- 에러 발생 시 : NULL 포인터 반환. errno를 ENOMEM으로 설정

## calloc

- **함수 원형**
	- `#include <stdlib.h>`
	- `void * calloc(size_t count, size_t size);`
- **설명**
	- size 바이트인 객체에 대해 count개 만큼의 인접한 공간을 할당한다.
- **리턴값**
	- 성공 시 : 할당된 메모리에 대한 포인터
	- 에러 발생 시 : NULL 포인터 반환. errno를 ENOMEM으로 설정

## realloc

- **함수 원형**
	- `#include <stdlib.h>`
    - `void *realloc(void *ptr, size_t size);`
- **설명**
    - ptr이 가르키는 할당된 공간의 크기를 size로 변경한 뒤 ptr을 반환한다.
    - ptr의 메모리를 확장할 공간이 부족하다면?
        - 새롭게 할당한다.
        - 기존의 할당은 해제한다.
        - 할당된 메모리를 반한한다.
    - ptr이 NULL이라면 malloc과 동일하게 동작한다.
- **리턴값**
	- 성공 시 : 할당된 메모리에 대한 포인터
	- 에러 발생 시 : NULL 포인터 반환. errno를 ENOMEM으로 설정
	- 재할당에 실패한 경우 입력된 포인터는 유효하다.

# 메모리 해제
## free
- **함수 원형**
	- `#include <stdlib.h>`
	- `void free(void *ptr);`
- **설명**
	- ptr이 가리키는 메모리 블록을 할당 해제한다.
	- ptr이 NULL 포인터라면 어떤 동작도 수행하지 않는다.
- **리턴값**
	- 값을 반환하지 않는다.
- **예시**
	```c
	#include <stdlib.h>

	int main(void)
	{
		char *ptr;

		ptr = (char *)malloc(sizeof(char) * 10);
		free(ptr);
		return (0);
	}
	```