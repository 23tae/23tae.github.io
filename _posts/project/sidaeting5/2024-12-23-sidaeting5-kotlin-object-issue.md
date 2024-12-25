---
title: "Kotlin object에서 Spring 설정값이 주입되지 않는 문제"
date: 2024-12-23T08:00:00.000Z
categories: [Project, 시대팅5]
tags: [spring-boot]
---

## 문제 상황

Spring Boot 애플리케이션에서 JWT 토큰 설정값을 `application.yml`로 관리하던 중 한 가지 문제가 발생했다. `SecurityConstants` object에서 `@Value` 어노테이션으로 토큰 만료 시간을 주입받으려 했으나, 값이 정상적으로 주입되지 않아 모든 토큰의 만료 시간이 0으로 설정되는 현상이었다.

### 설정값 구성

- `application.yml`
  ```yaml
  jwt:
      access:
          secret: ${JWT_ACCESS_SECRET}
          expiration: 3600000 # 유효시간(ms): 1시간
      refresh:
          secret: ${JWT_REFRESH_SECRET}
          expiration: 604800000 # 유효시간(ms): 7일
  ```

위 설정에는 두 가지 방식의 설정값이 사용된다.

1. **환경변수**: `${JWT_ACCESS_SECRET}`, `${JWT_REFRESH_SECRET}`와 같이 `${}`로 감싸진 값들은 실제 환경변수에서 가져오는 동적 값이다.
2. **정적 설정값**: `expiration`과 같이 직접 YAML 파일에 하드코딩된 값들은 정적 설정값이다.

### 설정값 주입 방식

`SecurityConstants`는 싱글톤(Singleton) 패턴을 구현하기 위해 Kotlin object로 선언하였고, `@Value` 어노테이션을 사용하여 `application.yml`의 설정값을 주입하려고 하였다.

```kotlin
@Component
object SecurityConstants {
...
  @Value("\${jwt.access.expiration}") var ACCESS_TOKEN_EXPIRATION: Long = 0
  @Value("\${jwt.refresh.expiration}") var REFRESH_TOKEN_EXPIRATION: Long = 0
}

@Component
class JwtTokenGenerator() {
  fun createAccessToken(id: Long): String {
      return createToken(id, accessKeyPair.private, SecurityConstants.ACCESS_TOKEN_EXPIRATION)
  }
}
```

## 원인 분석

이 문제의 근본적인 원인은 **Kotlin object의 초기화 시점**과 **Spring의 의존성 주입 시점** 간의 불일치에 있다.

1. **Kotlin object의 초기화**
    
    > The initialization of an object declaration is thread-safe and done on first access.

    - Kotlin 공식 문서에 따르면 `object`는 **최초 접근 시 단 한 번만 초기화**된다.
    - 하지만 이 초기화는 JVM 클래스 로더가 `object`에 처음 접근하는 시점에 이루어지며, 이는 Spring 애플리케이션 컨텍스트가 생성되기 전에 발생할 수 있다.
2. **Spring의 의존성 주입**
    - Spring 컨테이너는 애플리케이션 컨텍스트 초기화 과정에서 빈을 생성하고 의존성을 주입한다.
    - `@Value` 어노테이션을 통한 설정값 주입은 Spring 빈 생명주기 중 프로퍼티 설정 단계에서 이루어진다.
3. **문제 발생 원인**
    1. Kotlin `object`는 JVM에 의해 먼저 초기화된다.
    2. 이때 Spring 컨텍스트는 아직 생성되지 않은 상태이다.
    3. `@Value` 어노테이션이 동작할 수 없어 필드는 초기값(0)을 유지한다.
    4. 결과적으로 `application.yml`에 설정한 값이 아닌 초기값이 사용된다.

## 해결 방법

`SecurityConstants` object에서 설정값을 직접 주입받는 대신, 각 클래스가 필요한 설정값을 생성자나 프로퍼티를 통해 주입받는 방식으로 변경한다. 이를 통해 Spring의 의존성 주입 생명주기와의 불일치 문제를 해결할 수 있다.

### 방법 1: 클래스별 설정값 주입

`SecurityConstants` object를 사용하지 않고, 설정값을 각 클래스에서 직접 주입받도록 변경한다.

```kotlin
@Component
class JwtTokenGenerator(
  @Value("\${jwt.access.expiration}") private val accessTokenExpiration: Long,
  @Value("\${jwt.refresh.expiration}") private val refreshTokenExpiration: Long,
)
```

### 방법 2. @ConfigurationProperties 사용

`@ConfigurationProperties`를 사용하면 설정값을 한곳에 모아 관리하고, 각 클래스에서 쉽게 참조할 수 있다.

- 설정값 매핑 클래스
  ```kotlin
  @Configuration
  @ConfigurationProperties(prefix = "jwt")
  data class JwtProperties(
    val access: TokenProperties = TokenProperties(),
    val refresh: TokenProperties = TokenProperties(),
  ) {
    data class TokenProperties(
        var secret: String = "",
        var expiration: Long = 0,
    )
  }
  ```

- 사용 예시
  ```kotlin
  @Component
  class JwtTokenGenerator(private val jwtProperties: JwtProperties) {
    fun createAccessToken(id: Long): String {
      return createToken(id, accessKeyPair.private, jwtProperties.access.expiration)
    }
  }
  ```

## 참고자료

- [Object declarations and expressions \| Kotlin Documentation](https://kotlinlang.org/docs/object-declarations.html)
- [Using @Value :: Spring Framework](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/value-annotations.html)
- [Externalized Configuration :: Spring Boot](https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.typesafe-configuration-properties)
- [Kotlin object의 초기화 시점과 companion object의 초기화 시점 차이 알아보기 — 조세영의 Kotlin World](https://kotlinworld.com/420)
- [Spring Bean life cycle. In Spring Boot, the bean lifecycle is… \| by Namrata \| Medium](https://codescoddler.medium.com/spring-bean-life-cycle-34647f8fa1ab)
