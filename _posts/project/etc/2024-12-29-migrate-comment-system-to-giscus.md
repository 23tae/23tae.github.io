---
title: "Jekyll 블로그에 giscus 댓글 시스템 적용하기"
date: 2024-12-29T08:00:00.000Z
categories: [Project, etc_p]
tags: [blog]
---

최근 블로그 댓글 시스템을 utterances에서 giscus로 전환했다. 이 글에서는 giscus를 적용한 과정을 다룬다.

## 배경

### 기존 댓글 시스템

![utterances comment](/assets/img/project/etc/migrate-comment-system-to-giscus/utterances-comment-example.png)
_utterances에서 댓글을 관리하던 방식_

블로그를 처음 만들 당시에 댓글 시스템으로 **utterances**를 선택했다. utterances는 **GitHub Issues**를 기반으로 동작하며, 다음과 같은 장점이 있었다.

- **설치와 사용이 간단함**: 간단한 스크립트 삽입만으로 적용 가능.
- **데이터베이스 관리 불필요**: 별도 서버 없이 GitHub Issues로 댓글 저장.
- **GitHub 인증 지원**: GitHub 계정으로 로그인 가능.
- **광고 없음:** 더 나은 사용자 경험 제공.

그 이후로 별다른 문제 없이 계속 **utterances**를 사용해왔다.

### giscus로 전환

그러던 중 최근에 **giscus**라는 댓글 시스템을 알게 되었다. giscus는 **GitHub Discussions**를 기반으로 동작하며, **utterances**와 비교할 때 아래와 같은 장점이 있었다.

1. **GitHub Discussions 활용**: Issues보다 댓글 관리에 적합한 구조를 제공.
2. **게시글 반응(Reaction) 지원**: 댓글뿐만 아니라 게시글에 대한 사용자 반응(이모지)도 확인 가능.
3. **대화 뷰 제공**: 답글이 계층적으로 표시되어 대화 흐름 파악이 쉬움.

특히 **사용자 반응**과 **대화 흐름**을 볼 수 있는 부분이 블로그 운영을 더 편리하게 만들어줄 것 같아, giscus로 전환을 결정하게 되었다.

## giscus 적용

giscus를 블로그에 적용하는 과정은 크게 세 단계로 나뉜다.

### 댓글 관리용 저장소 준비

giscus는 **Public 저장소**에서만 작동한다. 내 블로그 저장소는 Private이었기 때문에 별도의 댓글 관리용 Public 저장소를 생성했다.

1. **저장소 준비 과정**
    1. **새로운 Public 저장소 생성** (예시: `blog-comments`)
    2. **Discussions 기능 활성화**
        - `Settings > General > Features`에서 **Discussions**를 활성화.
    3. **카테고리 설정**
        1. 댓글 카테고리 생성 (예시: `Comments`)
            
            ![create category](/assets/img/project/etc/migrate-comment-system-to-giscus/github-discussions-create-category.png)
            
        2. 기존 카테고리 삭제
            
            ![manage category](/assets/img/project/etc/migrate-comment-system-to-giscus/github-discussion-manage-category.png)
            
2. **giscus 앱 설치**
    1. [giscus 앱](https://github.com/apps/giscus)을 설치한다.
        
        ![install giscus](/assets/img/project/etc/migrate-comment-system-to-giscus/github-install-giscus-1.png)
        
    2. 댓글 관리용 저장소를 선택한다.
        
        ![install giscus](/assets/img/project/etc/migrate-comment-system-to-giscus/github-install-giscus-2.png)
        

### giscus 스크립트 생성

[giscus](https://giscus.app/ko)의 설정 마법사를 이용해 스크립트를 생성한다.

1. **저장소**
    - 위에서 만든 저장소로 설정 (예시: `23tae/blog-comments`)
2. **페이지 ↔️ Discussions 연결**
    - Discussion 제목이 페이지 `경로`를 포함
3. **Discussion 카테고리**
    1. 위에서 만든 카테고리로 설정 (예시: `Comments`)
    2. '이 카테고리에서만 Discussion 찾기' 체크
4. **기능**
    
    ![feature](/assets/img/project/etc/migrate-comment-system-to-giscus/giscus-generate-script-feature.png)
    

### 블로그 설정

위에서 생성한 스크립트를 바탕으로 `_config.yml` 파일을 설정한다.

```yaml
comments:
  active: 'giscus'

  giscus:
    repo: 'your-github-username/blog-comments'
    repo_id: 'your-data-repo-id'
    category: 'Comments'
    category_id: 'your-data-category-id'
    mapping: 'pathname'
    input_position:   # optional, default to 'bottom'
    lang:             # optional, default to the value of `site.lang`
```

## 결과

![result 2](/assets/img/project/etc/migrate-comment-system-to-giscus/result-2.png)
_Discussions에 추가된 댓글_

![result 3](/assets/img/project/etc/migrate-comment-system-to-giscus/result-3.png)
_답글과 이모지 모두 확인할 수 있다_

## 참고 자료

- [giscus](https://giscus.app/ko)
- [GitHub 블로그에 댓글 기능 추가하기 (giscus) \| 아무튼 워라밸](https://hleecaster.github.io/posts/github_blog_giscus/)
- [[Blog] GitHub Blog Giscus 댓글 설정 방법 \| 찬스의 개발 블로그 : Chance Devlog](https://blog.false.kr/posts/Personal/Blog/GitHub-Blog-Giscus-Setting.html)
