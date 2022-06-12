---
title: "조건문 안에서 변수 할당하기"
date: 2022-04-22T02:48:45.866Z
categories: [TIL]
tags: [c]
---
# 사용예시
```c
#include <stdio.h>

int main() {
  int a;
  if ((a = 1))
      printf("a의 값은 %d입니다", a);
  return 0;
}
```

# 동작원리
if문은 조건문 안의 값이 0이 아닐 때 작동한다.
위의 경우는 a에 1이라는 값을 먼저 할당한 후 조건문 안의 값이 1이 되어서 식이 동작하게 된다.

# Ref.
<https://dojang.io/mod/page/view.php?id=130>