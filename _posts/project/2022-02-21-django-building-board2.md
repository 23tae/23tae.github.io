---
title: "Django 웹 개발 : 게시판 만들기 (2)"
date: 2022-02-21T05:34:08.540Z
categories: [Project]
tags: [django]
---
# 추가할 기능

- [ ] 답변 페이징과 정렬
- [ ] 카테고리
- [ ] 비밀번호 찾기와 변경
- [ ] 프로필
- [ ] 최근 답변과 최근 댓글
- [ ] 조회 수
- [ ] 소셜 로그인
- [ ] 마크다운 에디터


# 답변 페이징과 정렬
## 테스트 답변 만들기

`py manange.py shell`을 통해 장고 셸을 실행시킨다.

```shell
from pybo.models import Question
from pybo.models import Answer
from django.utils import timezone

for i in range(50):
    a = Answer(question_id=302, content='테스트 답변입니다:[%02d]' % i, create_date=timezone.now(), author_id=1)
    a.save()
```
위와 같이 입력하면 테스트 답변을 얻을 수 있다. for문의 `question_id`에는 본인이 답변을 입력할 question에 해당하는 id를 입력하면 된다.