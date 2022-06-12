---
title: "[Python] List Comprehension"
date: 2022-03-03T07:40:53.569Z
categories: [TIL]
tags: [python]
---

## List Comprehension이란?
리스트를 간단하게 한 줄로 표현하는 파이썬 문법

## 형태
`[ ( 변수를 활용한 값 ) for ( 사용할 변수 이름 ) in ( 순회할 수 있는 값 )]`

## 예시

```py
size = 10
arr = [i * 2 for i in range(size)]

print(arr)
```

```shell
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

# Ref
<https://shoark7.github.io/programming/python/about-list-comprehension-python>