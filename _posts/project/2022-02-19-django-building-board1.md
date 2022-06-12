---
title: "Django 웹 개발 : 게시판 만들기 (1)"
date: 2022-02-19T07:33:08.272Z
categories: [Project]
tags: [django]
---
![django](/assets/img/project/django.png)

# Django project 생성
`mkdir "프로젝트 최상위 폴더"`
`cd "프로젝트 최상위 폴더"`
`django-admin startproject config .`


# Django 기능 개발 순서

1. 템플릿에 추가 기능을 위한 링크나 버튼 추가
2. urls.py에 링크에 해당되는 URL 매핑을 작성
3. forms.py에 폼 작성 (폼이 필요없는 경우에는 생략가능)
4. views.py 파일에 URL 매핑에 추가한 함수 작성
5. 함수에서 사용할 템플릿 작성

# Ref
<https://wikidocs.net/71655>