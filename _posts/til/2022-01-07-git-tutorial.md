---
title: "Git 기본 개념"
categories: [TIL]
tags: [git]
date: 2022-01-07
---

# Git이란?
Git은 리누스 토르발스가 개발한 **분산형 버전 관리 시스템**(Distributed Version Control Systems)이다.  
이 시스템을 통해 개발자가 중앙 서버에 접속하지 않고도 코드작업을 할 수 있다.

# 저장소의 종류
- 원격 저장소(Remote Repository)  
파일이 원격 저장소 전용 서버에서 관리되며 여러 사람이 함께 공유하기 위한 저장소
- 로컬 저장소(Local Repository)  
내 PC에 파일이 저장되는 개인 전용 저장소

# 로컬 저장소의 구성
Git의 로컬저장소 버전관리는 아래 3가지 영역을 통해 이루어진다.

- Working Directory  
내가 작업하고 있는 프로젝트의 디렉토리
- Staging Area  
커밋을 하기 위해 `git add` 명령어로 추가한 파일들이 모여있는 공간
- Repository  
커밋들이 모여있는 저장소

# 명령어
```shell
git clone [repo url] [dir name]
# 저장소 복사. 원격 저장소의 레포지토리를 로컬 저장소로 복사함.

git init
# git 저장소 초기화. 일반 디렉토리에 .git 디렉토리를 생성하여 깃 레포지토리로 전환함.

git add [filename]
# 파일 변경사항 추가. Working Directory에서 Staging Area로 변경내역을 추가함. [filename]에 * 또는 A를 입력하면 모든 파일의 변경사항이 추가됨.

git commit -m "text"
# 커밋 생성. git add를 통해 Staging Area로 넘겨준 모든 파일을 하나의 스냅샷으로 기록함.

git push
# 원격 저장소에 커밋 전송.

git pull
# 원격 저장소의 변경된 내용을 로컬 저장소로 가져옴.

git rm [filename]
# 로컬 저장소에 있는 파일을 삭제함. 이후에 commit을 통해 삭제내역을 반영해아함.

git log
# 커밋한 내역을 확인함.

git status
# 저장소 상태 체크. 저장소의 브랜치, 저장소 안의 변경사항 등을 확인함.

git config —global —edit
# 편집

Git config user.name
# 이름 설정

git config user.email
# 이메일 설정

git ls-files -o -i --exclude-standard
# git에서 ignore 되는 파일 찾는 명령어
```

# Ref.
<https://ko.wikipedia.org/wiki/분산_버전_관리>  
<https://git-scm.com/book/ko/v2/부록-C%3A-Git-명령어-스냅샷-다루기>  
<https://velog.io/@shin6403/Git-이란>  
<https://iseunghan.tistory.com/322>  