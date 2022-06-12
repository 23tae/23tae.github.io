---
title: "void 포인터"
date: 2022-05-05T07:41:46.292Z
categories: [TIL]
tags: [c]
---
> 대상이 되는 데이터의 타입을 명시하지 않은 포인터

# 주요 특징
1. 어떤 자료형 포인터도 void 포인터에 넣을 수 있음

2. void 포인터를 어떤 자료형 포인터에도 넣을 수 있음

3. 역참조를 할 수 없음
(자료형이 없음 == 값을 가져오거나 저장할 size가 정해지지 않음)

# 유의할 점
사용하려면 명시적 형변환 과정을 먼저 거쳐야 함


# Ref.
<https://www.tcpschool.com/c/c_pointer_various>  
<https://dojang.io/mod/page/view.php?id=278>