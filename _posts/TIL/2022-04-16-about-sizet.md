---
title: "size_t 자료형이란?"
date: 2022-04-16T02:10:07.000Z
categories: [TIL]
tags: [c]
---
# 사용하는 이유
컴파일하는 시스템이 32bit인지, 64bit인지에 상관없이 같은 값을 표현하기 위해서 사용

# 헤더
`stdlib.h`에 선언되어 있음

# 출력
`printf("%zu",a)`와 같이 사용해야 함