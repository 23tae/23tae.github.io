---
title: "[C언어] int형 배열의 길이 확인"
date: 2022-10-13
categories: [TIL]
tags: [cs, c]
---

# 개요

이번 주에 42서울 라피신 rush01 평가 봉사를 하면서 int형 배열의 길이를 `while(arr != '\0')`로 확인할 경우 `gcc -fsanitize=address -g *.c` 로 옵션을 지정하여 컴파일하면 buffer-overflow가 발생할 수 있다는 사실을 알게됐다.

# 원인

문자열의 경우 끝을 알려주는 종결자(terminator, sentinels)가 있어 while문으로 길이를 확인할 수 있다. 반면 int형 배열은 이러한 종결자가 없다.

또한 배열을 int형으로 생성하고 초기화를 하지 않으면 배열에는 쓰레기 값이 들어있다. 이 상황에서 while문으로 배열의 요소가 ‘\0’일 때까지 반복문을 돌린다면 위와 같은 문제가 발생할 수 있다. 아래는 예전 라피신 기간에 러쉬01을 하며 작성한 코드이다.

```c
int	check_length(int *args)
{
	int	i;

	i = 0;
	while (args[i] != '\0')
		i++;
	return (i);
}
```

# 해결 방법

이런 문제를 피하려면 `sizeof()` 연산자를 사용하면 된다. `sizeof(arr)/sizeof(arr[0])` 과 같이 배열의 크기를 자료형의 크기로 나눠주면 배열의 요소의 개수를 구할 수있다.

이 방식을 사용할 때의 주의할 점은 배열 부식이 발생할 수 있다는 것이다. 예를 들어 길이가 5인 배열의 요소의 개수를 구하는 함수를 정의하여 배열을 매개변수로 받은 경우 `sizeof(arr)/sizeof(arr[0])`은 `sizeof(int *)/sizeof(int)`가 되어 2(=8/4)가 나온다.

- **배열 부식 (array decay)**
    - 의미 : 배열을 매개변수로 넘길 경우 배열의 타입과 크기를 잃어버리게 되는 현상
    - 원인 : C에서 배열은 매개변수는 포인터로 취급된다.
    - 해결 : 배열을 매개변수로 넘길 때는 배열의 길이를 별도의 인자로 넘겨준다.

# Ref.

<https://www.quora.com/Is-there-a-null-character-too-in-an-integer-array-and-how-do-you-find-the-size-of-an-array-in-the-C-language>  
<https://www.geeksforgeeks.org/what-is-array-decay-in-c-how-can-it-be-prevented/>  
<https://www.geeksforgeeks.org/using-sizof-operator-with-array-paratmeters-in-c/>