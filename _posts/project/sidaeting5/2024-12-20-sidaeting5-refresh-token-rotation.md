---
title: "Refresh Token Rotation으로 보안 강화하기"
date: 2024-12-20T08:00:00.000Z
categories: [Project, 시대팅5]
tags: [spring-boot, jwt, cookie]
mermaid: true
---

## 배경

![image.png](/assets/img/project/sidaeting5/03-jwt/refresh-token-rotation-example.png)
_출처: https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them_

JWT 인증 기능을 개발할 당시에는 토큰 재발급시 **액세스 토큰만 재발급**하도록 구현했었다. 시대팅의 운영 기간은 짧았기 때문에 리프레시 토큰의 유효 기간을 7일로 설정하고, 재발급 없이 액세스 토큰만 갱신하는 방식을 선택했었다. 그러나 이 방식은 **리프레시 토큰이 탈취**될 경우, 공격자가 해당 토큰을 이용해 액세스 토큰을 재발급받을 수 있는 보안 취약점이 있었다. 이를 해결하기 위해, 리프레시 토큰이 탈취되더라도 재발급된 **새로운 리프레시 토큰**을 통해 보안을 강화할 수 있는 **Refresh Token Rotation** 방식을 도입하게 되었다.

### Refresh Token Rotation이란?

Refresh Token Rotation은 보안성을 높이기 위해 **리프레시 토큰을 한 번만 사용할 수 있도록 제한**하는 기술이다. 사용자가 리프레시 토큰을 사용해 새로운 액세스 토큰을 요청하면, 서버는 기존 리프레시 토큰을 폐기하고 새 리프레시 토큰을 발급한다. 이를 통해 만약 리프레시 토큰이 유출되더라도 반복 사용이 불가능해져 보안 위협을 줄일 수 있다.

**주요 동작 과정**

1. 클라이언트는 서버로부터 **액세스 토큰**과 **리프레시 토큰**을 발급받는다.
2. 액세스 토큰이 만료되면 리프레시 토큰을 사용해 **새 액세스 토큰을 요청**한다.
3. 서버는 **새 액세스 토큰**과 **새 리프레시 토큰**을 발급하며, **기존 리프레시 토큰은 폐기**한다.
4. 클라이언트는 이후 새로 발급된 리프레시 토큰만 사용할 수 있다.

## Refresh Token Rotation 구현

### 주요 메커니즘

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Redis

    Client->>Server: 토큰 재발급 요청
    Note over Client,Server: 기존 refresh token 전달

    Server->>Redis: refresh token 검증
    Note over Server, Redis: 토큰 재사용 여부 확인

    Server->>Redis: 기존 refresh token 무효화
    Note over Server, Redis: 해당 유저의 토큰 정보를 업데이트

    Server-->>Client: 새로운 토큰 발급 (access, refresh)

    Client->Client: 토큰 업데이트
```

### 리프레시 토큰 관리: PostgreSQL vs Redis

리프레시 토큰이 재사용되었는지를 확인하기 위해 최신 리프레시 토큰을 관리해야 했다. 우선 기존에 사용 중인 PostgreSQL의 `meeting_user` 테이블에 `refresh_token` 컬럼을 추가하는 방안을 검토했다. 이 방법은 새로운 인프라 구축 없이 기존 시스템을 활용할 수 있다는 장점이 있었다.

**PostgreSQL 사용시 문제점**

하지만 검토 과정에서 PostgreSQL을 사용하는 것에는 다음과 같은 문제가 있었다.

1. **데이터베이스 부하 증가:** 리프레시 토큰은 빈번한 읽기/쓰기가 발생하는 데이터다. 이런 단기 데이터의 잦은 갱신은 전체 데이터베이스 성능에 부담이 될 수 있었다.
2. **불필요한 트랜잭션 오버헤드:** PostgreSQL은 ACID 특성을 보장하기 위해 모든 작업을 트랜잭션으로 처리한다. 단순 키-값 형태인 토큰 데이터에 이러한 무거운 트랜잭션 처리는 불필요한 오버헤드를 발생시킨다.
3. **단기 데이터 관리의 비효율성:** PostgreSQL은 장기 데이터 관리에 최적화되어 있다. 반면 리프레시 토큰은 짧은 생명주기를 가진 데이터로, 이를 효율적으로 관리하기 위해서는 추가적인 만료 데이터 정리 작업이 필요했다.
4. **테이블 설계의 복잡성:** `meeting_user` 테이블은 유저 정보 관리가 주 목적이다. 여기에 토큰 관리 기능을 추가하면 테이블의 책임이 모호해지고 데이터 모델이 복잡해진다.

**Redis를 선택한 이유**

따라서 **Redis를 토큰 저장소**로 선택했다. Redis는 다음과 같은 이점을 제공한다.

1. **빠른 데이터 접근:** 인메모리 데이터베이스인 Redis는 디스크 I/O가 없어 토큰 검증과 갱신에서 탁월한 성능을 보장한다.
2. **TTL 기능 활용:** Redis의 기본 기능인 TTL을 통해 만료된 토큰을 자동으로 제거할 수 있다. 이는 PostgreSQL에서 필요했던 별도의 정리 작업을 불필요하게 만든다.
3. **시스템 역할 분리:** Redis는 토큰과 같은 임시 데이터를 처리하고, PostgreSQL은 사용자 정보와 같은 영구 데이터를 관리하도록 역할을 명확히 분리할 수 있다.
4. **확장성 확보:** Redis 클러스터를 통해 사용자 증가나 트래픽 급증에도 유연하게 대응할 수 있다.

### 주요 기능

**토큰 재발급**

토큰을 재발급 하기 전에 Redis에 저장된 리프레시 토큰을 통해 유효성을 검증하였고 생성된 토큰을 Redis에 저장하였다.

- `AuthService.kt`
    
  ```kotlin
  fun reissueTokens(request: HttpServletRequest, response: HttpServletResponse): JwtResponse {
      val requestInfo = requestUtils.toRequestInfoDto(request)
      val refreshToken =
          cookieUtils.getRefreshTokenFromCookie(request)
              ?: run {
                  logger.warn("[재발급 실패(토큰없음)] $requestInfo")
                  throw JwtRefreshTokenNotFoundException()
              }
      try {
          val userId = jwtTokenProvider.getUserIdFromRefreshToken(refreshToken)
  
          val storedToken = jwtTokenProvider.getStoredRefreshToken(userId)
          if (storedToken != refreshToken) {
              logger.warn("[재발급 실패(재사용)] $requestInfo")
              throw JwtRefreshTokenReusedException()
          }
  
          val newAccessToken = jwtTokenProvider.createAccessToken(userId)
          val newRefreshToken = jwtTokenProvider.createRefreshToken(userId)
  
          jwtTokenProvider.saveRefreshToken(userId, newRefreshToken)
  
          cookieUtils.addRefreshTokenCookie(response, newRefreshToken, refreshTokenExpiration)
          logger.info("[재발급 성공] userId: $userId")
          return JwtResponse(newAccessToken)
      } catch (e: ExpiredJwtException) {
          logger.warn("[재발급 실패(만료)] $requestInfo")
          throw JwtRefreshTokenExpiredException()
      } catch (e: JwtException) {
          logger.warn("[재발급 실패(서명)] $requestInfo")
          throw JwtTokenInvalidSignatureException()
      }
  }
  ```
    

**로그아웃**

유저가 로그아웃을 요청하면 Redis에 저장된 리프레시 토큰을 함께 삭제하였다.

- `AuthService.kt`
    
  ```kotlin
  fun logout(request: HttpServletRequest, response: HttpServletResponse) {
      val requestInfo = requestUtils.toRequestInfoDto(request)
  
      val refreshToken = cookieUtils.getRefreshTokenFromCookie(request)
      if (refreshToken != null) {
          try {
              val userId = jwtTokenProvider.getUserIdFromRefreshToken(refreshToken)
              jwtTokenProvider.deleteRefreshToken(userId)
          } catch (e: JwtException) {
              logger.warn("[로그아웃 요청] 유효하지 않은 리프레시 토큰 사용 $requestInfo")
          }
      } else {
          logger.warn("[로그아웃 요청] 리프레시 토큰 없음 $requestInfo")
      }
      cookieUtils.deleteRefreshTokenCookie(response)
  }
  ```
    

### 기타 기능

**리프레시 토큰 저장, 조회, 삭제**

- `JwtTokenStore.kt`

```kotlin
@Component
class JwtTokenStore(
    private val redisTemplate: RedisTemplate<String, Any>,
    @Value("\${jwt.refresh.expiration}") private val refreshTokenExpiration: Long,
) {
  fun saveRefreshToken(userId: Long, refreshToken: String) {
      val key = "${SecurityConstants.REFRESH_TOKEN_PREFIX}:$userId"
      redisTemplate
          .opsForValue()
          .set(key, encryptedToken, refreshTokenExpiration, TimeUnit.MILLISECONDS)
  }
  
  fun getStoredRefreshToken(userId: Long): String? {
      val key = "${SecurityConstants.REFRESH_TOKEN_PREFIX}:$userId"
      val token = redisTemplate.opsForValue().get(key)?.toString() ?: return null
      return token
  }
  
  fun deleteRefreshToken(userId: Long) {
      val key = "${SecurityConstants.REFRESH_TOKEN_PREFIX}:$userId"
      redisTemplate.delete(key)
  }
}
```

```
127.0.0.1:6379> get "refresh_token:1"
"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwiaXNzIjoiVU9TTElGRSIsImF1ZCI6WyJVT1NMSUZFIFVTRVIiXSwiaWF0IjoxNzMyOTQ3NzY2LCJleHAiOjE3MzM1NTI1NjZ9.V7Cy4fbB2KrLGWo0I6hRZbyDifFmYDkwXMfyZ1er2Ys"
127.0.0.1:6379> get "refresh_token:1"
"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwiaXNzIjoiVU9TTElGRSIsImF1ZCI6WyJVT1NMSUZFIFVTRVIiXSwiaWF0IjoxNzMyOTQ4MDExLCJleHAiOjE3MzM1NTI4MTF9.W3y7Sd6o2Bmb5g2rDRBDA2sEL4aCHoilyMi9cK5uoZU"
```

**토큰 암호화 (AES)**

- `JwtTokenStore.kt`

```kotlin
@Component
class JwtTokenStore(
    @Value("\${jwt.encryption.aes-secret-key}") private val aesSecretKey: String,
) {
    private val cipher: Cipher = Cipher.getInstance("AES")
    private val encryptionKey: SecretKey = SecretKeySpec(aesSecretKey.toByteArray(), "AES")

    private fun encrypt(text: String): String {
        cipher.init(Cipher.ENCRYPT_MODE, encryptionKey)
        val encryptedBytes = cipher.doFinal(text.toByteArray())
        return Base64.getEncoder().encodeToString(encryptedBytes)
    }

    private fun decrypt(encryptedText: String): String {
        cipher.init(Cipher.DECRYPT_MODE, encryptionKey)
        val decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(encryptedText))
        return String(decryptedBytes)
    }
```

```
127.0.0.1:6379> get "refresh_token:1"
"mFAqdvxjoD8GWb4EPsCKOMkkpPUCt14ryw/8sV32GebLCzyCwZPUt+08l+nYja/rwPJpR8XZw7AbN2KcnwVT3ZYr7Fphhf3S389DuVFpjePAgEThF6UsogL19f8zaZ8RnO7z9t9KXnmXsAgIOBrv4ckWqvXPtLmgb3PMFuO9T8RfOSC0QmNFXruY8RsfeiQacjTlJ0cbKjkBV4TgJYgvfGkrE38lIN9nTDZBA6u14V0CMuuuaiH1nnwk3JogJCJJ"
```

### 구조 개선

**책임 분리**

토큰 관련 로직의 복잡도가 증가함에 따라 다음과 같이 책임을 분리했다.

- `JwtTokenGenerator`: 토큰 생성 담당
- `JwtTokenParser`: 토큰 검증 및 파싱
- `JwtTokenStore`: Redis 저장소 관리
- `JwtTokenProvider`: Facade 패턴으로 전체 기능 통합

```kotlin
@Component
class JwtTokenProvider(
    private val tokenGenerator: JwtTokenGenerator,
    private val tokenParser: JwtTokenParser,
    private val tokenStore: JwtTokenStore,
) {
    // Token 생성 관련 메서드
    fun createAccessToken(id: Long) = tokenGenerator.createAccessToken(id)
    fun createRefreshToken(id: Long) = tokenGenerator.createRefreshToken(id)

    // Token 검증 및 파싱 관련 메서드
    fun validateAccessToken(token: String) = tokenParser.validateAccessToken(token)
    fun validateRefreshToken(token: String) = tokenParser.validateRefreshToken(token)
    fun getUserIdFromAccessToken(token: String) = tokenParser.getUserIdFromAccessToken(token)
    fun getUserIdFromRefreshToken(token: String) = tokenParser.getUserIdFromRefreshToken(token)

    // Redis 저장소 관련 메서드
    fun saveRefreshToken(userId: Long, refreshToken: String) = tokenStore.saveRefreshToken(userId, refreshToken)
    fun getStoredRefreshToken(userId: Long) = tokenStore.getStoredRefreshToken(userId)
    fun deleteRefreshToken(userId: Long) = tokenStore.deleteRefreshToken(userId)
}
```

## 결론

Refresh Token Rotation의 도입으로 토큰 재사용으로 인한 보안 위험을 줄일 수 있었다. 또한 코드 구조 개선을 통해 유지보수성과 확장성도 함께 향상시켰다.

## 참고자료

- [Refreshing Access Tokens - OAuth 2.0 Simplified](https://www.oauth.com/oauth2-servers/access-tokens/refreshing-access-tokens/)
- [What Are Refresh Tokens and How to Use Them Securely \| Auth0](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)
- [OAuth 2.0 Refresh Token Rotation - Auth0](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [JWT Best Practices - IETF](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-jwt-bcp)
- [authentication - Should we renew refresh tokens along with access tokens? - Stack Overflow](https://stackoverflow.com/questions/67958824/should-we-renew-refresh-tokens-along-with-access-tokens)
- [Kinde \| How to use refresh tokens](https://kinde.com/guides/authentication/types-and-methods/how-to-use-refresh-tokens/)
- [[인증/인가] RefreshToken은 왜 Redis를 사용해 관리할까? (with. RTR 방식)](https://pgmjun.tistory.com/125)
- [[DB] 관계형 데이터베이스(RDBMS) 구조 : DDL, DML, DCL, TCL — Contributor9](https://adjh54.tistory.com/314)
- [Redis 클러스터](https://kjw1313.tistory.com/118)