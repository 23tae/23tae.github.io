---
title: "메모리 구조"
date: 2022-06-01T13:22:58.513Z
categories: [TIL]
tags: [cs]
---
# 메모리 공간의 구분

1. 코드(code) 영역
2. 데이터(data) 영역
3. 스택(stack) 영역
4. 힙(heap) 영역

![memory](/assets/img/til/memory_area.png)

*Heap과 Stack의 데이터 저장 방향은 운영체제 따라 다르다고 함.

# 메모리 해제

Data 영역: 프로세스가 종료되면 소멸함

Stack 영역: 함수 종료 시 제거됨

Heap 영역: **개발자가 직접 free !!**

# Ref.

<http://www.tcpschool.com/c/c_memory_structure>