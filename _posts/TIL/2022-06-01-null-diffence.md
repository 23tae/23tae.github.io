---
title: "0, ‘\\0’, NULL 차이점"
date: 2022-06-01T13:15:57.286Z
categories: [TIL]
tags: [c]
---
# 의미

```c
char null_1 = '\0';
char null_2 = 0;
char null_3 = (char)NULL;
```

3개 모두 결과는 동일하지만 해석방식의 차이가 있음.

null_1 : \0의 아스키 값인 0이 저장됨

null_2 : 0이 바로 저장됨

null_3 : NULL은 헤더파일에 (void *)0으로 저장되어 있음.→ 0이 저장됨.

### 예외사항

`char null_4 = '0';`

null_4 : ‘0’은 문자 0을 의미하기 때문에 아스키코드 상의 값인 48이 저장됨.

# Ref.

<https://modoocode.com/29>  
<https://code4human.tistory.com/116>