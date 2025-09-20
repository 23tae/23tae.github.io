---
title: "Spring Boot 전역 예외 처리 시스템 구축하기"
date: 2025-09-10T09:00:00.000Z
categories: [Project, 어디GO]
tags: [spring-boot, kotlin, exception-handling]
---

## 들어가며

어디GO의 백엔드 API 개발을 시작하기 전에 **전역 예외 처리 시스템**을 구축했다. API에서 발생하는 예외를 일관된 형식으로 응답하는 것은 프론트엔드와의 원활한 협업을 위해 필수적이기 때문이다.

이 글에서는 내가 정의한 에러 응답 규약과, Spring의 동작 원리에 기반해 `GlobalExceptionHandler`를 구현한 과정을 기록한다.

## 1. 에러 응답 규약 정의

본격적인 구현에 앞서, 클라이언트와 약속할 두 가지 규약을 먼저 정의했다.

- **`ErrorResponse` DTO**  
모든 예외는 `status`, `code`, `message` 필드를 갖는 `ErrorResponse` DTO 형식으로 반환하기로 했다. 이를 통해 클라이언트는 어떤 에러가 발생하더라도 항상 동일한 구조의 응답을 파싱하여 처리할 수 있다.
    - `ErrorResponse.kt`

      ```kotlin
      package com.eodigo.common.exception
      
      /**
        * 예외 발생 시 클라이언트에게 반환될 공통 응답 DTO
        *
        * @property status HTTP 상태 코드 값
        * @property code 에러 코드
        * @property message 에러 메시지
        */
      data class ErrorResponse(val status: Int, val code: String, val message: String) {
          companion object {
              /** ErrorCode를 기반으로 ErrorResponse 객체를 생성하는 정적 팩토리 메서드 */
              fun of(errorCode: ErrorCode): ErrorResponse {
                  return ErrorResponse(
                      status = errorCode.status.value(),
                      code = errorCode.code,
                      message = errorCode.message,
                  )
              }
      
              /** ErrorCode와 커스텀 메시지를 기반으로 ErrorResponse 객체를 생성하는 정적 팩토리 메서드 */
              fun of(errorCode: ErrorCode, message: String): ErrorResponse {
                  return ErrorResponse(
                      status = errorCode.status.value(),
                      code = errorCode.code,
                      message = message,
                  )
              }
          }
      }
      ```
        
- **`ErrorCode` Enum**  
에러의 종류를 구분하는 코드는 `String` 상수가 아닌 `Enum`으로 관리했다. `ErrorCode` Enum은 HTTP 상태 코드, 고유 에러 코드, 기본 메시지를 하나의 객체로 묶어 관리의 용이성을 높이고, 컴파일 시점에 타입을 체크하여 휴먼 에러를 방지한다.
    - `ErrorCode.kt`

      ```kotlin
      package com.eodigo.common.exception
      
      import org.springframework.http.HttpStatus
      
      /**
        * 전역에서 사용할 에러 코드를 정의하는 Enum 클래스
        *
        * @property status HTTP 상태 코드
        * @property code 에러 코드
        * @property message 기본 에러 메시지
        */
      enum class ErrorCode(val status: HttpStatus, val code: String, val message: String) {
          // Common
          INVALID_INPUT_VALUE(HttpStatus.BAD_REQUEST, "C001", "유효하지 않은 입력 값입니다."),
          METHOD_NOT_ALLOWED(HttpStatus.METHOD_NOT_ALLOWED, "C002", "지원하지 않는 HTTP 메서드입니다."),
          INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "C003", "서버 내부에서 오류가 발생했습니다."),
          INVALID_TYPE_VALUE(HttpStatus.BAD_REQUEST, "C004", "요청 값의 타입이 유효하지 않습니다."),
          ACCESS_DENIED(HttpStatus.FORBIDDEN, "C005", "접근 권한이 없습니다."),
      }
      ```
        

## 2. `GlobalExceptionHandler` 구현

- **`@RestControllerAdvice`의 동작 흐름**
    
    `GlobalExceptionHandler` 구현에 사용한 `@RestControllerAdvice`는 Spring MVC 내부에서 특정 흐름을 통해 동작한다. 요청 처리 중 컨트롤러에서 예외가 발생하면 `DispatcherServlet`은 등록된 `HandlerExceptionResolver`들에게 처리를 위임한다. 이 중 `ExceptionHandlerExceptionResolver`가 `@ControllerAdvice` 클래스 내부에 정의된 `@ExceptionHandler` 메서드를 찾아 실행하는 역할을 수행한다.
    
    ![spring_exception_handling.png](/assets/img/project/eodigo/global-exception-handler/spring_exception_handling.png)
    
- **예외 종류에 따른 처리**

  위 동작 원리를 바탕으로 `GlobalExceptionHandler`를 구현했다. `@Valid` 유효성 검사 실패 시 발생하는 `MethodArgumentNotValidException`, URL 경로 변수의 타입이 맞지 않을 때의 `MethodArgumentTypeMismatchException` 등 Spring에서 발생하는 주요 예외와 우리가 직접 정의한 `CustomException`을 각각의 `@ExceptionHandler` 메서드에서 처리하도록 구성했다.

  - `GlobalExceptionHandler.kt`
      
    ```kotlin
    package com.eodigo.common.exception
    
    import org.slf4j.LoggerFactory
    import org.springframework.http.ResponseEntity
    import org.springframework.web.HttpRequestMethodNotSupportedException
    import org.springframework.web.bind.MethodArgumentNotValidException
    import org.springframework.web.bind.annotation.ExceptionHandler
    import org.springframework.web.bind.annotation.RestControllerAdvice
    import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException
    
    @RestControllerAdvice
    class GlobalExceptionHandler {
    
        private val log = LoggerFactory.getLogger(javaClass)
    
        /** @Valid 어노테이션을 통한 DTO 검증 실패 시 발생하는 예외를 처리 */
        @ExceptionHandler(MethodArgumentNotValidException::class)
        private fun handleMethodArgumentNotValidException(
            e: MethodArgumentNotValidException
        ): ResponseEntity<ErrorResponse> {
            val errorCode = ErrorCode.INVALID_INPUT_VALUE
            val summarizedErrors =
                e.bindingResult.fieldErrors.joinToString(", ") { "${it.field}:${it.defaultMessage}" }
            log.warn(
                "MethodArgumentNotValidException: code={}, message='{}'",
                errorCode.code,
                summarizedErrors,
            )
            val fieldError = e.bindingResult.fieldErrors.firstOrNull()
            val errorMessage = fieldError?.defaultMessage ?: errorCode.message
            val response = ErrorResponse.of(errorCode, errorMessage)
            return ResponseEntity(response, errorCode.status)
        }
    
        /** @PathVariable 등에서 타입 변환 실패 시 발생하는 예외를 처리 */
        @ExceptionHandler(MethodArgumentTypeMismatchException::class)
        private fun handleMethodArgumentTypeMismatchException(
            e: MethodArgumentTypeMismatchException
        ): ResponseEntity<ErrorResponse> {
            val errorCode = ErrorCode.INVALID_TYPE_VALUE
            log.warn(
                "MethodArgumentTypeMismatchException: code={}, message='{}'",
                errorCode.code,
                e.message,
            )
            val response = ErrorResponse.of(errorCode)
            return ResponseEntity(response, errorCode.status)
        }
    
        /** 지원하지 않는 HTTP 메서드 요청 시 발생하는 예외를 처리 */
        @ExceptionHandler(HttpRequestMethodNotSupportedException::class)
        private fun handleHttpRequestMethodNotSupportedException(
            e: HttpRequestMethodNotSupportedException
        ): ResponseEntity<ErrorResponse> {
            val errorCode = ErrorCode.METHOD_NOT_ALLOWED
            log.warn(
                "HttpRequestMethodNotSupportedException: code={}, message='{}'",
                errorCode.code,
                e.message,
            )
            val response = ErrorResponse.of(errorCode)
            return ResponseEntity(response, errorCode.status)
        }
    
        /** 직접 정의한 비즈니스 예외를 처리 */
        @ExceptionHandler(CustomException::class)
        private fun handleCustomException(e: CustomException): ResponseEntity<ErrorResponse> {
            val errorCode = e.errorCode
            log.warn("Custom Exception: code={}, message='{}'", errorCode.code, e.message)
            val response = ErrorResponse.of(errorCode)
            return ResponseEntity(response, errorCode.status)
        }
    
        /** 위에서 처리되지 못한 모든 예외를 처리 */
        @ExceptionHandler(Exception::class)
        private fun handleException(e: Exception): ResponseEntity<ErrorResponse> {
            log.error("Unhandled Exception", e)
            val response = ErrorResponse.of(ErrorCode.INTERNAL_SERVER_ERROR)
            return ResponseEntity(response, ErrorCode.INTERNAL_SERVER_ERROR.status)
        }
    }
    ```
    

## 마치며

전역 예외 처리 시스템 덕분에 중복적인 `try-catch` 코드 없이 모든 예외 처리 로직을 한 곳에서 관리할 수 있게 되었다. 또한 코드의 유지보수성을 높이고 클라이언트 팀과의 명확한 소통 기반이 마련되었다.

## 참고자료

- [Exceptions :: Spring Framework](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/exceptionhandlers.html)
- [Exception Handling in Spring MVC](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc)
- [[Java] Global Exception 이해하고 구성하기 : Controller Exception](https://adjh54.tistory.com/79)
