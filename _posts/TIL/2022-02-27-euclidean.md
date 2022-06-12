---
title: "유클리드 호제법(Euclidean algorithm)"
date: 2022-02-27T14:48:38.276Z
categories: [TIL]
tags: [python, algorithm]
---
## 요약
2개의 자연수 또는 다항식의 최대공약수를 구하는 알고리즘

## 개념
2개의 자연수(또는 정식) a, b에 대해서 a를 b로 나눈 나머지를 r이라 하면(단, a>b), a와 b의 최대공약수는 b와 r의 최대공약수와 같다. 이 성질에 따라, b를 r로 나눈 나머지 r'를 구하고, 다시 r을 r'로 나눈 나머지를 구하는 과정을 반복하여 나머지가 0이 되었을 때 나누는 수가 a와 b의 최대공약수이다.

## 소스 코드

```py
def gcd(m,n):
    while n != 0:
       t = m%n
       (m,n) = (n,t)
    return abs(m)
```

# Ref.
<https://ko.wikipedia.org/wiki/유클리드_호제법>