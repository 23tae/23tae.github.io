---
title: "시대팅5 이메일 인증 기능 구현하기 with Amazon SES"
date: 2024-12-15T09:00:00.000Z
categories: [Project, 시대팅5]
tags: [spring-boot, aws, redis]
mermaid: true
---

|![email verification page 1](/assets/img/project/sidaeting5/02-email-verification/email-verification-page-1.png)|![email verification page 2](/assets/img/project/sidaeting5/02-email-verification/email-verification-page-2.png)|![email verification page 3](/assets/img/project/sidaeting5/02-email-verification/email-verification-page-3.png)|

이 글에서는 시대팅을 개발하며 AWS SDK를 이용한 Amazon SES 이메일 발송, Redis를 이용한 인증코드 관리, 사용량 제한 등의 이메일 인증 기능을 구현한 과정을 다룬다.

## 배경

기존 시대생 앱에서는 어카운트 서버를 사용한 **포털 인증**으로 유저가 재학생임을 확인하였다. 하지만 이번 프로젝트는 **어카운트 서버를 사용하지 않기로 결정**하였기 때문에 **별도의 재학생 인증 기능을 구현**해야 했다. 또한 올해 상반기에 포털이 개편되면서 포털 인증을 할 수 없게 되었다. 따라서 교내 웹메일 인증을 통해 재학생임을 확인하기로 했다.

## 요구사항 분석

**이메일 인증 기능의 주요 요구사항**은 다음과 같다.

- 사용자가 입력한 이메일로 인증코드를 발송한다.
- 사용자가 인증코드를 입력해 유효성을 검증한다.
- 이메일 주소는 반드시 `@uos.ac.kr` 도메인을 사용해야 한다.
- 인증코드는 일정 시간(10분)이 지나면 만료되어야 한다.
- 일일 이메일 발송 횟수(5회)와 인증코드 검증 시도 횟수(5회)를 제한한다.

## 기능 설계

1. **사용자 요청 흐름**:
    - **인증 메일 발송**
        - 사용자가 이메일 인증을 요청하면 서버에서 인증코드를 생성해 Redis에 저장한다.
        - 이후 Amazon SES를 이용해 사용자가 입력한 이메일로 인증코드를 포함한 메일을 발송한다.
    - **인증 코드 검증**
        - 사용자가 인증코드를 입력하면 서버가 Redis에서 저장된 인증코드를 조회해 유효성을 검증한다.
        - 인증에 성공하면 Redis에서 관련 데이터를 삭제한다.
2. **주요 기술 스택**:
    - **Amazon SES**: 인증 메일 발송용 서비스.
    - **Redis**: 인증코드 및 사용량 데이터 관리.

### Amazon SES (Simple Email Service)

![ses logo](/assets/img/project/sidaeting5/02-email-verification/amazon-ses-logo.png){: width="70%" style="display: block; margin: auto; "}

**주요 기능**

1. 이메일 발송
- HTML/텍스트 형식 지원
- 템플릿 기반 발송
- 첨부 파일 지원

2. 보안 및 모니터링
- SPF, DKIM, DMARC 인증 프로토콜 지원
- 발송 지표 실시간 추적
- CloudWatch 통합 모니터링

**인터페이스**

  1. API 인터페이스
  - HTTPS 프로토콜로 RESTful API 호출
  - AWS SDK나 IAM 인증 사용
  - JSON 형식의 데이터 교환

  2. SMTP 인터페이스
  - SMTP 프로토콜 사용
  - 기존 메일 시스템과 호환성이 높음
  - 표준 메일 전송 프로토콜로 구현이 단순함

**선택한 이유**

- **안정성과 확장성**: AWS 인프라를 기반으로 대규모 이메일 발송에도 안정적으로 대응할 수 있다.
- **저비용 구조**: 이메일 발송 비용이 1000건당 0.1USD로 저렴하며 하루 최대 5만건까지 발송이 가능하다.
- **코드 통합**: AWS SDK로 코드에 쉽게 통합할 수 있어 개발 생산성을 높일 수 있다.
- **관리 기능**: AWS 콘솔 및 CloudWatch를 활용하여 상세한 모니터링과 관리가 가능하다.

### Redis

![redis logo](/assets/img/project/sidaeting5/02-email-verification/redis-logo.jpg){: width="70%" style="display: block; margin: auto; "}

**특징**

  1. 주요 기능
  - TTL 기반의 데이터 만료 처리
  - RDB/AOF 방식의 영속성 지원
  - Pub/Sub 메시징 시스템
  - 클러스터 모드 지원

  2. 데이터 구조
  - String, List, Hash 등 다양한 데이터 타입 지원
  - 각 데이터 타입별 최적화된 연산 제공
  - 원자적 작업 처리 가능

  3. 성능
  - 인메모리 기반의 고속 데이터 처리
  - 단일 스레드 작업으로 데이터 일관성 보장
  - 데이터 복제를 통한 가용성 확보

**선택한 이유**

- **속도와 효율성**: 인증코드와 같은 **휘발성 데이터**를 빠르게 읽고 쓸 수 있다.
- **TTL 지원**: **인증코드 만료 시간**을 손쉽게 설정하고 관리할 수 있다.
- **단순한 키-값 구조**: 이메일 **발송 횟수**와 **인증 시도 횟수**를 관리하기에 적합하다.

**키 설계**

- `email_verification_code:{email}`: 인증 코드 저장 및 TTL 설정.
- `email_send_count:{email}:{date}`: 일일 발송 횟수 관리.
- `verification_attempts:{email}:{date}`: 일일 인증 시도 횟수 관리.

### 플로우차트

**인증 메일 전송**

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    participant R as Redis
    participant SES

    C->>S: 인증 메일 요청
    S->>R: 발송 제한 확인
    S->>SES: 인증 메일 발송
    SES-->>S: 발송 결과 전달
    S->S: 발송 이력 기록
    S-->>C: 인증 만료시간 반환
```

**인증 코드 검증**

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    participant R as Redis
    participant P as PostgreSQL

    C->>S: 인증 코드 검증 요청
    S->>R: 코드 검증
    alt 검증 성공
        S->S: 성공 이력 기록
        S->>P: 유저 조회 및 생성
        P-->>S: 유저 ID 반환
        S->>S: 토큰 생성
        S-->>C: 토큰 반환
    else 검증 실패
        S->S: 실패 이력 기록
        S-->>C: 에러코드 반환
    end
```

## 주요 기능 구현

### Amazon SES 설정

![aws-ses-configuration](/assets/img/project/sidaeting5/02-email-verification/aws-ses-configuration.png)
_AWS 콘솔_

AWS 콘솔에서 SES를 이용하는데 필요한 기본적인 설정은 이미 되어 있었기 때문에 콘솔 상에서는 IAM 사용자 생성 외에 추가적인 설정은 하지 않았다.

- `build.gradle.kts`

  ```kotlin
  dependencies {
      implementation("com.amazonaws:aws-java-sdk-ses:1.12.777")
  }
  ```

- `application.yml`

  ```yaml
  aws:
      access-key-id: ${AWS_SES_ACCESS_KEY}
      secret-access-key: ${AWS_SES_SECRET_KEY}
      region: ap-northeast-2
      ses:
          email:
              title: UOSLIFE 인증 메일입니다.
              from: 시대생팀 <no-reply@uoslife.team>
  ```

- `AwsSesConfig.kt`

  ```kotlin
  import com.amazonaws.auth.AWSStaticCredentialsProvider
  import com.amazonaws.auth.BasicAWSCredentials
  import com.amazonaws.services.simpleemail.AmazonSimpleEmailService
  import com.amazonaws.services.simpleemail.AmazonSimpleEmailServiceClientBuilder

  @Configuration
  class AwsSesConfig(
      @Value("\${aws.access-key-id}") private val accessKeyId: String,
      @Value("\${aws.secret-access-key}") private val secretAccessKey: String,
      @Value("\${aws.region}") private val region: String
  ) {

      @Bean
      fun amazonSimpleEmailService(): AmazonSimpleEmailService {
          val awsCredentials = createAwsCredentials()
          return AmazonSimpleEmailServiceClientBuilder.standard()
              .withCredentials(AWSStaticCredentialsProvider(awsCredentials))
              .withRegion(region)
              .build()
      }

      private fun createAwsCredentials(): BasicAWSCredentials {
          return BasicAWSCredentials(accessKeyId, secretAccessKey)
      }
  }
  ```

### Redis 설정

- `build.gradle.kts`

  ```kotlin
  dependencies {
      implementation("org.springframework.boot:spring-boot-starter-data-redis")
  }
  ```

- `RedisConfig.kt`

  ```kotlin
  import org.springframework.data.redis.connection.RedisConnectionFactory
  import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory
  import org.springframework.data.redis.core.RedisTemplate
  import org.springframework.data.redis.serializer.StringRedisSerializer

  @Configuration
  class RedisConfig(
      @Value("\${spring.data.redis.host}") private val redisHost: String,
      @Value("\${spring.data.redis.port}") private val redisPort: Int,
  ) {

      @Bean
      fun redisConnectionFactory(): RedisConnectionFactory {
          return LettuceConnectionFactory(redisHost, redisPort)
      }

      @Bean
      fun redisTemplate(): RedisTemplate<String, Any> {
          return RedisTemplate<String, Any>().apply {
              setConnectionFactory(redisConnectionFactory())
              keySerializer = StringRedisSerializer()
              valueSerializer = StringRedisSerializer()
              hashKeySerializer = StringRedisSerializer()
              hashValueSerializer = StringRedisSerializer()
          }
      }
  }
  ```

### 인증 메일 발송

![image.png](/assets/img/project/sidaeting5/02-email-verification/verification-mail.png)

- **API 엔드포인트**: `POST /api/verification/send-email`
- **구현 내용**:
    - 사용자가 입력한 이메일의 형식을 검증하고 `@uos.ac.kr` 도메인 여부를 확인한다.
    - **인증코드를 생성**하여 Redis에 저장하고 만료 시간을 설정한다.
    - AWS SDK를 사용하여 사용자에게 인증코드가 포함된 **메일을 발송**한다.
    - **일일 발송 횟수**를 Redis를 통해 관리하고, 초과 시 에러를 반환한다.

**관련 코드**

템플릿(`resources/templates/email-template.html`)을 활용해서 이메일을 전송하도록 하였다.

```kotlin
import com.amazonaws.services.simpleemail.AmazonSimpleEmailService
import com.amazonaws.services.simpleemail.model.*

@Service
class EmailVerificationService(
) {
  fun sendVerificationEmail(email: String): SendVerificationEmailResponse {
      validateEmail(email) // 이메일 형식 검증
      validateSendCount(email) // 발송 제한 확인
      val verificationCode = generateVerificationCode() // 인증 코드 생성
      saveVerificationCode(email, verificationCode) // 인증 코드 저장
      incrementSendCount(email) // 발송 횟수 증가
      sendEmail(email, verificationCode) // 이메일 전송
      val expirationTime = calculateExpirationTime() // 코드 만료 시각 계산

      return SendVerificationEmailResponse(
          expirationTime = expirationTime,
          validDuration = codeExpiry
      )
  }

  private fun sendEmail(email: String, verificationCode: String) {
      val codeExpiryMinutes = codeExpiry / 60
      val context = Context()
      context.setVariable("expiryMinutes", codeExpiryMinutes)
      context.setVariable("verificationCode", verificationCode)

      val emailBody = templateEngine.process("email-template", context)

      val request =
          SendEmailRequest()
              .withDestination(Destination().withToAddresses(email))
              .withMessage(
                  Message()
                      .withSubject(Content(emailTitle))
                      .withBody(Body().withHtml(Content(emailBody)))
              )
              .withSource(emailFrom)
      sesClient.sendEmail(request)
  }
}
```
    
- 응답 예시
    
  ```json
  {
      "expirationTime": 1734123014092,
      "validDuration": 600
  }
  ```
    

### 인증 코드 검증

- **API 엔드포인트**: `POST /api/verification/verify-email`
- **구현 내용**:
    - 사용자가 입력한 이메일과 인증코드를 검증한다.
    - Redis에서 해당 이메일의 인증코드를 조회하고 만료 여부를 확인한다.
    - 검증 성공 시 유저를 생성(등록되지 않은 유저인 경우)한 뒤 토큰을 반환한다.
    - 검증 실패 시 에러를 반환한다.

**관련 코드**

- `VerificationApi.kt`

  ```kotlin
  @PostMapping("/verify-email")
  fun verifyEmail(
      @Valid @RequestBody body: VerifyEmailRequest,
      request: HttpServletRequest,
      response: HttpServletResponse
  ): ResponseEntity<JwtResponse> {
      val requestInfo = requestUtils.toRequestInfoDto(request)
      emailVerificationService.verifyEmail(body.email, body.code, requestInfo)

      val user = userService.getOrCreateUser(body.email)

      val accessToken = authService.issueTokens(user.id!!, response)

      return ResponseEntity.ok(accessToken)
  }
  ```

- `VerificationService.kt`

  ```kotlin
  fun verifyEmail(email: String, code: String) {
      // 인증 횟수 확인
      validateVerificationAttempts(email)
      incrementVerificationAttempts(email)

      // 인증 코드 검증
      val redisCode = getVerificationCode(email)
      validateVerificationCode(redisCode, code)

      // 인증 성공한 코드 삭제
      clearVerificationData(email)
  }
  ```

### 사용량 제한 구현

**일일 발송 횟수 제한**

- Redis 키에 일일 발송 횟수를 저장하고 하루 뒤 만료되도록 설정하였다.
    
  ```kotlin
  private fun incrementSendCount(email: String) {
      val sendCountKey = VerificationUtils.generateRedisKey(VerificationConstants.SEND_COUNT_PREFIX, email, true)
      redisTemplate.opsForValue().increment(sendCountKey)
      redisTemplate.expire(sendCountKey, Duration.ofDays(1))
  }
  ```
    
- 예시
    
  ```
  127.0.0.1:6379> keys *
  1) "email_send_count:example@uos.ac.kr:20241212"
  ```
    
- 이후 이메일 전송 요청을 받으면 Redis에서 발송 횟수를 확인하였다.
    
  ```kotlin
  private fun validateSendCount(email: String) {
      val sendCountKey = generateRedisKey(SEND_COUNT_PREFIX, email, true)
      val currentCount = redisTemplate.opsForValue().get(sendCountKey)?.toString()?.toInt() ?: 0
      if (currentCount >= dailySendLimit) {
          throw DailyEmailSendLimitExceededException()
      }
  }
  ```
    

**일일 인증 시도 횟수 제한**

- 동일한 방식으로 인증 시도 횟수를 관리하였다.
- Redis 키에 일일 인증 시도 횟수를 저장하고 제한 초과 시 예외를 발생시키도록 하였다.
    
  ```kotlin
  private fun validateVerificationAttempts(email: String) {
      val attemptsKey =
          VerificationUtils.generateRedisKey(
              VerificationConstants.VERIFY_COUNT_PREFIX,
              email,
              true
          )
      val attempts = redisTemplate.opsForValue().get(attemptsKey)?.toString()?.toInt() ?: 0
      if (attempts >= codeVerifyLimit) {
          throw DailyVerificationAttemptLimitExceededException()
      }
  }
  ```
    

### 에러 처리

|![email error 1](/assets/img/project/sidaeting5/02-email-verification/email-error-1.png)|![email error 2](/assets/img/project/sidaeting5/02-email-verification/email-error-2.png)|![email error 3](/assets/img/project/sidaeting5/02-email-verification/email-error-3.png)|![email error 4](/assets/img/project/sidaeting5/02-email-verification/email-error-4.png)|


- 예외 케이스
    - **이메일 형식**이 잘못된 경우.
    - 이메일이 **대학 도메인**이 아닌 경우.
    - **일일 발송 한도**를 초과한 경우.
    - 인증 코드가 **일치하지 않는** 경우.
    - 인증코드가 **만료**된 경우.
    - **일일 인증 한도**를 초과한 경우.
- 에러코드 설정

  ```kotlin
  EMAIL_INVALID_FORMAT("E01", "Invalid email format.", HttpStatus.BAD_REQUEST.value()),
  EMAIL_INVALID_DOMAIN("E02", "Email domain is not allowed.", HttpStatus.BAD_REQUEST.value()),
  EMAIL_VERIFICATION_CODE_MISMATCH("E03", "Verification code does not match.", HttpStatus.BAD_REQUEST.value()),
  EMAIL_DAILY_SEND_LIMIT_EXCEEDED("E04", "Daily email send limit exceeded.", HttpStatus.TOO_MANY_REQUESTS.value()),
  EMAIL_DAILY_VERIFY_LIMIT_EXCEEDED("E05", "Daily verification attempt limit exceeded.", HttpStatus.TOO_MANY_REQUESTS.value()),
  EMAIL_SEND_FAILED("E06", "Failed to send email.", HttpStatus.INTERNAL_SERVER_ERROR.value()),
  EMAIL_VERIFICATION_CODE_EXPIRED("E06", "Verification code expired.", HttpStatus.BAD_REQUEST.value()),
  ```
    
- 예외 클래스 예시

  ```kotlin
  class DailyEmailSendLimitExceededException :
      EmailLimitExceededException(ErrorCode.EMAIL_DAILY_SEND_LIMIT_EXCEEDED)
  ```
    

### 개발 중 겪은 문제

- **인증 메일 요청 시간 문제**: 비동기 처리를 적용하여 개선하였다. (다음 글에서 다룰 예정)
- 다양한 케이스의 예외를 구분하지 않음: 인증코드가 일치하지 않는 경우와 만료된 경우 등의 예외처리를 추가하였다.

## 마치며

![email usage](/assets/img/project/sidaeting5/02-email-verification/email-usage.png)
_서비스 기간 일일 이메일 사용량_

이메일 인증 기능은 이번 시대팅 개발에서 제일 먼저 구현했던 기능이다. 이메일 전송 기능 자체는 SES API를 사용하여 간단하게 구현이 가능했지만 인증 코드 검증, 인증 제한 관리 등을 구현하는 과정에서 했던 다양한 고민이 의미있었던 것 같다.

### 추가 개선 방향

현재의 시스템은 보안성에 있어 개선의 필요성이 있다. 우선 인증코드의 복잡성을 높이고 암호화하여 저장하는 방식을 도입할 수 있다. 또한 인증코드를 직접 입력하는 대신 인증 링크를 통해 이메일을 확인하는 방식도 고려해볼 만하다.

더불어 **악성 유저에 대한 대응책**도 강화할 필요가 있다. 현재는 **이메일 주소**별로 발송 횟수를 제한하고 있어 동일한 이메일로의 과도한 요청은 차단할 수 있다. 하지만 악의적인 사용자가 타인의(혹은 존재하지 않는) 이메일로 인증 요청을 보내는 경우는 차단하지 못한다.

이를 해결하기 위해 프로젝트 초기에 **IP 주소 기반**의 제한을 고려하였으나, 교내 와이파이 네트워크의 특성상 다수의 학생이 동일한 IP를 사용하는 상황에서는 유저를 구별하기 힘들고 정상적인 사용자들의 이용에도 불편이 생길 수 있다는 문제가 있어 도입하지 않기로 했었다. 하지만 **IP 주소와 User Agent 정보를 조합**한다면 유저를 충분히 구별할 수 있을 것이라 생각한다. 이를 통해 교내 네트워크 사용자들의 정상적인 이용은 보장하면서도 악성 유저의 비정상적인 요청 패턴을 감지하고 차단할 수 있을 것이다.

## 참고자료

- [Using the Amazon SES API to send email - Amazon Simple Email Service](https://docs.aws.amazon.com/ses/latest/dg/send-email-api.html)
- [Using the Amazon SES SMTP interface to send email - Amazon Simple Email Service](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
- [Redis - Wikipedia](https://en.wikipedia.org/wiki/Redis)
- [Sending email through Amazon SES using an AWS SDK - Amazon Simple Email Service](https://docs.aws.amazon.com/ses/latest/dg/send-an-email-using-sdk-programmatically.html)
- [[Django] AWS SES로 이메일 보내기](https://velog.io/@hnnynh/AWS-SES로-django에서-이메일-인증-코드-보내기)
- [Spring - Redis를 사용해보자 — 개발하는 콩](https://green-bin.tistory.com/69)
- [Spring kotlin RedisTemplate 사용법 \| Yoon Sung's Blog](https://unluckyjung.github.io/spring/2023/03/11/spring-redisTemplate/)
