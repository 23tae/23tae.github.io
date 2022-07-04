---
title: "Dangling pointer(댕글링 포인터)"
date: 2022-06-01T13:24:31.950Z
categories: [TIL]
tags: [c]
---
# Description

해제된 메모리 영역을 가리키고 있는 포인터
![dangling](/assets/img/til/dangling_pointer.png)

```c
char *ptr1 = (char *)malloc(sizeof(char));
...
free(ptr1);
```

위의 ptr1이 가리키는 메모리 영역은 free함수로 해제됐지만 ptr1은 여전히 해당 영역을 가리킨다.(변수가 삭제되지 않았기 때문)

# Problem

예측 불가능한 동작 (메모리 접근 시)
Segmentation fault (메모리 접근 불가 시)
잠재적인 보안 위험

# Solution

메모리 해제 후 포인터를 NULL로 설정

```c
char *ptr1 = (char *)malloc(sizeof(char));
...
free(ptr1);
ptr1 = NULL;
```

# Ref.

<https://80000coding.oopy.io/77c32056-1ba6-40ad-9a2b-0d3cdf270888>  
<https://thinkpro.tistory.com/67>