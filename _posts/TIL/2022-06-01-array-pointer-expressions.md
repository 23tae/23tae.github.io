---
title: "배열과 포인터를 활용한 다양한 표현"
date: 2022-06-01T13:28:37.547Z
categories: [TIL]
tags: [c]
---
`char arr[3] = {'a','b','c'};`

# 배열

## 요소

`arr[0]` : a

`arr[1]` : b

## 주소값

`&arr[0]` : 0x7ffd209e6cb9

`&arr[0]+1` : 0x7ffe523d59ca

# 포인터

## 요소

`*arr` : a

`(arr+1)` : b

`*(&arr[0]+1)` : b

## 주소값

`arr` : 0x7ffdb7fbe519

`arr+1` : 0x7ffc831da80a

# 배열에서의 주소값

예시 1.

```c
#include <stdio.h>

int main()
{
  char arr[] = {'a','b','c'};
  printf("%p\n", arr);
  printf("%p\n", arr+1);
  printf("%p\n\n", arr+2);
  printf("%p\n", &arr[0]);
  printf("%p\n", &arr[1]);
  printf("%p\n\n", &arr[2]);

  printf("%lu", sizeof(arr));
  return(0);
}
```

실행결과 1.

```shell
0x7ffe88b76139
0x7ffe88b7613a
0x7ffe88b7613b

0x7ffe88b76139
0x7ffe88b7613a
0x7ffe88b7613b
3
```

위와 같이 char타입의 배열을 만들게 되면 배열의 크기는 char형의 크기인 1바이트와 배열의 개수인 3개의 곱으로 나오게 됨.

주소값의 경우 자료형의 크기인 1만큼 간격이 발생함.

예시 2.

```c
#include <stdio.h>

int main()
{
  int arr[] = {1,2,3};
  printf("%p\n", arr);
  printf("%p\n", arr+1);
  printf("%p\n\n", arr+2);

  printf("%p\n", &arr[0]);
  printf("%p\n", &arr[1]);
  printf("%p\n\n", &arr[2]);

  printf("%lu", sizeof(arr));
  return(0);
}
```

실행결과 2.

```shell
0x7fffe86cbbf0
0x7fffe86cbbf4
0x7fffe86cbbf8

0x7fffe86cbbf0
0x7fffe86cbbf4
0x7fffe86cbbf8

12
```

반면 이렇게 배열을 int형으로 선언하게 되면 자료형의 크기인 4바이트와 개수인 3개의 곱인 12가 나오게 됨.

주소값의 경우 자료형의 크기인 4만큼 간격이 발생함.

- **arr+1의 의미** (포인터 변수 arr에 1을 더하는게 `ox7f…f1`로 1씩이 아니라 4씩 차이가 나는 이유?)
포인터 변수에 1을 더하는 것은 주소를 1 증가시키는 것이 아니라 다음 위치의 데이터로 이동한다는 의미이기 때문
    
- 포인터간의 뺄셈
    두 주소 사이에 값이 몇개가 있는지를 의미함.
    
- 포인터간의 덧셈
    오류가 발생함 (정의되어 있지 않음)
    

# Ref.
<https://m.blog.naver.com/tipsware/221192754086>