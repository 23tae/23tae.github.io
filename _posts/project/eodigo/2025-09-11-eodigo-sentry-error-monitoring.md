---
title: "Sentry 기반 실시간 에러 모니터링 환경 구축기"
date: 2025-09-11T09:00:00.000Z
categories: [Project, 어디GO]
tags: [sentry, error-monitoring, ci-cd]
---

![sentry](/assets/img/project/eodigo/sentry-error-monitoring/sentry_logo.png)

## 들어가며

[**이전 글**](../eodigo-global-exception-handler)에서 어디GO의 예외 처리 시스템을 구축한 과정을 다루었다. 하지만 해당 시스템만으로는 예측하지 못한 런타임 에러가 발생했을 때 이를 신속하게 인지하기 어려웠다.

이 글에서는 이 문제를 해결하기 위해 실시간 에러 트래킹 도구인 Sentry를 도입하고, GitHub Actions 기반의 CI/CD 파이프라인에 통합한 과정을 설명한다.

## 1. Sentry 프로젝트 설정 및 알림 연동

애플리케이션에 코드를 추가하기 전, Sentry 웹사이트와 Slack에서 사전 설정 작업을 진행했다.

- **Sentry 프로젝트 생성 및 키 발급**
    - Sentry에 접속하여 'eodigo-backend' 이름으로 새 프로젝트를 생성했다.
    - 프로젝트 생성 후 발급된 **DSN(Data Source Name)** 키를 복사해두었다. 이 키는 Spring Boot 애플리케이션이 어느 Sentry 프로젝트로 에러를 전송할지 알려주는 고유 주소 역할을 한다.
    - CI/CD 연동에 필요한 **인증 토큰(Auth Token)**을 추가로 생성했다.
        
        ![image.png](/assets/img/project/eodigo/sentry-error-monitoring/sentry_auth_token.png)
        
- **Slack 연동 및 알림 규칙 설정**
    - Sentry 대시보드의 `Integrations` 메뉴에서 Slack을 추가하고 인증을 완료했다.
        
        ![image.png](/assets/img/project/eodigo/sentry-error-monitoring/sentry_add_slack_app.png)
        
        ![image.png](/assets/img/project/eodigo/sentry-error-monitoring/sentry_authorize.png)
        
    - 알림을 받을 Slack 채널에 Sentry 앱을 추가했다.
    - **Alert Rule**을 설정하여 `dev` 환경에서 **새로운 유형의 에러(New Issue)**가 발생했을 때만 지정된 Slack 채널로 알림이 오도록 구성했다. 이를 통해 중복 알림을 방지하고 중요한 문제에만 집중할 수 있도록 했다.
        
        ![sentry_alert_rule.png](/assets/img/project/eodigo/sentry-error-monitoring/sentry_alert_rule.png)
        

## 2. Spring Boot 애플리케이션 연동

Sentry 프로젝트 설정 후, Spring Boot 애플리케이션에 관련 의존성 및 설정을 추가했다.

- **의존성 및 플러그인 추가**  
  `build.gradle.kts`에 `sentry-spring-boot-starter` 의존성과 Sentry Gradle 플러그인을 추가했다. 플러그인은 빌드 시점에 소스 컨텍스트를 Sentry 서버로 업로드하여, 에러 발생 시 정확한 코드 라인을 추적할 수 있게 돕는다.
  - `build.gradle.kts`
      
    ```kotlin
    plugins {
        id("io.sentry.jvm.gradle") version "5.9.0"
    }
    
    sentry {
        includeSourceContext.set(true)
    
        org.set("dnd1302")
        projectName.set("eodigo-backend")
        authToken.set(System.getenv("SENTRY_AUTH_TOKEN"))
    }
    ```
        
- **애플리케이션 설정**  
  이전에 발급받은 DSN 키를 `application-dev.yml`에 설정했다. 개발 환경에서만 Sentry를 활성화하고, 다양한 옵션을 이곳에서 관리하도록 구성했다.
  - `application-dev.yml`
      
    ```yaml
    sentry:
      dsn: ${SENTRY_DSN}
      environment: dev
      traces-sample-rate: 1.0
      send-default-pii: true
      enabled: true
    ```
        

## 3. CI/CD 파이프라인 통합

애플리케이션 설정만으로는 Sentry의 모든 기능을 활용할 수 없다. 에러 발생 시 정확한 코드 위치를 추적하기 위해서는, Docker 이미지를 빌드하는 시점에 Sentry Gradle 플러그인이 소스 컨텍스트(Source Context)를 Sentry 서버로 업로드해야 한다. 이 과정에서 `SENTRY_AUTH_TOKEN`이 필요하며, 우리는 이 민감 정보를 안전하게 처리하기 위해 GitHub Actions 워크플로우를 수정했다.

- **GitHub Secrets 등록**`SENTRY_AUTH_TOKEN`과 `SENTRY_DSN` 같은 민감 정보는 코드에 노출되지 않도록 GitHub Repository의 Secrets에 등록했다.
- **Docker 빌드 시 안전하게 Secret 주입하기**  
  단순히 빌드 인자(`-build-arg`)로 인증 토큰을 전달하면 Docker 이미지 레이어에 민감 정보가 남을 수 있는 보안 취약점이 있다. 우리는 이 문제를 해결하기 위해 Docker의 **빌드 시크릿(build-time secrets)** 기능을 활용했다.
    
  먼저, `dev-ci-cd.yml` 워크플로우에서 `docker buildx build` 명령에 `--secret` 플래그를 추가했다. 이 플래그는 GitHub Secrets에 저장된 `SENTRY_AUTH_TOKEN`을 `sentryauthtoken`이라는 ID의 시크릿으로 빌드 컨텍스트에 전달한다.
    
  - `dev-ci-cd.yml`

    ```yaml
    - name: Build and push Docker image
      # ...
      env:
        SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
      run: |
        docker buildx build \\
          --platform linux/amd64 \\
          --secret id=sentryauthtoken,env=SENTRY_AUTH_TOKEN \\
          # ...
    ```
        
    
    다음으로, `Dockerfile`을 수정하여 Gradle 빌드 단계에서 이 시크릿을 사용하도록 했다. `RUN` 명령어에 `--mount=type=secret` 옵션을 추가하면, 시크릿이 빌드 과정 중인 컨테이너 내의 특정 경로(`/run/secrets/sentryauthtoken`)에 파일로 마운트된다. Gradle 빌드 전에 이 파일의 내용을 읽어 환경 변수로 설정하면, Sentry 플러그인이 이 토큰을 사용하여 소스 컨텍스트를 안전하게 업로드할 수 있다. 이 방식은 최종 이미지에는 토큰 정보가 전혀 남지 않는다는 장점이 있다.
    
    - `Dockerfile`
        
      ```docker
      # ...
      COPY src ./src
      
      RUN --mount=type=secret,id=sentryauthtoken \\
          export SENTRY_AUTH_TOKEN=$(cat /run/secrets/sentryauthtoken) && \\
          ./gradlew bootJar --no-daemon
      # ...
      
      ```
        
- **컨테이너 실행 시 DSN 환경 변수 주입**  
빌드가 완료된 이미지를 EC2에서 실행할 때는, `e` 옵션을 통해 GitHub Secrets에 저장된 `SENTRY_DSN`을 컨테이너의 환경 변수로 주입했다. 이를 통해 애플리케이션은 실행 시점에 어떤 Sentry 프로젝트로 에러를 전송해야 할지 알 수 있게 된다.
    - `dev-ci-cd.yml`

      ```yaml
      docker run -d --name $CONTAINER_NAME -p $HOST_PORT:8080 --restart always \
      -e SENTRY_DSN="$SENTRY_DSN" \
      $IMAGE_URI
      ```
        

## 4. 동작 확인

모든 설정이 완료된 후, `GlobalExceptionHandler`의 마지막 `handleException(e: Exception)` 메서드에서 처리되지 않은 예외가 발생하면 Sentry SDK가 이를 감지한다. 감지된 에러는 Sentry 대시보드에 이슈로 등록되고, 1.2 단계에서 설정한 규칙에 따라 Slack 채널로 즉시 알림이 전송된다.

![sentry_issue.png](/assets/img/project/eodigo/sentry-error-monitoring/sentry_issue.png)

![image.png](/assets/img/project/eodigo/sentry-error-monitoring/slack_notification_channel.png)

## 한계: Sentry 무료 플랜

Sentry에서 제공하는 공식 Slack 연동 기능은 Business Plan 이상의 유료 플랜에서만 사용 가능하다. 우리는 14일짜리 Business Plan 평가판을 사용했다.

당시 MVP 개발 기간이 약 1주일 정도 남아있었기 때문에, 평가판 기간 내에 프로젝트를 마무리할 수 있다고 판단했기 때문이다. 무료 플랜에서 Slack 알림을 구현하려면 **웹훅(Webhook)을 이용해 별도의 연동 로직을 구축**해야 하지만, 남은 기간과 개발 리소스를 고려했을 때 평가판의 **공식 연동 기능을 활용**하는 것이 더 효율적이었다.

평가판 기간이 종료된 후, Sentry 계정은 무료(Developer) 플랜으로 전환되었고 Slack 연동도 비활성화되었다.

![sentry_slack_disable.png](/assets/img/project/eodigo/sentry-error-monitoring/sentry_slack_disable.png)

만약 이 프로젝트의 개발을 이어간다면 무료 플랜에서 사용 가능한 알림 시스템으로 전환해야 한다. 서버에서 **Slack의 Incoming Webhooks**으로 메시지를 전달하는 방식 등이 대안이 될  수 있다.

## 마치며

Sentry 연동을 통해 우리는 더 이상 서버 로그를 주기적으로 확인하지 않아도 예측하지 못한 에러 발생을 실시간으로 인지하고 대응할 수 있는 환경을 갖추게 되었다. 이는 서비스 안정성을 유지하고 잠재적인 버그를 선제적으로 수정해 나가는 데 중요한 기반이 되었다.

## 참고

- [https://docs.sentry.io/platforms/java/guides/spring-boot/](https://docs.sentry.io/platforms/java/guides/spring-boot/)
- [https://docs.sentry.io/platforms/java/guides/spring-boot/configuration/options/](https://docs.sentry.io/platforms/java/guides/spring-boot/configuration/options/)
- [https://medium.com/@Hailey24/spring-서버에서-발생한-예외-sentry-issue-정보-slack-알림으로-보내기-dbb7e50ee49b](https://medium.com/@Hailey24/spring-%EC%84%9C%EB%B2%84%EC%97%90%EC%84%9C-%EB%B0%9C%EC%83%9D%ED%95%9C-%EC%98%88%EC%99%B8-sentry-issue-%EC%A0%95%EB%B3%B4-slack-%EC%95%8C%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EB%B3%B4%EB%82%B4%EA%B8%B0-dbb7e50ee49b)
