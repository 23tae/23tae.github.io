---
title: "시대팅 이메일 전송 횟수 초기화용 어드민 API 구현"
date: 2024-12-21T11:00:00.000Z
categories: [Project, 시대팅5]
tags: [spring-boot]
---

## 배경

![email cs](/assets/img/project/sidaeting5/05-admin-api/kakaotalk-cs-email.png)
_인증 횟수 초기화 요청_

이메일 인증 시스템에서는 일일 인증 횟수를 5회로 제한하고 있다. 하지만 이러한 제한은 정상적인 사용자에게도 영향을 미칠 수 있어 CS팀이 필요한 경우 사용자의 인증 횟수를 초기화할 수 있는 기능이 필요했다.

처음에는 DataGrip이나 Redis CLI 등을 사용해 **Redis 데이터에 직접 접근**하여 값을 수정하는 방안을 고려했다. 하지만 이 방법은 다음과 같은 문제가 있었다.

- 데이터베이스 직접 접근에 따른 보안 위험
- 작업 내역 추적 불가능
- 실수로 인한 데이터 손상 가능성

## 어드민 API

### API 설계

우선 **관리자 인증 방식**에 대해 다음과 같은 논의가 있었다.

1. 별도의 인증 없이 사용하는 방법
2. **Request Body**로 관리자 비밀번호를 전달하는 방법
3. **Request Header**로 API Key를 전달하는 방법

첫 번째 방법은 별도의 인증 없이 누구나 API에 접근할 수 있어 보안상 부적절하다고 판단하여 제외했다. 나머지 방법 중에서 논의 결과, 아래와 같은 이유로 **Header에 API Key를 포함**하는 세 번째 방법을 채택했다.

- REST API의 표준적인 인증 방식을 준수한다.
- 매 요청마다 비밀번호를 입력할 필요가 없다.
- 인터셉터를 통해 인증 로직을 분리하기가 용이하다.

이후 **해당 API를 관리할 위치**를 고민하였다. 처음에는 이메일 인증을 관련 API가 모인 Verfication 컨트롤러에 인증 초기화 API를 위치시키려고 했다. 하지만 **접근 제어**와 **확장성** 측면에서 별도의 Admin 컨트롤러를 두는 것이 좋을 것이라고 판단하였다. 실제로 이후에 어드민 API에 유저 삭제, 환불, 캐시 워밍과 같이 다양한 관리자용 API가 추가되었다.

### 설정

- `.env`
  ```
  ADMIN_API_KEY=your_api_key
  ```

- `application.yml`
  ```yaml
  api:
    admin:
      key: ${ADMIN_API_KEY}
  ```

- `AdminApiKeyInterceptor.kt`
  ```kotlin
  class AdminApiKeyInterceptor(private val adminApiKey: String) : HandlerInterceptor {
      companion object {
          private const val API_KEY_HEADER = "X-API-Key"
      }

      override fun preHandle(
          request: HttpServletRequest,
          response: HttpServletResponse,
          handler: Any
      ): Boolean {
          val apiKey = request.getHeader(API_KEY_HEADER) ?: throw ApiKeyNotFoundException()

          if (apiKey != adminApiKey) {
              throw ApiKeyInvalidException()
          }
          return true
      }
  }
  ```

- `WebConfig.kt`
  ```kotlin
  @Configuration
  class WebConfig : WebMvcConfigurer {
      @Value("\${api.admin.key}") private lateinit var adminApiKey: String

      override fun addInterceptors(registry: InterceptorRegistry) {
          registry
              .addInterceptor(AdminApiKeyInterceptor(adminApiKey))
              .addPathPatterns("/api/admin/**")
      }
  }
  ```

- `SecurityConfig.kt`
  ```kotlin
  @Configuration
  @EnableWebSecurity
  class SecurityConfig(
      private val jwtAuthenticationFilter: JwtAuthenticationFilter,
      @Qualifier("handlerExceptionResolver") private val resolver: HandlerExceptionResolver,
  ) {
      @Bean
      @Throws(Exception::class)
      fun filterChain(http: HttpSecurity): SecurityFilterChain? {
          http.authorizeHttpRequests {
                  it.requestMatchers(
                          ...
                          "/api/admin/**"
                      ) // 토큰 검사 미실시 리스트
                      .permitAll()
              }

          http.addFilterBefore(
              jwtAuthenticationFilter,
              UsernamePasswordAuthenticationFilter::class.java
          )
          return http.build()
      }
  ```

### API 구현

- `AdminApi.kt`
  ```kotlin
  @RestController
  @RequestMapping("/api/admin")
  class AdminApi(
      private val adminService: AdminService,
      private val requestUtils: RequestUtils,
  ) {

      @PostMapping("/verification/send-email/reset")
      fun resetEmailSendCount(
          request: HttpServletRequest,
          @RequestBody body: ResetEmailRequest
      ): ResponseEntity<Unit> {
          val requestInfo = requestUtils.toRequestInfoDto(request)

          adminService.resetEmailSendCount(body.email, requestInfo)

          return ResponseEntity.status(HttpStatus.NO_CONTENT).build()
      }
  }
  ```

- `AdminService.kt`
  ```kotlin
  @Service
  class AdminService(
      private val redisTemplate: RedisTemplate<String, Any>,
  ) {
      fun resetEmailSendCount(email: String, requestInfo: RequestInfoDto) {
          val sendCountKey =
              VerificationUtils.generateRedisKey(VerificationConstants.SEND_COUNT_PREFIX, email, true)

          redisTemplate.opsForValue().set(sendCountKey, "0")
          redisTemplate.expire(sendCountKey, Duration.ofDays(1))

          logger.info("[ADMIN-이메일 발송 횟수 초기화] targetEmail: $email, $requestInfo")
      }
  }
  ```

### 보안 고려사항

1. **API Key는 환경 변수로 관리**하여 코드에 노출되지 않도록 하였다.
2. 모든 관리자 작업에 대해 **로깅**을 하여 추적성을 확보하였다.
  ```kotlin
  logger.info("[ADMIN-이메일 발송 횟수 초기화] targetEmail: $email, $requestInfo")
  ```
  ```
  2024-12-13T00:58:24.037+09:00  INFO 96508 --- [nio-8081-exec-3] u.s.admin.service.AdminService: [ADMIN-이메일 발송 횟수 초기화] targetEmail: example@uos.ac.kr, ip: 0:0:0:0:0:0:0:1, userAgent: PostmanRuntime/7.43.0
  ```

## 참고자료

- [[JAVA] Spring으로 REST API 구현하기 (2) - Interceptor](https://heodolf.tistory.com/40)
