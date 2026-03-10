---
title: "Airflow 메모리 과점유 문제 해결: 웹서버 런타임 경량화와 모듈 격리"
date: 2026-03-01T09:00:00.000Z
categories: [Project, 뉴스낵]
tags: [troubleshooting, airflow, docker]
---

![airflow_logo](/assets/img/project/newsnack/introduction/airflow_logo.png)

## 배경 및 문제 상황

뉴스낵의 데이터 파이프라인은 단일 EC2 `t4g.medium` (ARM64, 4GB RAM) 인스턴스에서 동작한다. 이 한정된 자원 안에서 Airflow, FastAPI, Nginx 컨테이너가 함께 동작해야 하기에 메모리 최적화는 안정적인 서비스 운영을 위해 필수적이었다.

[**이전 포스팅**](../newsnack-arm64-migration)에서 다루었듯, 인프라 통합 당시 Airflow 웹서버의 기본 Gunicorn 워커(4개)가 메모리를 1.2GB까지 고갈시키는 이슈가 있었다. 당시 **워커 수를 1개로 제한**하여 점유율을 750MB대로 낮추며 메모리 고갈 문제를 해결할 수 있었다.

그러나 컨테이너들의 리소스 사용량을 점검하던 중, 여전히 해결되지 않은 근본적인 의문점이 생겼다.

```text
NAME                                    MEM USAGE / LIMIT   MEM %
newsnack-api                            297.6MiB / 1GiB     29.06%
newsnack-ai                             139.5MiB / 800MiB   17.43%
newsnack-pipeline-airflow-webserver-1   760.7MiB / 1.2GiB   61.91%
newsnack-pipeline-airflow-scheduler-1   817.6MiB / 2.2GiB   36.29%
nginx                                   3.906MiB / 128MiB   3.05%
```

무거운 DAG 파싱과 Task 관리를 담당하는 스케줄러(`newsnack-pipeline-airflow-scheduler-1`)가 메모리를 많이 쓰는 것은 납득할 수 있다. 그런데 **워커가 하나뿐인 웹서버(`newsnack-pipeline-airflow-webserver-1`)가 760MB의 메모리를 점유**하는건 납득하기 어려웠다. 

실제로 Stack Overflow나 공식 GitHub 이슈를 찾아보면, Airflow 특유의 구조 탓에 기본 설정만으로도 웹서버가 2GB 이상의 메모리를 점유하거나(Stack Overflow 사례), 무거운 모듈 로딩 탓에 런타임 환경에서 웹서버가 구동되기도 전에 메모리 고갈(OOM)을 일으킨다(GitHub Issue 사례)는 고질적인 문제 보고를 쉽게 찾아볼 수 있다. 단순 UI 서버 역할만을 수행해야 할 웹서버가 본질과 다르게 비대해진다는 건 시스템 아키텍처나 런타임 환경 구성에 불필요한 오버헤드가 존재한다는 뜻이다.

현재의 수치는 컨테이너의 메모리 예약량(Reservation, 800MB)에 근접한 수치였다. '단일 워커 환경인데도 왜 이렇게까지 메모리 점유율이 높을까?'라는 질문에서 출발해, 그 원인을 분석하고 런타임 환경을 근본적으로 경량화한 과정을 정리했다.

---

## 원인 분석

문제의 실마리를 찾기 위해 프로세스 추적부터 스케줄러의 아키텍처까지 4가지 단계로 나누어 진단을 진행했다.

### 1. 프로세스 레벨 메모리 점유 주체 파악

워커 수를 줄였음에도 여전히 750MB의 높은 점유율을 유지하는 상황에서, 추가적인 단서를 얻기 위해 웹서버 컨테이너의 로그를 확인했다.

- **Gunicorn 시작 로그 일부**
  ```text
  /home/airflow/.local/lib/python3.12/site-packages/azure/batch/models/_models_py3.py:4839 SyntaxWarning: invalid escape sequence '\s'
  /home/airflow/.local/lib/python3.12/site-packages/azure/synapse/artifacts/models/_models_py3.py:175 SyntaxWarning: invalid escape sequence '\d'
  ...
  ```

우리 서비스에서는 Azure를 전혀 사용하지 않는데 로그에는 `azure-batch` 등의 패키지가 로드된게 이상했다.

정확한 메모리 점유 주체를 찾기 위해 컨테이너 내부 프로세스별 메모리를 확인하고자 했다. Airflow 공식 이미지는 경량화를 이유로 `ps` 명령어가 존재하지 않았기에 리눅스 `/proc` 파일 시스템을 통해 프로세스별 **VmRSS**(물리 메모리 점유량)를 확인했다.

```text
541568 KB | PID 17  | gunicorn: master [airflow-webserver]
485756 KB | PID 195 | [ready] gunicorn: worker [airflow-webserver]
173380 KB | PID 8   | /home/airflow/.local/bin/python ...airflow
```

원인은 자식 워커가 아니라 **부모 프로세스인 Gunicorn Master**였다. Master 프로세스 단독으로 약 529MB의 메모리를 점유하고 있었다.

> 리눅스에서 `VmRSS`는 공유 프로세스 메모리를 중복 계산한다. `Gunicorn`은 마스터가 처음 파이썬 의존성 환경을 로드해 놓고 `fork()` 복제를 통해 워커를 생성하므로, 워커 메모리의 대부분은 마스터와 **Copy-on-Write**된 공유 가상 메모리다. 마스터 프로세스의 크기가 크다는 것은 웹서버 실행 시점에 과도한 모듈을 한 번에 `import`하고 있음을 의미했다.

### 2. 베이스 이미지 내장 패키지 오버헤드

마스터 프로세스가 의존성을 로드하는 내역을 파악하기 위해 컨테이너 내부 환경을 점검했다. (당시 `apache/airflow:2.10.4`를 베이스 이미지로 사용)

<details markdown="1">
<summary><strong>airflow-webserver 내부 Provider 목록</strong></summary>

```bash
$ docker exec -u airflow $(docker ps -qf "name=airflow-webserver") python -m pip list | grep apache-airflow-providers
apache-airflow-providers-amazon          9.1.0
apache-airflow-providers-celery          3.8.5
apache-airflow-providers-cncf-kubernetes 10.0.1
apache-airflow-providers-common-compat   1.2.2
apache-airflow-providers-common-io       1.4.2
apache-airflow-providers-common-sql      1.20.0
apache-airflow-providers-docker          3.14.1
apache-airflow-providers-elasticsearch   5.5.3
apache-airflow-providers-fab             1.5.1
apache-airflow-providers-ftp             3.11.1
apache-airflow-providers-google          11.0.0
apache-airflow-providers-grpc            3.6.0
apache-airflow-providers-hashicorp       3.8.0
apache-airflow-providers-http            4.13.3
apache-airflow-providers-imap            3.7.0
apache-airflow-providers-microsoft-azure 11.1.0
apache-airflow-providers-mysql           5.7.4
apache-airflow-providers-odbc            4.8.1
apache-airflow-providers-openlineage     1.14.0
apache-airflow-providers-postgres        5.14.0
apache-airflow-providers-redis           3.8.0
apache-airflow-providers-sendgrid        3.6.0
apache-airflow-providers-sftp            4.11.1
apache-airflow-providers-slack           8.9.2
apache-airflow-providers-smtp            1.8.1
apache-airflow-providers-snowflake       5.8.1
apache-airflow-providers-sqlite          3.9.1
apache-airflow-providers-ssh             3.14.0
```

</details>

확인 결과 Amazon, Google 등 총 **28개**의 클라우드 Provider SDK들이 기본적으로 설치되어 있었다. 실제로 우리 파이프라인에서 사용하는 모듈은 `postgres`와 `http` 단 2개뿐임에도 불구하고, 미사용 모듈 전체가 Gunicorn Master 시작 시점에 로드되어 메모리를 비대하게 만든 것이었다. 

패키지 수를 확인해본 결과 전체 설치 패키지 수가 무려 **376개**에 달했으며, 운영 환경에 배포할 Docker 이미지의 크기마저 **1.84GB**에 육박하는 등 극심한 오버헤드가 누적된 상태였다.

- **전체 패키지 수 및 이미지 크기**

  {% raw %}
  ```bash
  $ docker exec -u airflow $(docker ps -qf "name=airflow-webserver") python -m pip list | wc -l
  376

  $ docker images newsnack-pipeline:latest --format "{{.Repository}}:{{.Tag}}\t{{.Size}}"
  newsnack-pipeline:latest	1.84GB
  ```
  {% endraw %}

이 이미지는 시작을 빠르게 돕는 Full 이미지일 뿐 프로덕션을 위한 최적화된 이미지가 아니었다.

### 3. 단일 파일 기반 의존성 관리의 한계

추가적으로, 기존 코드는 `setup.py` 파일 내부에서 `requirements.txt`에 명시된 일반 패키지와 Airflow Provider 의존성을 함께 읽어들여 런타임 환경에 주입하는 **안티패턴**을 따르고 있었다. 

Airflow는 공식적으로 각 버전별 호환성을 보장하는 `constraints.txt`라는 제약 파일 구조를 갖는다. 그런데 `setup.py`에 모든 환경을 통합하여 관리하다 보니, 우리가 사용하는 `feedparser`나 `scikit-learn` 같은 일반 파이프라인 패키지가 Airflow의 엄격한 제약 시스템과 버젼 충돌을 일으키고, 복잡한 의존성 문제를 유발하는 구조적 한계가 존재했다.

### 4. DAG 파싱 방식과 모듈 강결합

웹서버 외에 **스케줄러 환경**에서도 발견된 또 다른 숨은 메모리 낭비 요인이 있었다. 내 `content_generation_dag.py` 파일의 최상단에는 아래와 같은 모듈 참조가 존재했다.

- `content_generation_dag.py`
```python
from newsnack_etl.database.models import IssueStatusEnum
```

순수하게 이슈 상태코드(`PENDING`, `COMPLETED` 등)를 문자열로 참조하기 위한 의도였다. 하지만 이런 Top-level 스크립트 공간의 모듈 참조는 Airflow 아키텍처 관점에서 매우 치명적인 성능 저하를 야기했다. 

Airflow 스케줄러는 데몬 루프를 돌며 30초마다 `dags_folder` 내의 모든 DAG 파일을 파싱(Parsing)해 `DagBag`을 갱신한다. 이때 스케줄러는 파일 내 **Top-level Import 구문을 매번 처음부터 다시 평가(Evaluate)한다.** 위 코드 한 줄로 인해, 단순한 Enum 상태 값을 얻기 위해 거대한 `SQLAlchemy` ORM 전체 모듈과 DB 연결 설정 관련 클래스들이 30초마다 지속적으로 메모리에 로드되고 해제되는 현상이 반복되었던 것이다.

---

## 해결 방법

이렇게 밝혀진 구조적 문제들을 세 단계에 걸쳐 해결했다.

### 1. Slim 기반 런타임 환경 경량화

불필요한 Provider가 모두 내장된 Full 이미지 대신, 코어만 포함되어 프로바이더가 내장되지 않은 `apache/airflow:slim-2.10.4` 로 베이스 이미지를 교체했다. 동시에 공식 Constraint 버전을 외부 변수(`ARG`)로 제공받아, 오직 명시한 패키지만 호환성에 맞게 설치되도록 `Dockerfile`을 재설계했다.

- `Dockerfile`

  ```diff
  -FROM apache/airflow:2.10.4-python3.12
  +# 필요한 Provider는 requirements.txt에서 명시적으로 관리
  +FROM apache/airflow:slim-2.10.4-python3.12
  
  +# Airflow 공식 constraint 파일
  +ARG AIRFLOW_VERSION=2.10.4
  +ARG CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-3.12.txt"
  +
  COPY --chown=airflow:root requirements.txt ./
  -RUN pip install --upgrade pip setuptools wheel && pip install -r requirements.txt
  +RUN pip install --upgrade pip setuptools wheel \
  +    && pip install -r requirements.txt --constraint "${CONSTRAINT_URL}"
  ```

### 2. 패키지 의존성 관리 체계 분리

의존성 충돌의 원인이었던 `setup.py`를 프로젝트에서 완전히 제거했다. 그 대신 성격에 맞게 2가지 체계로 **역할을 물리적으로 분리**했다.

- `requirements.txt`: Airflow 환경 의존성. 파이프라인에서 실제 사용할 `apache-airflow-providers-postgres`, `http` 2개만 명시.
- `pyproject.toml`: 뉴스낵 파이프라인 스크립트(`newsnack_etl`) 내부 자체 실행을 위한 라이브러리.

- `pyproject.toml`

  ```diff
  +# newsnack_etl 패키지 자체의 일반 의존성 분리
  +dependencies = [
  +    "feedparser==6.0.12",
  +    "scikit-learn==1.4.2",
  +    "psycopg2-binary",
  +]
  ```

이를 통해 새로운 Provider 플러그인이 필요할 때는 `requirements.txt`를, 파이썬 라이브러리가 필요할 때는 `pyproject.toml`을 수정하여 독립적으로 의존성을 관리할 수 있게 되었다.

### 3. SSOT 기반 상태 모듈 격리

30초마다 일어나는 스케줄러의 DAG 파싱 병목을 제거하기 위해, 강결합된 모델 레이어의 설계 구조를 개선했다.
순수 문자열 상태를 나타내는 `IssueStatusEnum`을 기존 `SQLAlchemy` 영역에서 분리하여, 어떠한 외부 프레임워크나 서드파티 패키지도 임포트하지 않는 **순수 파이썬 모듈(`common/enums.py`)로 격리**하였다.

결과적으로 DB 모델인 `database.models` 모듈과 통신 로직인 `content_generation_dag.py` 양쪽에서 이 가벼운 Enum 모듈을 각각 임포트하여 사용하는 형태가 되었다. 이렇게 하여 **단일 진실 공급원(SSOT)**을 보장함과 동시에 스케줄러가 파싱할 때 헤비한 리소스를 로드하는 문제를 완벽히 차단했다.

- `dags/content_generation_dag.py`

  ```diff
  -from newsnack_etl.database.models import IssueStatusEnum
  +from newsnack_etl.common.enums import IssueStatusEnum
  ```

---

## 결과

이러한 전면적인 런타임 설계 및 구조 리팩토링 직후, 시스템 전반의 리소스 점유 상태를 최종 측정했다.

### 1. 전체 컨테이너 메모리 감소

```text
NAME                                    MEM USAGE / LIMIT   MEM %
newsnack-api                            297.6MiB / 1GiB     29.06%
newsnack-pipeline-airflow-webserver-1   249.4MiB / 1.2GiB   20.30%
newsnack-pipeline-airflow-scheduler-1   435MiB / 2.2GiB     19.31%
newsnack-ai                             139.5MiB / 800MiB   17.43%
nginx                                   3.906MiB / 128MiB   3.05%
```

기존에 단독으로 760MB 이상을 점유하던 웹서버 메모리가 **249MB**로 대폭 감소했으며, 동일한 이미지를 공유하는 스케줄러 메모리 역시 817MB에서 435MB까지 절반 가까이 안정화되었다.

### 2. Gunicorn 프로세스 및 런타임 환경 경량화

**`/proc` 기반 프로세스 VmRSS 재측정 결과:**
```text
143008 KB | PID 16 | gunicorn: master [airflow-webserver]
127740 KB | PID 7  | /home/airflow/.local/bin/python /home/airflow/.local/bin/air
120712 KB | PID 19 | [ready] gunicorn: worker [airflow-webserver]
```

문제를 일으킨 주요 원인이었던 Gunicorn Master 프로세스의 메모리 점유량이 기존 529MB에서 **140MB**로 크게 감소했다. 

더불어 불필요한 클라우드 Provider SDK들이 대거 제거되었으며, 전체 패키지 수와 Docker 이미지 크기에서도 유의미한 경량화를 이뤄냈다.

- **airflow-webserver 내부 Provider 목록**

  ```bash
  $ docker exec -u airflow $(docker ps -qf "name=airflow-webserver") python -m pip list | grep apache-airflow-providers
  apache-airflow-providers-common-compat   1.2.2
  apache-airflow-providers-common-io       1.4.2
  apache-airflow-providers-common-sql      1.20.0
  apache-airflow-providers-fab             1.5.1
  apache-airflow-providers-ftp             3.11.1
  apache-airflow-providers-http            4.13.3
  apache-airflow-providers-imap            3.7.0
  apache-airflow-providers-postgres        5.14.0
  apache-airflow-providers-smtp            1.8.1
  apache-airflow-providers-sqlite          3.9.1
  ```

- **패키지 수 및 이미지 크기**

  {% raw %}
  ```bash
  $ docker exec -u airflow $(docker ps -qf "name=airflow-webserver") python -m pip list | wc -l
  154

  $ docker images newsnack-pipeline:latest --format "{{.Repository}}:{{.Tag}}\t{{.Size}}"
  newsnack-pipeline:latest	1GB
  ```
  {% endraw %}

최종적으로 설치된 Provider 개수는 10개, 코어 패키지 수는 154개, 파이프라인 이미지 크기는 1GB 수준으로 감소했다.

### 3. 지표별 개선율 요약

| 지표 | Before | After | 개선율 |
| --- | --- | --- | --- |
| **웹서버 전체 메모리** | 760.7 MiB (61.9%) | **249.4 MiB (20.3%)** | **▼ 67.2%** |
| **스케줄러 메모리** | 817.6 MiB (36.3%) | **435.0 MiB (19.3%)** | **▼ 46.8%** |
| 리눅스 Gunicorn Master | 529 MB | **140 MB** | **▼ 73.6%** |
| Docker 이미지 크기 | 1.84 GB | **1.00 GB** | **▼ 45.7%** |
| Provider 번들 개수 | 28개 | **10개** | **▼ 64.3%** |
| 총 패키지 개수 | 376개 | **154개** | **▼ 59.0%** |

이번 트러블슈팅을 통해 아래의 내용들을 배웠다.

1. **편리함과 최적화의 차이 알기:** `apache/airflow:latest` 같은 Full 이미지는 범용성과 편리함을 위한 것이며 항상 정답은 아니다.
2. **패키지 의존성 관리의 역할 분리:** 환경(Airflow) 의존성과 내 패키지(Pipeline) 의존성 관리 도구를 분리하면 충돌 위험이 줄어들고 관리 포인트가 명확해진다.
3. **증거 기반의 접근 방식:** 단순히 '메모리가 많다'가 아니라, `docker logs` 오류 메시지와 우회적인 `/proc` 메모리 추적 스크립트를 통해 **의존성 오버헤드**라는 정확한 원인을 파악할 수 있었다.

---

## 참고 자료

- [Stack Overflow: Airflow using all system resources](https://stackoverflow.com/questions/42419834/airbnb-airflow-using-all-system-resources)
- [GitHub Issue #29841: high memory leak, cannot start even webserver](https://github.com/apache/airflow/issues/29841)
- [Apache Airflow Best Practices - Top level Python Code](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#top-level-python-code)
- [Apache Airflow Docs - Installation With Constraints](https://airflow.apache.org/docs/apache-airflow/stable/installation/installing-from-pypi.html#constraints-files)
