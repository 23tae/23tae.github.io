---
title: "Memory leak(메모리 누수)"
date: 2022-06-01T13:20:55.023Z
categories: [TIL]
tags: [c]
---
free(buf) 와 buf = 0 의 차이?
   - free(buf) : 할당받은 메모리를 해제
   - buf = 0 : 할당받은 메모리 주소를 담고있는 buf 변수가 0 값을 갖게 됨 
    → memory leak!