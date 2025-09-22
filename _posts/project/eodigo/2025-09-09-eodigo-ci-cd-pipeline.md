---
title: "GitHub Actions 기반 CI/CD 파이프라인 구축기"
date: 2025-09-09T09:00:00.000Z
categories: [Project, 어디GO]
tags: [배포, aws, ci-cd]
---

## 서론

만약 별도의 배포 자동화를 구축하지 않는다면 개발자는 EC2 인스턴스에 직접 접속하여 `git pull`을 받고, `./gradlew build`로 애플리케이션을 빌드한 뒤, 실행 중인 프로세스를 죽이고 새 jar 파일을 실행하는 등의 과정을 거쳐 배포를 해야한다. 이 방식은 단순하지만 시간 소모와 배포 실패 가능성이라는 비효율을 지니고 있다.

이 문제를 예방하기 위해 ‘어디GO’ 프로젝트에서는 개발 초기부터 CI/CD 파이프라인을 구축하기로 계획했다. `develop` 브랜치에 코드가 병합되면 자동으로 테스트, 빌드, 배포까지 완료되는 안정적인 파이프라인 구축이 목표였다.

### CI/CD란?

![cicd](/assets/img/project/eodigo/ci-cd-pipeline/cicd.jpg)

**CI/CD**는 `개발 → 빌드 → 테스트 → 배포`에 이르는 소프트웨어 출시 과정을 자동화하는 방법론을 말한다. 이 자동화된 흐름을 **'CI/CD 파이프라인'**이라 부르며, 이를 통해 개발자는 코드의 품질을 일관되게 유지하고 사용자에게 더 빠르고 안정적으로 새로운 기능을 제공할 수 있다.

-   **CI (Continuous Integration, 지속적 통합)**  
    여러 개발자가 작업한 코드를 주기적으로 중앙 저장소에 통합(Merge)하고, 이때마다 빌드와 테스트를 자동으로 실행하여 코드의 문제를 조기에 발견하는 단계이다.

-   **CD (Continuous Delivery/Deployment, 지속적 제공/배포)**  
    CI를 통과한 코드를 사용자에게 전달하는 과정이다. **지속적 제공(Delivery)**은 배포 준비까지 자동화하되 실제 배포는 사람이 결정하는 것이고, **지속적 배포(Deployment)**는 이 마지막 단계까지 자동화하여 테스트 통과 시 코드가 자동으로 운영 환경에 반영되는 것을 의미한다.

## CI/CD 파이프라인 설계

### 전체 아키텍처 구성도

![cicd_pipeline.png](/assets/img/project/eodigo/ci-cd-pipeline/cicd_pipeline.png)
_CI(초록색)와 CD(빨간색)_

### 기술 스택 선정 이유

- **GitHub Actions:** 코드 저장소인 GitHub에 내장되어 있어 별도의 CI/CD 서버 구축 없이 즉시 사용할 수 있고, 다양한 오픈소스 Action들을 활용하여 워크플로우를 쉽게 구성할 수 있었다.
- **Docker & Amazon ECR:** 애플리케이션을 컨테이너화하여 개발 환경과 운영 환경의 차이를 없애고, 빌드된 이미지를 AWS ECR(Elastic Container Registry)에 안전하게 저장 및 관리하고자 했다.

### Github-hosted Runner와 Self-hosted Runner의 역할 분리

파이프라인의 효율과 보안을 위해 두 종류의 Runner를 함께 사용했다.

- **GitHub-hosted Runner:** 빌드 및 테스트(CI), Docker 이미지 빌드 및 ECR 푸시(CD의 일부)처럼 외부 인터넷 접근이 필요하고 강력한 컴퓨팅 자원이 요구되는 작업을 담당한다.
- **Self-hosted Runner:** 실제 운영 서버인 EC2 인스턴스에 직접 설치하여, ECR에서 이미지를 내려받고 컨테이너를 실행하는 배포(CD) 작업을 담당한다. 이 방식은 배포 서버에 SSH 접속 정보를 노출할 필요가 없어 보안적으로 더 안전하다.

## 파이프라인 구축을 위한 사전 준비

워크플로우(YAML) 파일을 작성하기에 앞서 몇 가지 사전 준비가 필요했다.

### ECR(Elastic Container Registry) 리포지토리 생성

![create ecr repository](/assets/img/project/eodigo/ci-cd-pipeline/create_ecr_repository.png)

도커 이미지를 관리할 비공개 저장소를 ECR로 생성했다.

### AWS와 GitHub Actions 연동: IAM 사용자 생성

외부 서비스인 GitHub Actions가 내 AWS 계정에 접근하려면 인증이 필요하다. 보안을 위해, CI/CD 작업에 필요한 최소한의 권한만 가진 전용 IAM 사용자를 생성했다.

1. **권한 정책 생성:** 먼저 `github-actions-policy`라는 이름의 고객 관리형 정책을 생성했다. 이 정책에는 ECR 로그인 및 이미지 푸시, EC2 인스턴스 정보 조회 등 CI/CD에 필요한 최소한의 권한(Action)만 포함시켰다.
2. **사용자 생성:** `github-actions-user`라는 이름으로 프로그래밍 방식 액세스 유형의 IAM 사용자를 생성하고, 위에서 만든 정책을 연결했다.
    
    ![aws_create_github_actions_user.png](/assets/img/project/eodigo/ci-cd-pipeline/aws_create_github_actions_user.png)
    
3. **GitHub Secrets 등록:** 생성된 사용자의 `액세스 키 ID`와 `비밀 액세스 키`를 Repository Secrets로 등록했다. 워크플로우에서는 이 Secrets를 통해 안전하게 AWS에 접근하였다.
    
    ![github_secrets.png](/assets/img/project/eodigo/ci-cd-pipeline/github_secrets.png)
    

### EC2에 Self-hosted Runner 설정

다음으로, 배포 작업을 수행할 EC2 인스턴스에 Self-hosted Runner를 설치했다.

1. **Runner 등록 (GitHub):** 먼저 GitHub 리포지토리에서 새로운 Runner를 등록했다. OS는 Linux, 아키텍처는 x64를 선택했다.
    
    ![github_self_hosted_runner.png](/assets/img/project/eodigo/ci-cd-pipeline/github_self_hosted_runner.png)
    
2. **Runner 설치 및 설정 (EC2):** GitHub에서 안내하는 명령어들을 EC2 인스턴스 터미널에서 순서대로 실행했다. `actions-runner` 디렉터리를 만들고, Runner 패키지를 다운로드하여 압축을 푼 뒤, `config.sh` 스크립트를 실행하여 내 리포지토리와 연결했다.
    
    ```bash
    # EC2 인스턴스에서 실행
    $ mkdir actions-runner && cd actions-runner
    $ curl -o actions-runner-linux-x64-....tar.gz -L https://...
    $ tar xzf ./actions-runner-linux-x64-....tar.gz
    $ ./config.sh --url <https://github.com/user/repo> --token YOUR_TOKEN
    ```
    
3. **서비스로 등록:** Runner가 EC2 인스턴스가 재부팅되어도 항상 실행되도록, `systemd` 서비스로 등록했다.
    
    ```bash
    $ sudo ./svc.sh install
    $ sudo ./svc.sh start
    ```
    

![self_hosted_runner_status.jpg](/assets/img/project/eodigo/ci-cd-pipeline/self_hosted_runner_status.jpg)

## CI 파이프라인 구현: Pull Request 코드 검증

`develop` 브랜치의 코드 품질을 유지하기 위해 **모든 merge가 PR**을 반드시 거치도록 하고, **force push 또한 금지**하는 **브랜치 보호 규칙(Branch Protection Rule)**을 설정했다.

![branch_protection_rule.png](/assets/img/project/eodigo/ci-cd-pipeline/branch_protection_rule.png)

`ci.yml` 워크플로우는 `develop` 브랜치로의 PR이 생성될 때마다 실행되며, Gradle 캐싱을 적용하여 빌드 속도를 최적화했다.

{% raw %}
```yaml
name: CI - Build and Test

on:
  pull_request:
    branches:
      - develop

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      # 1. 소스 코드 체크아웃
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. JDK 설정
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      # 3. Gradle 캐싱 설정
      - name: Cache Gradle packages
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

      # 4. gradlew 파일에 실행 권한 부여
      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew

      # 5. Gradle로 빌드 및 테스트 실행
      - name: Build with Gradle
        run: ./gradlew build
```
{% endraw %}

## CD 파이프라인 구현: EC2 서버 자동 배포

사전 준비가 완료된 후, `develop` 브랜치에 코드가 Push될 때 실행되는 CD 파이프라인을 작성했다.

### **Dockerfile**

효율적인 CI/CD를 위해서는 가볍고 빌드가 빠른 Docker 이미지를 만드는 것이 중요하다. 나는 다음과 같은 최적화 전략을 적용하여 Dockerfile을 작성했다.

- **멀티 스테이지 빌드(Multi-stage build):** AS builder 키워드를 사용해 빌드 환경과 실행 환경을 분리했다. 빌드에만 필요한 Gradle과 JDK는 첫 번째 스테이지에서만 사용되고, 최종 이미지에는 실행에 필요한 JRE와 app.jar 파일만 포함된다. 이를 통해 최종 이미지의 크기를 획기적으로 줄이고 보안을 강화했다.
- **경량 베이스 이미지 사용:** 최종 이미지의 베이스로 `eclipse-temurin:17-jre-alpine`을 선택했다. JDK가 아닌 JRE(Java Runtime Environment)만 포함하고, 최소한의 패키지만 설치된 Alpine Linux 기반이라 이미지 용량이 매우 작다.
    
    ![docker_image_jdk.png](/assets/img/project/eodigo/ci-cd-pipeline/docker_image_jdk.png)
    _JDK 용량: 142MB_
    
    ![docker_image_jre.png](/assets/img/project/eodigo/ci-cd-pipeline/docker_image_jre.png)
    _JRE 용량: 63MB_
    
- **Docker 레이어 캐싱 활용:** `COPY src ./src` 이전에 `COPY build.gradle.kts .`와 `RUN ./gradlew dependencies`를 먼저 실행했다. 이렇게 하면 소스 코드가 변경되더라도 `build.gradle.kts` 파일이 변경되지 않았다면, 의존성을 다운로드하는 무거운 작업을 건너뛰고 캐시된 레이어를 재사용하여 빌드 속도를 크게 향상시킬 수 있다.

최종 Dockerfile은 다음과 같다.

```docker
# --- 1단계: 애플리케이션 빌드 스테이지 ---
FROM gradle:8.5-jdk17 AS builder

WORKDIR /app

COPY --chown=gradle:gradle gradlew .
COPY --chown=gradle:gradle gradle ./gradle

# 의존성 파일만 먼저 복사하여 빌드 캐시 활용
COPY build.gradle.kts .
COPY settings.gradle.kts .
RUN ./gradlew dependencies

# 소스 코드 복사 후 빌드 실행
COPY src ./src
RUN ./gradlew bootJar --no-daemon

# --- 2단계: 최종 이미지 생성 스테이지 ---
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# EC2 t3.small 환경에 맞는 JVM 메모리 옵션 설정
ENV JAVA_TOOL_OPTIONS="-Xms256m -Xmx1024m"

# 빌드 스테이지에서 생성된 jar 파일만 복사
COPY --from=builder /app/build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

이후 워크플로우 파일(`dev-ci-cd.yml`)을 작성했다. 이 워크플로우는 두 개의 Job으로 구성된다. ([최종 dev-ci-cd.yml](https://github.com/dnd-side-project/dnd-13th-2-backend/blob/e2e34089961d36316feca3249d35d7d5502cab3c/.github/workflows/dev-ci-cd.yml))

### Job 1: Docker 이미지 빌드 및 ECR 푸시

GitHub-hosted Runner에서 실행되며, AWS 자격 증명을 이용해 ECR에 로그인한 뒤 `docker buildx build` 명령어로 Docker 이미지를 빌드하고 ECR에 푸시한다. 이미지 태그는 `github.sha`를 사용하여 각 커밋마다 고유한 이미지가 생성되도록 했다.

```yaml
# .github/workflows/dev-ci-cd.yml (build-and-push Job)

build-and-push:
  runs-on: ubuntu-latest
  steps:
    # ... (Checkout, Configure AWS credentials, Login to ECR)
    - name: Build and push Docker image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker buildx build \\
          --platform linux/amd64 \\
          --tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \\
          --output type=image,push=true \\
          .
```

### Job 2: Self-hosted Runner를 이용한 애플리케이션 배포

`build-and-push` Job이 성공하면, EC2의 Self-hosted Runner에서 `deploy` Job이 실행된다. 이 Job은 간단한 쉘 스크립트로 구성되어 있다.

1. **ECR 로그인:** `aws ecr get-login-password` 명령어로 ECR에 로그인한다.
2. **최신 이미지 Pull:** `build-and-push` Job에서 생성된 이미지 URI를 받아 이미지를 내려받는다.
3. **기존 컨테이너 교체:** 실행 중인 기존 컨테이너가 있다면 중지하고 삭제한다.
4. **새 컨테이너 실행:** 새로운 이미지로 컨테이너를 실행한다.
5. **헬스체크:** `curl` 명령어로 `/actuator/health` 엔드포인트를 주기적으로 호출하여, 애플리케이션이 정상적으로 실행되었는지 확인한다. (이 엔드포인트를 활성화하려면 `spring-boot-starter-actuator` 의존성을 추가해주어야 한다.) 12번의 시도에도 `200 OK` 응답이 오지 않으면 배포를 실패로 간주하고 컨테이너 로그를 출력한다.

![cicd_workflow_success.png](/assets/img/project/eodigo/ci-cd-pipeline/cicd_workflow_success.png)

## 문제 및 해결 과정

파이프라인 구축 과정에서 여러 문제를 겪었다. 그중 가장 의미 있었던 세 가지 문제와 해결 과정을 공유한다.

### 문제 1: CPU 아키텍처 불일치로 인한 컨테이너 실행 오류

워크플로우가 실패해서 `docker ps`를 확인했는데 컨테이너가 `Restarting (255)` 상태를 무한 반복하고 있었다.

```bash
sh-5.2$ docker ps
CONTAINER ID   IMAGE                                                                                                        COMMAND               CREATED         STATUS                            PORTS     NAMES
8a27cf1e35e3   032068930858.dkr.ecr.ap-northeast-2.amazonaws.com/dnd-project/app:fb376a81522b1680291158141ad719efc3c88df4   "java -jar app.jar"   2 minutes ago   Restarting (255) 58 seconds ago             dnd-app
```

![cd_workflow_fail_1.png](/assets/img/project/eodigo/ci-cd-pipeline/cd_workflow_fail_1.png)

배포 로그에 찍힌 `WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64)` 메시지가 결정적인 단서였다. GitHub Runner(x86)에서 빌드된 이미지가 `t4g.small` EC2(ARM)에서 실행되지 않는 CPU 아키텍처 불일치 문제였다.

초기에는 `docker buildx`를 이용해 ARM용 이미지를 빌드(크로스 컴파일)하도록 파이프라인을 수정했다.

```docker
jobs:
    steps:
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
```

하지만 QEMU 에뮬레이션으로 인해 빌드 시간이 증가했고 `setup-buildx-action` 사용으로 인한 추가적인 권한 문제도 발생했다. 결국 복잡한 빌드 파이프라인 대신, 문제의 근원인 **EC2 인스턴스를 `t3.small`(x86)로 교체**하여 빌드 환경과 배포 환경의 아키텍처를 통일하는 것으로 최종 해결했다.

### 문제 2: AWS IAM 정책 미비로 인한 ECR 접근 실패

![cd_workflow_fail_2.png](/assets/img/project/eodigo/ci-cd-pipeline/cd_workflow_fail_2.png)

아키텍처 문제를 해결한 직후, 이번엔 ECR에 이미지를 푸시하는 과정에서 `403 Forbidden` 에러가 발생했다. 원인은 CI/CD용 IAM 사용자가 ECR에 이미지를 푸시하는 데 필요한 세부 권한이 부족했기 때문이었다.

![modify_aws_github_actions_policy.png](/assets/img/project/eodigo/ci-cd-pipeline/modify_aws_github_actions_policy.png)

에러 로그를 분석하여 `ecr:GetAuthorizationToken`, `ecr:PutImage`, `ecr:GetDownloadUrlForLayer` 등 꼭 필요한 액션(Action)만 기존의 고객 관리형 정책에 하나씩 추가하는 방식으로 최소 권한 원칙을 지키며 문제를 해결했다.

### 문제 3: Self-hosted Runner 비활성화로 인한 워크플로우 지연

![cd_workflow_queued.png](/assets/img/project/eodigo/ci-cd-pipeline/cd_workflow_queued.png)

모든 문제를 해결했다고 생각했을 때, 배포 Job이 `Queued` 상태로 무한 대기하는 문제가 발생했다. Github Runner 탭에서도 runner가 Offline으로 표시되어 있었다.

![github_self_hosted_runner_offline.png](/assets/img/project/eodigo/ci-cd-pipeline/github_self_hosted_runner_offline.png)

EC2에 접속하여 `sudo ./svc.sh status`로 상태를 확인하니 `inactive (dead)` 상태로 멈춰 있었다. EC2 인스턴스를 다시 생성하는 과정에서 Runner 서비스를 systemd에 등록하는 작업을 누락했던 것으로 추정한다.

![ec2_self_hosted_runner_inactive.png](/assets/img/project/eodigo/ci-cd-pipeline/ec2_self_hosted_runner_inactive.png)

`sudo ./svc.sh start`로 서비스를 재시작하자, 대기 중이던 워크플로우가 즉시 실행되며 최종적으로 배포에 성공할 수 있었다.

### 문제 4: ECR 이미지 누적으로 인한 스토리지 비용 우려

![ecr images 1](/assets/img/project/eodigo/ci-cd-pipeline/ecr_images.png)

CI/CD 파이프라인이 안정적으로 동작한 지 3일째 되던 날, ECR 리포지토리를 확인해보니 이미지가 계속 쌓여 47개에 달했다. 일부 이미지의 크기는 1GB에 육박하여, 계속 방치될 경우 불필요한 스토리지 비용이 과금될 것이 우려되었다.

> **요금 세부 정보(프리 티어 한도를 초과하는 경우)**  
> 스토리지: 프라이빗 또는 퍼블릭 리포지토리에 저장된 데이터의 경우에 **GB/월당 USD 0.10**입니다.

이 문제를 해결하기 위해 ECR의 **수명 주기 정책(Lifecycle Policy)**을 설정하여 오래되거나 불필요한 이미지를 자동으로 정리하도록 했다.

설정한 규칙은 다음과 같다.

1. **태그 없는 이미지 정리 (우선순위 1)**  
태그가 지정되지 않은 이미지가 푸시된 지 3일이 지나면 자동으로 삭제되도록 설정했다. 이는 빌드 과정에서 발생할 수 있는 중간 이미지나 잘못된 이미지를 정리하는 역할을 한다.
  ![ecr lifecycle policy 1](/assets/img/project/eodigo/ci-cd-pipeline/ecr_lifecyle_policy_1.png)

2. **태그된 이미지 개수 제한 (우선순위 2)**  
태그가 지정된 이미지의 경우, 최신순으로 최대 5개까지만 보관하도록 설정했다. 5개를 초과하는 가장 오래된 이미지는 자동으로 삭제된다. 이를 통해 롤백 등을 위한 최소한의 버전은 유지하면서도 이미지가 무한정 쌓이는 것을 방지했다.
  ![ecr lifecycle policy 2](/assets/img/project/eodigo/ci-cd-pipeline/ecr_lifecyle_policy_2.png)

정책을 적용하고 몇 시간 뒤 수명 주기 이벤트가 처음 실행되면서 이미지 개수가 47개에서 33개로 감소했다. 이를 통해 CI/CD 과정에서 생성되는 Docker 이미지를 자동으로 관리하고, ECR 스토리지 비용을 줄일 수 있게 되었다.

![ecr images 2](/assets/img/project/eodigo/ci-cd-pipeline/ecr_images_after_event.png)
_수명 주기 정책 적용 후_

## 결론

파이프라인 도입으로 인해 우리 팀의 개발자들은 더 이상 배포에 신경 쓰지 않고 코드 작성에만 집중할 수 있게 되었다. 모든 배포는 **자동화된 프로세스**를 통해 일관성 있게 이루어졌고, PR 단계에서 빌드와 테스트를 거치며 `develop` 브랜치의 안정성이 크게 향상되었다.

CI/CD 파이프라인 구축은 단순히 YAML 파일을 작성하는 작업이 아닌, 다양한 요소와의 상호작용을 이해해야 하는 복합적인 과정이었다. 각종 에러를 겪고 이를 해결해나가며 많은 것을 배운 것 같다.

## 참고 자료

- [CI/CD - Wikipedia](https://en.wikipedia.org/wiki/CI/CD)
- [What Is CI/CD and How Does It Work? \| Black Duck](https://www.blackduck.com/glossary/what-is-cicd.html)
- [Managing a branch protection rule - GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)
- [효율적인 도커 이미지 만들기 #1 - 작은 도커 이미지](https://bcho.tistory.com/1356)
- [Spring Boot의 헬스 체크 애플리케이션 상태를 모니터링하는 방법](https://positive-impactor.tistory.com/191)
-[완전관리형 컨테이너 레지스트리 - Amazon Elastic Container Registry 요금 - Amazon Web Services](https://aws.amazon.com/ko/ecr/pricing/)
- [Amazon ECR의 리포지토리에 대한 수명 주기 정책 생성 - Amazon ECR](https://docs.aws.amazon.com/ko_kr/AmazonECR/latest/userguide/lp_creation.html)
