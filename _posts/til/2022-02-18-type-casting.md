---
title: "[C언어] 자료형 변환"
date: 2022-02-18T14:59:52.761Z
categories: [TIL]
tags: [c]
---

# 자료형 변환
![type-casting](/assets/img/til/type-casting.svg)
## 형 확장 (암시적 형 변환)
자료형의 범위가 넓어지는 경우

## 형 축소 (명시적 형 변환) (= 형 변환)
자료형의 범위가 좁아지는 경우

int를 나누어서 float형인 변수에 저장하는 코드를 컴파일하게 되면 다음과 같은 경고가 발생한다.
```shell
test.c:8:14: warning: format ‘%d’ expects argument of type ‘int’, but argument 2 has type ‘double’ [-Wformat=]
     printf("%d MB",memory);
             ~^
             %f
```
이는 정수형인 int와의 계산을 실수형인 float에 저장하기 때문에 발생하는 문제이다. 이를 무시하고 실행시키면 소수점이 무시된 채로 `1.000000 MB`과 같이 결과가 출력된다.  
이처럼 int와의 계산에서 계산결과를 소숫점이 계산된 float형으로 얻으려면 계산하는 int형의 변수 중 일부를 float형으로 변환해야 한다.

`float memory = (float)h*b*c*s/8/1024/1024;` 

위 코드는 float형인 memory에 맞게 저장하기 위해 h를 float형으로 변환한 것이다.

[예제](https://codeup.kr/problem.php?id=1085)

# Ref.
<https://dojang.io/mod/page/view.php?id=493>