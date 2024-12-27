---
title: "시대팅 매칭 API 개선 과정: 대규모 트래픽 대응 전략"
date: 2024-12-21T08:00:00.000Z
categories: [Project, 시대팅5]
tags: [spring-boot, caching, redis]
---

## 배경

![match-result-page](/assets/img/project/sidaeting5/04-matching-api/match-result-page.png)
_매칭 결과 페이지_

이번 시즌은 매칭 신청 기간 동안 1:1과 3:3 미팅에 대해 참가 신청을 받은 뒤, 매칭 알고리즘으로 매칭한 결과를 주말동안 공개하는 방식으로 운영하였다. 특히 매칭 결과 조회는 발표 당일 대규모 트래픽이 몰릴 것으로 예상되었고, 이를 효율적으로 처리하기 위해 API를 지속적으로 개선해야 했다.

매칭 결과 조회는 다음과 같은 절차로 진행된다.

1. **신청 내역 조회**: 사용자의 매칭 신청 내역(1:1 또는 3:3)을 조회한다.
2. **매칭 성공 여부 조회**: 매칭 성공 또는 실패 여부를 확인한다.
3. **상대 정보 확인**: 매칭된 상대가 보낸 메시지(편지)를 확인한다.

### 도전과제

**데이터베이스의 복잡성**

![db schema](/assets/img/project/sidaeting5/01-retrospect/sidaeting-erd.png)
_ERD_

매칭 API는 여러 테이블 간의 복잡한 관계를 처리해야 했다. 주요 테이블은 다음과 같다.

- `meeting_user`: 유저의 기본 정보.
- `user_information`: 유저의 세부 정보.
- `meeting_team`: 유저의 시대팅 신청 여부를 팀 단위로 관리.
- `user_team`: `meeting_user`와 `meeting_team` 간의 중간 테이블
- `match`: 매칭 성공한 팀 정보를 저장.
- `payment`: 결제 기록.

이처럼 여러 테이블 간의 관계가 얽혀 있어 쿼리를 작성하는 과정에 어려움이 있었다.

**대규모 트래픽에 대한 대비**

매칭 결과가 발표되는 시점에는 수백명에서 천명 사이의 신청자가 **한꺼번에 매칭 결과를 조회**할 것으로 예상되었다. 하지만 데이터베이스 쿼리를 비효율적으로 작성하면 서버 응답 시간이 느려지거나, 심각한 경우 서버가 다운될 수 있었다. 따라서 **데이터베이스 최적화**와 **캐싱 전략** 도입이 중요한 과제로 떠올랐다.

## 문제점

처음에는 시간에 쫓겨 API 설계에 대한 충분한 고민 없이 개발을 진행했다. 하지만 이후 API 테스트 과정에서 설계상의 문제점들이 계속해서 드러났다.

### 엔드포인트 설계상의 문제점

기존 API 엔드포인트는 아래와 같았다.

1. `GET /api/match/me/participations` : 유저의 시대팅 신청 여부를 조회
2. `GET /api/match/teams/{meetingTeamId}/result` : 유저의 매칭 결과를 조회
3. `GET /api/match/{matchId}/partner` : 유저의 매칭 상대 정보를 조회

이러한 엔드포인트에서 **매칭된 상대방의 정보**를 조회하기 위해서는 **세 번의 API 호출**이 필요했다.

```
1. GET /api/match/me/participations
   → meetingTeamId 획득

2. GET /api/match/teams/{meetingTeamId}/result
   → matchId 획득

3. GET /api/match/{matchId}/partner
   → 최종적인 상대방 정보 획득
```

Postman으로 API를 테스트하는 과정에서도 이런 구조의 불편함을 직접 체감할 수 있었다. 각 API의 응답값을 다음 API의 파라미터로 직접 복사해서 넣어줘야 했고, 이는 테스트를 번거롭게 만들었다.

이런 구조는 백엔드 뿐만 아니라 프론트엔드에서도 여러 문제가 예상되었다.

- 불필요한 API 호출 증가
- 복잡한 데이터 흐름으로 인한 상태 관리의 어려움
- API 의존성으로 인한 에러 처리의 복잡성

### 캐싱 적용의 실패

매칭 결과 데이터는 유저가 매칭된 이후에는 변경되지 않는다. 그렇기에 API 호출시 매번 데이터베이스를 조회할 필요가 없다고 판단했다. 따라서 캐싱 전략을 도입하기로 결정하였다.

**초기 설계와 문제점**

초기에는 아래처럼 서비스 내부에서 `@Cacheable`을 적용한 메소드를 다른 메소드가 호출하는 구조로 설계하였다. 하지만 API 호출 시 캐싱이 제대로 동작하지 않는 문제가 발생했다. 이는 `@Cacheable`의 동작 원리에 대해 충분히 이해하지 못한 채로 사용했기 때문이었다.

```kotlin
@Service
@Transactional(readOnly = true)
class MatchingService() {
    @Cacheable(value = ["meeting-participation"], key = "#userId", unless = "#result == null")
    fun getUserMeetingParticipation(userId: Long): MeetingParticipationResponse {
        ...
    }

    @Cacheable(value = ["match-result"], key = "#meetingTeamId", unless = "#result == null")
    fun getMatchResult(userId: Long, meetingTeamId: Long): MatchResultResponse {
        // 미팅 참여 여부 확인
        val participation = getUserMeetingParticipation(userId)
        ...
    }

    @Cacheable(
        value = ["partner-info"],
        key = "#matchId + ':' + #userId",
        unless = "#result == null"
    )
    fun getMatchedPartnerInformation(
        userId: Long,
        matchId: Long
    ): MeetingTeamInformationGetResponse {
        val matchResult = getMatchResult(userId, teamType)
        ...
    }
}
```

**Spring AOP와 프록시**

스프링 AOP(Aspect Oriented Programming)는 핵심 비즈니스 로직과 부가 기능을 분리하여 관리하는 프로그래밍 방식이다. 여기서 실제 객체를 감싸고 있는 래퍼(Wrapper) 객체인 프록시가 실제 객체 대신 요청을 처리한다.

`@Cacheable`과 같은 AOP 기반의 어노테이션을 사용하면, 스프링은 해당 클래스의 프록시 객체를 생성한다. 이 프록시 객체는 메소드 호출을 가로채서 캐시 처리와 같은 부가 기능을 수행한 뒤, 필요한 경우에만 실제 객체의 메소드를 호출한다.

**캐시 동작 방식**

다음은 `@Cacheable`을 사용한 예시 코드이다.

```kotlin
@Service
class MatchingService(
    private val matchingRepository: MatchingRepository
) {
    @Cacheable(value = ["matching"])
    fun getMatchingResult(id: Long): MatchingResult {
        return matchingRepository.findById(id)
    }

    fun processMatching(id: Long): MatchingResult {
        return getMatchingResult(id)
    }
}
```

**외부**에서 `getMatchingResult()` 메소드를 호출할 때의 흐름은 다음과 같다.

1. 클라이언트가 프록시 객체의 `getMatchingResult()` 호출
2. 프록시가 캐시에서 데이터 조회 시도
3. 캐시에 데이터가 있으면 즉시 반환
4. 캐시에 데이터가 없으면 실제 객체의 메소드를 호출하여 데이터를 조회하고 캐시에 저장

하지만 **같은 클래스 내부**에서 호출(`processMatching()`)할 때는 다음과 같이 동작한다.

1. `processMatching()`이 `getMatchingResult()` 직접 호출
2. 프록시를 거치지 않고 실제 객체의 메소드가 바로 실행됨
3. 캐시 처리가 수행되지 않음

**해결 방법**

이 문제를 해결하기 위한 방법은 크게 두 가지가 있다.

1. **캐시 처리를 별도의 클래스로 분리**
    ```kotlin
    @Service
    class CacheableMatchingService(
        private val matchingRepository: MatchingRepository
    ) {
        @Cacheable(value = ["matching"])
        fun getMatchingResult(id: Long): MatchingResult {
            return matchingRepository.findById(id)
        }
    }
    
    @Service
    class MatchingService(
        private val cacheableService: CacheableMatchingService
    ) {
        fun processMatching(id: Long): MatchingResult {
            return cacheableService.getMatchingResult(id)
        }
    }
    ```
    
2. **자기 자신의 프록시 객체를 주입받아 사용 (Self-Injection)**
    ```kotlin
    @Service
    class MatchingService(
        @Autowired
        private val self: MatchingService,
        private val matchingRepository: MatchingRepository
    ) {
        @Cacheable(value = ["matching"])
        fun getMatchingResult(id: Long): MatchingResult {
            return matchingRepository.findById(id)
        }

        fun processMatching(id: Long): MatchingResult {
            return self.getMatchingResult(id)
        }
    }
    ```
    

### 트래픽에 대한 고려 부족

![pr comment](/assets/img/project/sidaeting5/04-matching-api/pr-comment.png)
_Pull Request에서 받은 피드백_

기존 API에 대규모 트래픽이 발생할 경우 문제가 예상된다는 의견을 받기도 했다. 따라서 API를 재설계하기로 결정했다.

## 개선 사항

### API 구조 재설계

가장 먼저, 각 메소드가 독립적으로 동작하도록 API 구조를 변경했다. 즉, 하나의 메소드가 다른 메소드를 호출하지 않도록 변경하고, 엔드포인트 간의 의존성을 최소화했다.

우선 API 엔드포인트 구조를 다음과 같이 변경했다.

1. `GET /api/match/me/participations` (기존과 동일)
2. `GET /api/match/{teamType}/info` : 유저의 매칭 정보를 조회

이렇게 하여 API 호출 간의 의존성을 제거하고 클라이언트 데이터 흐름을 간소화하였고 API 호출 횟수를 줄였다. 또 경로 파라미터로 미팅 유형(`teamType`)을 받아 리소스를 명확하게 구분하였으며 추후 시즌이 거듭될 경우를 고려하여 쿼리 파라미터로 시즌 정보(`season`)을 받도록 하였다.

### 캐싱 전략 개선

캐싱 문제를 해결하기 위해 다음과 같은 방식을 도입했다.

- **`@Cacheable` 적용 방식 수정**
    - 캐싱 메소드를 다른 메소드에서 내부 호출하지 않고, **API 레벨에서 직접 호출**하도록 변경했다.
    - **캐시 키 형식**
        - `meeting-participation::{season}:{userId}` : 시대팅 신청 여부
        - `match-info::{season}:{userId}` : 매칭 정보

      ```
      127.0.0.1:6379> keys *
        1) "meeting-participation::5:23"
        2) "match-info::5:SINGLE:35"
      ```
        
- **캐시 워밍(Cache Warming) 도입**
    - 매칭 데이터를 **미리 캐시에 적재**하여 발표 시점에 대규모 트래픽의 캐시 미스(cache miss)로 인한 서버 과부하를 방지했다.
    - 이를 위해 Admin API에 **캐시 초기화**를 수행하는 API를 구현하였다.
        
      ```kotlin
      class AdminApi() {
        @PostMapping("/cache/warmup")
        fun triggerCacheWarmup(@RequestParam season: Int): ResponseEntity<Unit> {
            adminService.warmUpCacheAsync(season)
            return ResponseEntity.status(HttpStatus.NO_CONTENT).build()
        }
      }
      ```
        
    - 캐시 워밍 관련 함수는 **비동기 처리**를 하여 서버가 블로킹 되는 것을 방지했다.
        
      ```kotlin
      @Service
      class AdminService() {
          @Async
          fun warmUpCacheAsync(season: Int) {
              logger.info("[캐시 웜업 시작]")
      
              try {
                  logger.info("[미팅 참여 정보 캐시 웜업 시작]")
                  val allUsers = userRepository.findAll()
                  allUsers.forEach { user ->
                      user.id?.let { userId ->
                          try {
                              matchingService.getUserMeetingParticipation(userId, season)
                          } catch (e: Exception) {
                              logger.info("[미팅 참여 정보 캐시 웜업 실패] userId: $userId message: ${e.message}")
                          }
                      }
                  }
                  logger.info("[미팅 참여 정보 캐시 웜업 완료] 대상 인원: ${allUsers.size}")
              } catch (e: Exception) {
                  logger.error("[미팅 참여 정보 캐시 웜업 전체 실패] message: ${e.message}")
              }
      
              try {
                  logger.info("[매칭 결과 캐시 웜업 시작]")
                  val participants = userTeamDao.findAllParticipantsBySeasonAndType(season)
                  participants.forEach { participant ->
                      try {
                          matchingService.getMatchInfo(participant.userId, participant.teamType, season)
                      } catch (e: Exception) {
                          logger.info(
                              "[매칭 결과 캐시 웜업 실패] userId: ${participant.userId} message: ${e.message}"
                          )
                      }
                  }
                  logger.info("[매칭 결과 캐시 웜업 완료] 대상 인원: ${participants.size}")
              } catch (e: Exception) {
                  logger.error("[매칭 결과 캐시 웜업 전체 실패] message: ${e.message}")
              }
              logger.info("[캐시 웜업 완료]")
          }
      }
      ```
        

### 개선 결과

위 개선사항을 적용한 결과, API 응답 시간이 97% 가량 단축되었다.

- **기존 방식**
    
    ![image.png](/assets/img/project/sidaeting5/04-matching-api/before-cache.png)
    _기존 API 호출 시간_

- **개선된 방식**
    
    ![image.png](/assets/img/project/sidaeting5/04-matching-api/after-cache.png)
    _개선된 API 호출 시간_
    

## 결론

이번 개선 작업으로 매칭 결과 발표일에 다수의 트래픽이 발생했음에도 서버가 안정적으로 운영될 수 있었다. 특히 API 호출 수를 줄이고 캐싱을 활용함으로써 데이터베이스 부하를 크게 줄일 수 있었다.

이번 매칭 API 개선 작업은 복잡한 테이블 구조를 고려하여 쿼리를 최적화하고, 대규모 트래픽을 효과적으로 처리하기 위한 설계를 고민할 수 있었던 좋은 경험이었다. 특히 캐싱을 통해 성능을 최적화하는 과정에서 캐시의 역할과 동작 방식을 명확히 이해할 수 있었다.

## 참고자료

- [caching - Spring cache @Cacheable method ignored when called from within the same class - Stack Overflow](https://stackoverflow.com/questions/12115996/spring-cache-cacheable-method-ignored-when-called-from-within-the-same-class)
- [Invoke Spring @Cacheable from Another Method of Same Bean \| Baeldung](https://www.baeldung.com/spring-invoke-cacheable-other-method-same-bean)
- [What is Cache Warming? - GeeksforGeeks](https://www.geeksforgeeks.org/what-is-cache-warming/)