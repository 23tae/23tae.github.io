---
title: "[Git] 명령어"
date: 2022-01-07
categories: [TIL]
tags: [git]
---
```shell
git clone [git repository addr] [dir name]
#새 디렉토리에 git 연결

git init
#시작

git add *
#파일 추가

git status
#현재 상태 확인

git commit -m “text”
#스냅샷 추가

git config —global —edit
#편집

Git config user.name
#이름 설정

git config user.email
#이메일 설정

git push
#git 서버에 저장

git rm [filename]
#git 서버에 올라간 파일 삭제

git ls-files -o -i --exclude-standard
#git에서 ignore 되는 파일 찾는 명령어
```