---
title: "Git Flow란?"
date: 2023-04-12
categories: [Project, etc_Project]
tags: [git, til]
---

# 개요

Git Flow는 Git 프로젝트 관리에 널리 쓰이는 **분기(branching) 모델**이다. 이 모델은 Vincent Driessen이 자신의 블로그에서 처음 소개하였으며 master, develop, feature, release, hotfix라는 이름을 가진 다섯 개의 브랜치를 사용한다.

# 주요 개념

![git_flow](/assets/img/project/git-flow.png)

Git flow는 Git의 **브랜치들을 구성**하고 이들이 어떻게 **상호작용**할 지를 정의하는데 사용하는 방법이다.

이 모델의 핵심은 두 개의 **main 브랜치(main, develop)**와 세 개의 **supporting 브랜치(feature, release, hotfix)**를 갖는 것이다.

## Main 브랜치

- `master`
    - 프로젝트의 공식 **릴리스 기록**을 저장하며 버전 번호를 태그한다.
- `develop`
    - **기능들을 통합**하는 용도로 쓰이며 `feature` 브랜치들이 준비가 되엇을 때 이곳에 merge한다.

## Supporting 브랜치

- `feature`
    - 프로젝트의 **새로운 기능**을 개발하거나 기존의 **기능을 향상**시키는데 사용된다.
    - `develop` 브랜치에서 생성되었다가 완성되면 다시 기존 브랜치로 merge된다.
- `release`
    - 프로젝트의 새로운 **릴리스를 준비**하는데 사용된다.
    - `develop` 브랜치에서 생성되었다가 릴리스가 끝나면 `master`와 `develop` 브랜치로 merge된다.
- `hotfix`
    - 긴급한 **버그나 이슈**를 고치기 위해 사용된다.
    - `master`에서 생성되었다가 수정을 마치고 나서 `master`와 `develop`으로 merge된다.

# 작동 방식

Git flow의 전체적인 흐름은 다음과 같다.

- `develop` 브랜치가 `master`로부터 생성된다.
- `feature` 브랜치가 `develop`으로부터 생성된다.
    - `feature` 브랜치가 완성되면 `develop` 브랜치로 merge된다.
- `release` 브랜치가 `develop`으로부터 생성된다.
    - `release` 브랜치의 사용이 끝나면 `develop`과 `master` 브랜치로 merge된다.
- `master`에 이슈가 발생하면 `master`에서 `hotfix` 브랜치가 생성된다.
    - `hotfix`가 완료되면 `develop`과 `master` 브랜치로 merge된다.

# 장단점

## 장점

Git flow가 다른 브랜칭 모델에 비해 우수한 점은 다음과 같다.

- 프로젝트에 대한 **확실한 구조**와 가이드라인을 제공한다.
- **릴리스**와 **버전**을 추적할 수 있게 해준다.
- 여러 기능과 수정이 **충돌없이 동시에** 이루어질 수 있게 해준다.
- 개발자 간의 **협업**과 코드리뷰를 용이하게 한다.
- **Master** 브랜치가 항상 **안정적**이며 **배포 가능**하다는 것을 보장한다.

## 단점

위와 같은 장점을 가진 Git flow이지만 단점도 존재한다.

- 복잡하다
    - Git flow는 복잡하며 설정과 관리에 일정 수준 이상의 전문 지식이 필요하다.
- 오버헤드가 높다
    - 다양한 브랜치와 merge 요청, 릴리스를 관리하는데에 상당한 오버헤드가 따른다.
- 피드백 주기가 느리다
    - 코드의 변경사항이 main 브랜치에 merge되기까지 여러 단계를 거쳐야 하기 때문에 개발자들이 피드백을 곧바로 받을 수 없다.
- 릴리스에 초점이 맞춰져있다
    - Git flow는 릴리스를 위한 브랜칭 모델이기 때문에 그렇지 않은 경우에는 맞지 않다.

# Git flow를 사용하면 좋은 경우

Git flow는 여러 개발자가 함께 작업하는 대규모의, 복잡한 프로젝트에서 특히 유용하다. 이 외에 git flow를 사용하면 좋은 경우는 다음과 같다.

- **여러 명의 개발자**가 하나의 프로젝트에서 작업하며 **통합**과 **테스트**에 명확한 프로세스가 필요한 경우
- 프로젝트의 **릴리스 주기가 길고** 사용자에게 릴리스하기 전에 **수차례의 테스트**와 품질 검증이 필요한 경우
- 엄격한 **버전 관리**와 코드베이스 변경사항에 대한 문서화가 필요한 경우
    - 코드베이스(codebase) : 특정 시스템, 애플리케이션, 컴포넌트 따위를 빌드할 때 사용되는 소스코드
    의 전체 집합, 그것을 담은 저장소

# 참고

- contents
    - [Gitflow Workflow \| Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
    - [Git Flow 개념 이해하기](https://ux.stories.pe.kr/183)
    - [우린 Git-flow를 사용하고 있어요 \| 우아한형제들 기술블로그](https://techblog.woowahan.com/2553/)
    - [코드베이스 - 제타위키](https://zetawiki.com/wiki/%EC%BD%94%EB%93%9C%EB%B2%A0%EC%9D%B4%EC%8A%A4)
- images
    - [Git Flow vs GitHub Flow](https://www.alexhyett.com/git-flow-github-flow/)
