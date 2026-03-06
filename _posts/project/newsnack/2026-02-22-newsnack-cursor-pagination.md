---
title: "무한 스크롤과 동적 조회: Cursor-based Pagination과 QueryDSL 적용 사례"
date: 2026-02-22T09:00:00.000Z
categories: [Project, Newsnack]
tags: [spring-boot, querydsl]
---

![newsnack_main_page_scroll](/assets/img/project/newsnack/cursor-pagination/newsnack_main_page_scroll.webp)

## 들어가며

뉴스낵은 매일 새로운 기사가 쏟아지는 뉴스 피드 중심의 서비스다. 피드형 애플리케이션에서 조회 성능은 사용자 경험(UX)과 직결된다.

백엔드 API를 설계할 때 가장 먼저 선택해야 했던 것이 **페이징 전략**이었다. 기술적 선택지는 크게 두 가지였다.

- **Offset 기반**: `SELECT ... LIMIT N OFFSET M`으로 원하는 페이지를 잘라오는 방식. 구현이 단순하지만 **뒷 페이지로 갈수록 DB가 앞의 M건을 전부 읽고 버린 뒤 N건을 반환해야 하는** 풀 스캔 비용이 선형적으로 증가한다.
- **Cursor 기반(No-Offset)**: 마지막으로 조회한 데이터의 PK를 커서로 넘겨 `WHERE id < cursor` 형태로 인덱스를 직접 활용하는 방식. 페이지 깊이와 무관하게 일정한 성능(O(log N))을 보장한다.

매일 수백 개의 AI 뉴스가 누적되는 뉴스낵의 홈 피드는 "데이터가 언제까지 쌓일지 모르는 피드 구조"이므로, 처음부터 Offset을 선택할 이유가 없었다. 아울러 프론트엔드 UX 요구사항이 **무한 스크롤**로 확정되어 있었고, 무한 스크롤은 "마지막으로 본 아이템 이후"를 요구하므로 Cursor 기반이 비즈니스 요건과 기술적 적합성 모두에서 완벽히 부합했다. 결과적으로 **처음부터 Cursor 기반으로 설계**했다.

더불어 다차원적인 감정 표현(Happy, Sad 등)에 따른 동적 정렬 쿼리를 타입-세이프하게 작성하기 위해 **QueryDSL**을 함께 채용했다. 이 글에서는 뉴스 피드 조회 성능을 고도화하기 위한 백엔드 적용 사례를 다룬다.



## 페이징 전략 결정: Cursor(No-Offset) 기반을 선택한 이유

전통적인 `LIMIT N OFFSET M` 방식의 쿼리는 페이지 번호가 뒤로 갈수록 심각한 성능 부하를 유발한다. 예를 들어 `OFFSET 100000 LIMIT 10`을 요청하면, 데이터베이스는 앞의 10만 건을 전부 읽은 뒤 버리고 마지막 10건만 반환해야 한다. 매일 수백 개의 AI 뉴스가 누적되는 뉴스낵의 홈 피드에서는 적합하지 않은 방식이다.

- **장점**: `WHERE id < lastId`와 같이 인덱스를 직접적으로 활용하기 때문에, 1페이지를 조회하든 1만 번째 페이지를 조회하든 쿼리의 스캔 범위와 응답 속도가 일정하다(O(1) 혹은 O(log N)에 가까움).
- **무한 스크롤 UX와의 적합성**: 모바일 웹 프론트엔드의 무한 스크롤 스펙은 명시적인 "페이지 번호"가 아닌 "마지막으로 본 아이템 이후"를 요구하므로, 비즈니스 요건과 기술적 적합성이 완전히 일치했다.

## N+1 쿼리 최적화: 'Count 쿼리' 없애기와 Limit + 1 기법

Cursor 기반 페이징의 또 다른 장점은 무거운 `COUNT(*)` 쿼리를 날릴 필요가 없다는 점이다. 기존 Spring Data JPA의 `Page` 객체를 쓰면 전체 레코드 개수를 구하기 위해 별도의 Count 쿼리가 필수적으로 실행되었다.

우리 팀은 "다음 페이지가 남아있는지" 여부(hasNext)만 알면 되었기에 **요청받은 Size보다 딱 1개 더 가져오는 논리(Limit + 1)**를 적용했다.

```kotlin
// 요청한 사이즈가 10개라면, DB에서는 11개를 조회하도록 LIMIT 설정
val fetchSize = requestSize + 1

val results = queryFactory.selectFrom(aiArticle)
    .where(
        ltArticleId(lastArticleId) // 커서 필터링
    )
    .orderBy(aiArticle.id.desc())
    .limit(fetchSize.toLong())
    .fetch()

// 11개가 조회되었다면 다음 페이지가 존재(hasNext = true)하는 것으로 판단하고, 반환 시에는 끝에 1개를 잘라냄
val hasNext = results.size > requestSize
if (hasNext) {
    results.removeAt(requestSize)
}
```

이 방식을 통해 불필요한 전체 테이블 카운트 연산을 생략하여 DB 리소스를 획기적으로 절약했다.

## QueryDSL을 통한 동적 쿼리와 타임 윈도우 통제

![newsnack_best_article_by_category](/assets/img/project/newsnack/cursor-pagination/newsnack_best_article_by_category.png)

API가 고도화되며 메인 홈 화면에는 "최근 24시간 내 카테고리별 베스트 기사"를 올려주는 기능이 추가되었다. 여기에서 순수 JPA나 JPQL 대신 **QueryDSL**이 강력한 도구가 되었다. 

특히 큐레이션 알고리즘에서 "과거 기사가 상위를 독점하는 현상(예전 기사가 좋아요를 독식하여 계속 상단에 머무는 현상)"을 방지하기 위해, 우리는 조건 동적 주입을 통해 타임 윈도우(Time Window) 기반 폴백 전략을 수립했다.

```kotlin
override fun findBestByCategory(categoryId: Int): AiArticle? {
    // 타임 윈도우 설정: 24시간
    val oneDayAgo = Instant.now().minus(1, ChronoUnit.DAYS)

    // 1전략: 24시간 이내 기사 중 반응(Reaction Count)이 가장 좋은 최신 기사 1건 (No-Offset 기반)
    val bestIn24h = queryFactory.selectFrom(aiArticle)
        .where(
            aiArticle.category.id.eq(categoryId),
            aiArticle.publishedAt.after(oneDayAgo)
        )
        // 감정(Reaction) 엔티티를 조인하여 동적 정렬 수행
        .orderBy(reactionCount.totalCount.desc(), aiArticle.publishedAt.desc())
        .fetchFirst()

    // 2전략 (Fallback): 만약 24시간 내 발행된 카테고리 기사가 없으면, 단순 최신순으로 1건을 조회 (Index Scan 활용)
    return bestIn24h ?: queryFactory.selectFrom(aiArticle)
        .where(aiArticle.category.id.eq(categoryId))
        .orderBy(aiArticle.publishedAt.desc())
        .fetchFirst()
}
```

QueryDSL 특유의 자바 리플렉션과 QClass를 통한 타입 안전한(Type-Safe) 쿼리 작성 덕분에, 위와 같은 복잡한 분기와 정렬 로직을 컴파일 시점에 완벽히 검증할 수 있었다. 오타로 인한 런타임 쿼리 에러를 사전에 발견하고 방지할 수 있었다.

## 마치며

이번 API 설계 과정은 "데이터베이스가 어떻게 인덱스를 통해 데이터를 읽어오는지"에 대한 이해를 높이는 계기가 됐다. JPA의 편리한 추상화 뒤에서 무심코 발생시키던 Offset 풀 스캔과 N+1 Count 쿼리의 비용을 제거함으로써, 스케일 아웃 없이도 안정적으로 트래픽을 감당하는 기반을 마련했다.

다양한 조건과 정렬을 요구하는 피드형 서비스의 백엔드에서, Cursor-based Pagination과 QueryDSL의 결합은 효과적인 조회 솔루션임을 직접 확인했다.

## 참고 자료

- [Paginating large datasets in production: Why OFFSET fails and cursors win \| Sentry](https://blog.sentry.io/paginating-large-datasets-in-production-why-offset-fails-and-cursors-win/)
- [Pagination: Offset or Cursor — Which One is Right for You? \| by Mus Priandi \| Medium](https://medium.com/@mus.priandi/pagination-offset-or-cursor-which-one-should-you-choose-4f543f924c54)
- [페이징 성능 개선하기 - No Offset 사용하기](https://jojoldu.tistory.com/528)
