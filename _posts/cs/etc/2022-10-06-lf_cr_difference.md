---
title: "Line Feed (LF)와 Carriage Return (CR)의 차이점"
date: 2022-10-06
categories: [Computer Science, etc_cs]
tags: [til]
---

# 의미

## Line Feed (LF)

`\n`

ASCII 코드 10번

커서를 줄의 시작 위치로 옮기지 않고 다음 줄로 넘긴다.

UNIX 계열 시스템에서는 new line으로 불린다.

## Carriage Return (CR)

`\r`

ASCII 코드 13번

커서를 다음 줄로 넘기지 않고 현재 줄의 시작 위치로 옮긴다.

# 기타

## End of Line (EOL)

`\r\n`

위의 CR과 LF를 합친 형태이다.

커서를 다음 줄로 넘김과 동시에 줄의 시작 위치로 옮긴다.

UNIX 계열이 아닌 운영체제(윈도우, 심비안 등)에서 new line 문자로 사용된다.

# Ref.

<https://stackoverflow.com/questions/1552749/difference-between-cr-lf-lf-and-cr-line-break-types>
