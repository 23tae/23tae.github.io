---
title: "멀티모달 AI 뉴스 플랫폼 '뉴스낵' 소개"
date: 2026-02-13T09:00:00.000Z
categories: [Project, 뉴스낵]
tags: [introduction]
---

![뉴스낵](/assets/img/project/newsnack/introduction/newsnack_cover.jpg)

**[뉴스낵 이용하기](https://newsnack.site/)**

## 들어가며

최근 신한 스퀘어브릿지 청년 해커톤 2기에 백엔드 개발자로 참여하였다. 이 해커톤은 6주 동안 참가자들이 실제 기업과 매칭되어 기업이 제안한 과제를 해결하는 프로젝트를 진행하는 방식으로 진행되었다.

나는 디자이너 2명, 프론트엔드 1명, 백엔드 1명으로 구성된 팀에서 백엔드 개발을 담당했다. 우리 팀은 AXZ에 배정되었는데, 이 글에서는 우리가 만든 멀티모달 AI 뉴스 플랫폼, **'뉴스낵(Newsnack)'**을 소개하고자 한다.

## 배경

AXZ의 가장 큰 고민은 **'이용자의 고령화와 신규 이용자의 부재'**였다. 플랫폼에 활력을 불어넣기 위해서는 젊은 층을 타깃으로 한 새로운 접근 방식이 필요했다.

기업 소개에서 플랫폼 이용자가 가장 많이 소비하는 콘텐츠는 **뉴스**라는 점을 확인했다. 뉴스 소비 경험을 개선하는 것이 유입 문제 해결에 가장 직접적인 임팩트를 줄 수 있다고 판단하고, 뉴스에 집중하기로 했다.

우리는 막연한 가설 대신, 데이터로 문제를 정의하기로 했다. 한국언론진흥재단이 발행한 '22년 디지털 뉴스리포트'에 따르면, 쏟아지는 정보량에 지쳐 한국 뉴스소비자의 약 **26%**가 뉴스를 의도적으로 피하는 **'의도적 뉴스 회피'** 현상이 나타나고 있었다.

직접 유저 리서치도 진행했다. 20~30대 **55명 설문 조사**와 핵심 타겟 **4명의 심층 인터뷰**를 통해 구체적인 인사이트를 도출했다.

- 응답자의 **80%**가 뉴스를 한 번 볼 때 **10분 미만**으로 짧게 소비하며, 주로 이동 중이나 자투리 시간에 소비한다.
    ![user_research_news_view_duration](/assets/img/project/newsnack/introduction/user_research_news_view_duration.png)
- 뉴스에서 불편함을 느끼는 이유로 **"정보 과잉과 텍스트 부담"**이라는 의견이 지배적이었다.
- "어떤 요소가 있으면 더 오래 머무나요?"라는 질문에 **"이미지 및 시각화 자료"** 와 **"텍스트 요약본"** 이라는 답변이 가장 많았다.
    ![user_research_news_engagement_factors](/assets/img/project/newsnack/introduction/user_research_news_engagement_factors.png)
- 뉴스 소비 시간대에 관한 질문에 대해서는 **출근/등교 시간대**에 소비한다는 응답이 가장 많았다.
    ![user_research_daily_usage_pattern](/assets/img/project/newsnack/introduction/user_research_daily_usage_pattern.png)

![affinity_diagram](/assets/img/project/newsnack/introduction/affinity_diagram.jpg)

수집한 응답들을 어피니티 다이어그램으로 정리하며 2030의 뉴스 소비 패턴을 구체화했다. 이 과정을 통해 우리 서비스가 지향해야 할 방향이 뚜렷해졌다.  
**짧고, 가볍고, 시각적인** 뉴스 경험이었다.


## 핵심 솔루션

![newsnack_logo](/assets/img/project/newsnack/introduction/newsnack_logo.png)

이러한 과정을 거쳐 뉴스를 가볍게 소비할 수 있는 **뉴스낵**이라는 솔루션을 도출했다. 주요 기능은 다음과 같다.

- **멀티모달 뉴스 큐레이션**: 단순한 텍스트 요약을 넘어, 2가지 포맷으로 뉴스를 제공한다.
  - 가볍고 재미있게 볼 수 있는 **'AI 뉴스툰'** (이미지)
    ![ai_newstoon](/assets/img/project/newsnack/introduction/ai_newstoon.jpg)
    _AI 뉴스툰_
  - 바쁜 출퇴근길에 들을 수 있는 **'오늘의 뉴스낵'** (오디오)
    ![today_newsnack](/assets/img/project/newsnack/introduction/today_newsnack.jpg)
    _오늘의 뉴스낵_
- **AI 에디터 페르소나**:
  딱딱하고 건조한 기자의 문체 대신, 각자의 독특한 말투와 관점을 가진 'AI 에디터'들이 뉴스를 큐레이션하고 재해석하여 훨씬 친근하게 전달한다.
  ![newsnack_editor](/assets/img/project/newsnack/introduction/newsnack_editor.png)
- **감정 중심 인터랙션**:
  복잡한 댓글 창 대신, 뉴스를 보고 느낀 감정(행복, 놀람, 분노 등)을 직관적으로 표현하고 스와이프하며 넘기는 UI를 통해 뉴스 소비 자체를 가벼운 놀이로 치환했다.
  ![newsnack_reaction](/assets/img/project/newsnack/introduction/newsnack_reaction.png)

## 시스템 아키텍처

이러한 아이디어를 실현하기 위해서는 단순히 API 하나를 서빙하는 것을 넘어, 파편화된 기사들을 수집하고, 분석하고, AI를 통해 재생산하는 견고한 인프라가 필요했다. 1인 백엔드라는 부담감이 있었지만, 데이터의 일관성과 확장성을 담보하기 위해 **초기부터 각자의 역할이 분리된 분산 아키텍처**를 설계했다.

![newsnack_system_architecture_20260212](/assets/img/project/newsnack/introduction/newsnack_system_architecture_v1.jpg)
_뉴스낵 시스템 아키텍처 (v1)_

### 아키텍처 방향 결정

분리 구조를 채택한 이유는 세 영역의 **자원 사용 패턴이 근본적으로 달랐기 때문**이다. 서비스가 커질수록 이 세 영역을 한 프로세스 안에 두면 서로 간섭할 수밖에 없다는 결론에 도달했다.

| 영역 | 실행 패턴 | 자원 특성 |
|---|---|---|
| **Spring Boot** | 사용자 요청마다 즉시 응답 | 짧고 빠른 I/O 위주 |
| **FastAPI** | AI 추론 1회당 수십 초 | 외부 AI API 응답 대기로 인한 장시간 I/O 블로킹 |
| **Airflow** | 스케줄에 따른 배치 실행 | 수집 → 군집화 → AI 호출 오케스트레이션 |

이 결론에 이르기까지 아래 네 단계의 검토를 거쳤다.

- **① 모놀리식 (Spring Boot 단일 서버)**: 이미지 생성(한 장당 1분)처럼 외부 AI API 응답을 기다리는 동안 Spring Boot의 스레드가 블로킹 상태로 오래 점유된다. 이 작업이 동시에 여러 건 쌓이면 스레드 풀이 고갈되어 일반 사용자 API 응답까지 지연되는 자원 경합 문제가 발생한다.
- **② AI 서버만 분리 (2개 서버)**: Spring Boot와 FastAPI를 나누더라도, 기사 수집·군집화 배치 작업을 Spring Boot에 남기면 스케줄 실행 시 동일한 자원 경합이 재발한다. 게다가 `scikit-learn`, `feedparser` 등 Python 생태계 의존성을 JVM 서버에서 처리하는 것은 구조적으로 부자연스럽다.
- **③ MSA**: 서비스 디스커버리, API Gateway, Database per Service 등 인프라 관리 포인트가 지나치게 많아 6주 안에 비즈니스 로직 완성이 불가능하다고 판단했다.
- **④ 역할 기반 3-Server 분리 (채택)**: 세 영역의 자원 사용 패턴이 다른 만큼 물리적으로 분리하되, MSA의 복잡한 인프라 계층(서비스 디스커버리 등)은 도입하지 않는다. 각 서버는 독립된 EC2 인스턴스에 배치했다.

### 기술 스택 결정

- **Spring Boot (Kotlin)**:
  ![spring_boot_kotlin_logo](/assets/img/project/newsnack/introduction/spring_boot_kotlin_logo.png)
  - JPA를 통한 RDS 연동, Spring MVC 기반 REST API 등 사용자 대면 서버에 필요한 기능을 검증된 생태계로 빠르게 구현할 수 있다. MVP 이후 사용자 인증 도입 시 Spring Security로 자연스럽게 확장할 수 있다는 점도 고려했다.
  - Kotlin은 null 안전성으로 NPE를 컴파일 단계에서 차단하고, 간결한 문법으로 보일러플레이트를 최소화한다.
- **FastAPI**:
  ![fastapi_logo](/assets/img/project/newsnack/introduction/fastapi_logo.png)
  - LangGraph·Google GenAI SDK·OpenAI SDK 등 **파이썬 AI 생태계**와의 호환성이 높다.
  - `asyncio` 기반 **비동기 처리**로 외부 AI API 응답을 기다리는 동안에도 이벤트 루프가 다른 요청을 동시에 처리할 수 있어, 장시간 I/O 블로킹이 잦은 AI 서버의 작업 특성에 적합했다.
- **Airflow**:
  ![airflow_logo](/assets/img/project/newsnack/introduction/airflow_logo.png)
  - DAG 내 태스크 간 의존성을 선언적으로 표현할 수 있다. 예를 들어 `content_generation_dag`는 이슈 선정 → 생성 요청 → 완료 대기 → 조립의 순차 의존성을 `>>`로 정의하고, 태스크 간 데이터를 `XCom`으로 전달한다.
  - 태스크 단위 실패 감지 및 재시도, Web UI를 통한 실행 이력 시각화로 운영 모니터링 부담을 줄일 수 있다.
  - 현재는 각 DAG가 고정된 스케줄로 독립 실행되지만, 군집화 알고리즘이 복잡해지거나 수집 규모가 커져 처리 시간이 늘어날 경우 `TriggerDagRunOperator`로 DAG 간 명시적 의존성을 추가하는 방향으로 자연스럽게 확장할 수 있다.

### 전체 파이프라인 구조

전체 구조는 크게 세 가지 독립된 서버로 구성된다.

1. **Orchestrator (Airflow)**:
   데이터 파이프라인의 심장이다. 20여 개 언론사의 RSS 피드를 주기적으로 수집하고, 유사한 기사들을 하나의 '이슈'로 군집화한다. 이후 통합된 이슈 데이터를 AI 서버로 넘겨 콘텐츠 생성을 조율하는 오케스트레이터 역할을 수행한다.
2. **AI Server (FastAPI & LangGraph)**:
   요청을 받아 AI 콘텐츠 생성 워크플로우를 관리하는 서버다. LangGraph를 활용해 기사를 분석하고 보조 이미지를 리서치한 후, 적절한 AI 에디터를 배정하여 LLM을 통해 이미지를 생성하거나 TTS 오디오를 만들어 S3에 적재한다.
3. **Core API Server (Spring Boot)**:
   프론트엔드와 직접 맞닿아 있는 메인 서버다. AI가 만들어낸 이슈와 콘텐츠(웹툰, 카드뉴스, 오디오 브리핑)를 DB에서 읽어와 클라이언트에게 페이지네이션하여 제공하고, 사용자의 감정표현 등의 비즈니스 로직을 전담한다.

각 서버는 별도의 EC2 인스턴스에 컨테이너화하여 통신하도록 구성했고, RDS(데이터베이스)는 보안을 위해 Private Subnet에 격리해두었다. 이러한 분리된 구조는 모듈 간의 결합도를 낮추고 각 시스템의 병목 원인을 정확히 찾아낼 수 있는 유연한 기반이 되었다.

## 마치며

6주간의 해커톤을 통해 뉴스낵의 MVP 개발을 성공적으로 마쳤다.

우리 팀은 앞으로도 서비스를 지속적으로 개선해 나갈 계획이다. 사용성 테스트를 진행하여 기능을 개선하고, 구체적인 마케팅 전략을 수립하여 유저를 유입시킬 것이다. 이후 체류 시간, 감정 표현의 전환율 파악 등 **다양한 지표**를 바탕으로 지속해서 피드백을 수용하고 서비스를 다듬어 나가는 진짜 운영의 사이클을 돌려보고자 한다.

뉴스낵이 유의미한 프로덕트로 성장하는 과정을 포스팅을 통해 하나씩 기록해 나갈 계획이다.
