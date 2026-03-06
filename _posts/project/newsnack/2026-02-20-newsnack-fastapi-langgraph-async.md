---
title: "FastAPI-LangGraph 연동 최적화: asyncio.to_thread 오남용 제거와 코루틴 전환"
date: 2026-02-20T09:00:00.000Z
categories: [Project, 뉴스낵]
tags: [troubleshooting, async, langgraph, fastapi]
---

## 들어가며

뉴스낵 백엔드 시스템은 FastAPI를 기반으로 수많은 외부 API(LLM 호출, AWS S3 통신 등)와 맞물려 돌아간다. 이러한 I/O 바운드 작업에서 병목을 막고 높은 처리량을 달성하기 위해 비동기(`async`/`await`) 프로그래밍을 도입했다. 또한 FastAPI의 `BackgroundTasks`를 사용하여 API 요청 시 즉각적으로 HTTP 202(Accepted)를 반환하고, 실제 LangGraph 기반의 파이프라인은 백그라운드 환경에서 실행되도록 진입점을 분리했다.

```python
# app/api/contents.py
@router.post("/today-newsnack", status_code=status.HTTP_202_ACCEPTED)
async def create_today_newsnack(
    request: TodayNewsnackRequest,
    background_tasks: BackgroundTasks
):
    # 무거운 LangGraph 파이프라인은 백그라운드 태스크로 넘겨버림
    background_tasks.add_task(workflow_service.run_today_newsnack_pipeline, request.issue_ids)
    
    return GenerationStatusResponse(
        status="accepted",
        message="오늘의 뉴스낵 생성 작업이 백그라운드에서 시작되었습니다."
    )
```

하지만 다중 에이전트 시스템인 LangGraph를 FastAPI와 연동하는 과정에서 비동기 프로그래밍 모델에 대한 부족한 이해로 여러 트러블슈팅을 겪었다. 이 글에서는 잘못된 `asyncio.to_thread` 활용이 가져온 이벤트 루프 부재(Missing Event Loop) 에러와, 이를 100% 비동기 네이티브 코드로 전환하며 해결한 과정을 분석한다.

## 1. 문제 상황: 이벤트 루프 부재와 파이프라인 중단

Postman으로 AI 콘텐츠 생성 API(`POST /ai-article`)를 호출했을 때, 의도와 다르게 애플리케이션 콘솔에 다수의 에러가 발생했다.

첫 번째는 **이벤트 루프 부재**와 관련된 에러였다.

```text
INFO:     127.0.0.1:53552 - "POST /ai-article HTTP/1.1" 202 Accepted
[Workflow] Starting single pipeline for issue with 1 articles
[Workflow] Error during generation for [1]: There is no current event loop in thread 'asyncio_0'.
```

두 번째는 **코루틴이 끝내 응답을 받지 못하고 소멸**해버렸다는 런타임 경고였다.

```text
RuntimeWarning: coroutine 'Pregel.ainvoke' was never awaited
```

API 요청은 `202 Accepted`로 정상 반환되었으나, 실제 백그라운드 환경에서는 AI 모델을 호출하지 않고 파이프라인 전체가 즉시 종료되는 현상이 발생했다.

## 2. 원인 분석: `asyncio.to_thread`의 오남용

가장 큰 원인은 백그라운드 태스크에서 그래프를 실행할 때 "비동기 프로그래밍"과 "멀티 스레딩"의 개념을 혼용하여 코드를 작성했다는 점에 있었다. 

초기에는 LangGraph의 실행 방식에 익숙하지 않아 파이프라인 처리가 메인 이벤트 루프를 블로킹할 것을 우려했다. 이를 회피하기 위해 그래프 실행 객체를 `asyncio.to_thread()`로 감싸 호출했다.

```python
# app/services/workflow_service.py (과거 코드)

async def run_ai_content_pipeline(self, initial_state: dict):
    # 의도: I/O 바운드 작업을 별도 워커 스레드로 분리하기 위함
    # 문제: 비동기 코루틴인 self.graph.ainvoke를 동기 스레드 환경으로 오프로딩함
    await asyncio.to_thread(self.graph.ainvoke, initial_state)
```

**가장 큰 문제는 `asyncio.to_thread()`의 역할을 잘못 이해한 데서 비롯되었다.**

- `self.graph.ainvoke`: 비동기 런타임 환경에서 실행되어야 하는 코루틴 객체다.
- `asyncio.to_thread(...)`: 전달받은 함수를 동기 함수로 간주하고, 메인 이벤트 루프의 블로킹을 방지하기 위해 별도의 워커 스레드에서 실행하는 역할을 한다.

`to_thread`는 동기 함수를 OS 레벨의 스레드에서 격리하여 실행하도록 돕는 유틸리티다. 여기에 비동기 함수인 `ainvoke`를 전달하게 되면, 워커 스레드는 생성되지만 해당 스레드 내부에는 코루틴을 대기하고 실행할 **메인 이벤트 루프**가 존재하지 않는다. 

결과적으로 스레드 내부에서 이벤트 루프를 찾을 수 없다는 `There is no current event loop` 에러가 발생했으며, 생성된 `ainvoke` 코루틴 객체는 스케줄링되지 못한 채 통신 없이 소멸되어 런타임 경고가 발생한 것이다.

## 3. 해결 방안: 전면 비동기(Async all the way) 전환

불필요한 워커 스레드 활용을 배제하고, FastAPI의 특성에 맞춰 **코루틴 기반의 100% 비동기 I/O 오프로딩**으로 아키텍처를 전면 수정했다.

### 3-1. LangGraph 노드의 `async def` 네이티브 선언

우선, 그래프를 구성하는 모든 노드(Node) 함수들을 동기가 아닌 비동기로 전환했다. LangGraph는 내부적으로 각 노드가 `async def`로 정의되어 있으면 알아서 이벤트 루프를 방해하지 않고 태스크를 병렬로 스케줄링해준다.

```diff
-def analyze_node(state: ArticleState):
-    response = analyze_llm.invoke(prompt)
-    return {"summary": response.summary}

+async def analyze_node(state: ArticleState):
+    # 노드를 async def로 변경하고 비동기 API(ainvoke) 채택
+    response = await analyze_llm.ainvoke(prompt)
+    return {"summary": response.summary}
```

### 3-2. 호출부의 래퍼(Wrapper) 걷어내기

이제 모든 노드가 비동기로 동작하므로 스레드를 강제로 생성하던 `asyncio.to_thread()` 로직을 제거했다. 워커 스레드에 낭비되던 리소스를 절약하고 현재 실행 중인 메인 이벤트 루프에 비동기 I/O 대기를 맡기는 정상적인 패턴으로 복구했다.

```python
# app/services/workflow_service.py (현재 코드)

async def run_ai_content_pipeline(self, source_article_ids: List[int], initial_state: dict):
    try:
        # 워커 스레드로 던지지 않고, 이벤트 루프 안에서 비동기 I/O로 순수하게 처리
        await self.graph.ainvoke(initial_state)
    except Exception as e:
        logger.error(f"[Workflow] Error during generation: {e}")
```

이러한 구조 변경을 통해 발생하던 에러를 해결함과 동시에, 서버가 AI 모델의 연산 결과를 기다리는 시간 동안 다른 HTTP 트래픽을 원활하게 처리할 수 있는 비동기 아키텍처의 이점을 극대화할 수 있었다.

## 마치며

이번 트러블슈팅은 무거운 I/O 작업을 피하기 위해 무분별하게 `asyncio.to_thread`를 통해 워커 스레드로 위임하려는 접근이 아키텍처에 미치는 부작용을 명확하게 보여주었다. `async def`, `to_thread`, `Event Loop`와 같은 비동기 프로그래밍의 핵심 개념을 다시 한번 확립할 수 있었으며, 외부 연동이 잦은 환경에서 일관된 비동기 설계를 유지하는 것이 성능과 안정성에 필수적이라는 것을 체감할 수 있었다.
