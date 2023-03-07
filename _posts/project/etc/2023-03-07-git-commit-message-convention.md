---
title: "Git 커밋 메시지 관련 (Commit Message Convention)"
date: 2023-03-07
categories: [Project, etc_Project]
tags: [git]
---

> Git 커밋 메시지 관련 지침을 다룸.

# 개념
커밋 메시지를 일관되고 표준화된 방식으로 형식화하여 작성하기 위한 지침

# 사용 목적
- 명확성
    - 다른 개발자가 특정 커밋에서 변경된 내용을 더 **쉽게 이해**할 수 있다.
    - 코드 변경 내역을 리뷰하거나 문제를 파악할 때 특히 유용하다.
- 기록
    - 커밋 메시지는 **프로젝트 기록**의 일부이다.
    - 잘 짜여진 커밋 메시지는 추후 다른 개발자가 특정 변경 사항이 수행된 이유와 프로젝트의 더 큰 맥락에 어떻게 적합한지를 이해하는데 도움이 된다.
- 자동화
    - 특정 자동화 도구(automated-changelog, issue tracker 등)를 통해 커밋 메시지를 파싱하여 리포트나 **이슈 추적**을 할 수 있다.
- 협업
    - 지침을 따름으로서 커밋이 발생할 때 어떤 일이 일어날 지에 대해 **팀원 모두가 이해**할 수 있다.
    - 이로 인해 혼란과 오해를 줄일 수 있고 효과적으로 협업할 수 있다.

# 메시지 형태

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

# 유형별 작성법

## Subject

### type

일반적으로 사용되는 type은 다음과 같다.

- **feat**
    - 사용자를 위한 **새로운 기능**을 추가한 경우
    - 빌드 스크립트 기능 추가는 제외
- **fix**
    - 사용자 **버그를 수정**한 경우
    - 빌드 스크립트 수정은 제외
- **perf**
    - **성능**을 향상시킨 경우
- **docs**
    - **문서**를 수정한 경우
- **style**
    - 포매팅 변경, 세미콜론 누락 등 (생산 코드에는 변경 사항 없음)
- **refactor**
    - 생산 코드를 **리팩토링**한 경우
        - 리팩토링 : 결과의 변경 없이 코드의 구조를 재조정하는 것
    - 예시) 변수명 변경
- **test**
    - 누락된 테스트, 리팩토링 **테스트를 추가**한 경우 (생산 코드에는 변경 사항 없음)
- **build**
    - **빌드 설정**을 업데이트한 경우
    - 개발 툴을 업데이트한 경우
    - 그 밖에 사용자와 무관한 변화가 있는 경우
- **chore**
    - **가벼운 작업**을 업데이트하는 경우 (생산 코드에는 변경 사항 없음)
    - 예시) `.gitignore` 수정

### scope (optional)

- init
- runner
- watcher
- config
- web-server
- proxy
- etc.

## Body

- **명령형**으로 작성한다.
- 변경한 **이유**와 이전과의 **차이점**을 포함한다.
- **현재 시제**를 사용한다.
    - change(o)
    - changed(x)
    - changes(x)

## Footer

### 이슈 참조 (Referencing issues)

- **종료된 issue**의 경우 별도의 줄에 "Closed" 키워드로 작성해야 한다.
- 예시
    
    ```
    Closed #123
    ```
    

### 획기적인 변화 (Breaking changes)

- 변경 사항, 정당성 및 마이그레이션 참고 사항에 대한 설명을 작성해야 한다.
- 예시
    
    ```
    BREAKING CHANGE:
    
    `port-runner` command line option has changed to `runner-port`, so that it is
    consistent with the configuration file syntax.
    
    To migrate your project, change all the commands, where you use `--port-runner`
    to `--runner-port`.
    ```
    

# 작성 예시

```
fix(middleware): ensure Range headers adhere more closely to RFC 2616

Add one new dependency, use `range-parser` (Express dependency) to compute
range. It is more well-tested in the wild.

Fixes #2310
```

# Ref.

- [Semantic Commit Messages](https://gist.github.com/joshbuchea/6f47e86d2510bce28f8e7f42ae84c716)
- [Karma - Git Commit Msg](http://karma-runner.github.io/6.3/dev/git-commit-msg.html)
