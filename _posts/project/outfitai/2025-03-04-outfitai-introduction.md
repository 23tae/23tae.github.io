---
title: "SaaS를 대체한 의류 이미지 분류 라이브러리, OutfitAI"
date: 2025-03-04T09:00:00.000Z
categories: [Project, OutfitAI]
tags: [multimodal, ai]
---

![outfitai_thumbnail](/assets/img/project/outfitai/introduction/outfitai_thumbnail.png)

## 들어가며

지난 몇 달간 패션 관련 사이드 프로젝트에 참여했다. 당시 프로젝트의 핵심 기능 중 하나는 **사용자가 옷 사진을 찍어 올리면, AI가 해당 옷의 색상·카테고리·드레스코드·시즌을 자동으로 분류하여 앱 내 옷장에 저장하는 기능**이었다.

이 기능을 구현하기 위해 처음에는 상용 이미지 분류 서비스를 도입하려 했지만, 비용 문제로 인해 직접 라이브러리를 개발하게 되었다. 이 글은 그 과정에서 탄생한 [**OutfitAI**](https://pypi.org/project/outfitai/)를 소개한다.

## 배경: SaaS의 높은 비용 장벽

![pixyle_ai_homepage-1](/assets/img/project/outfitai/introduction/pixyle_ai_homepage-1.png)

초기에 도입을 검토한 서비스는 [pixyle.ai](https://www.pixyle.ai/)였다. 패션 이미지 분류에 특화된 B2B SaaS로, 정확도 측면에서는 충분히 만족스러웠다.

![pixyle_ai_homepage-2](/assets/img/project/outfitai/introduction/pixyle_ai_homepage-2.png)

문제는 가격이었다.

- **월 구독료**: €120
- **API 호출 한도**: 5,000회/월
- **호출당 비용**: 약 €0.024 (≈ $0.026)

사이드 프로젝트 규모의 팀이 월 €120을 고정 비용으로 지불하는 것은 부담이 컸다. 게다가 서비스 규모가 커져 5,000회 한도를 초과하면 추가 비용이 발생하는 구조였다. 처음부터 외부 서비스에 종속되는 것에 대한 불안감도 있었다.

이를 해결하기 위해 **LLM API를 활용한 자체 이미지 분류 라이브러리를 직접 개발**하기로 결정했다.

## 비용 비교: 직접 개발의 경제성

OutfitAI는 기본 프로바이더로 **gpt-4o-mini**를 사용한다. 옷 이미지 한 장을 분류할 때 소비하는 토큰을 기준으로 비용을 계산하면 다음과 같다.

- **입력 토큰** (이미지 + 프롬프트): 약 $0.00015
- **출력 토큰** (JSON 응답, 약 100토큰): 약 $0.00006
- **호출당 합계**: 약 **$0.0002**

| 항목 | pixyle.ai | OutfitAI (gpt-4o-mini) |
|---|---|---|
| 월 고정 비용 | €120 (~$130) | 없음 (사용량 기반) |
| 호출당 비용 | ~$0.026 | ~$0.0002 |
| 5,000회 기준 월 비용 | ~$130 | ~$1 |
| 호출 한도 | 5,000회 | 무제한 |

동일한 5,000회 호출 기준으로 **기존 대비 약 99%의 비용 절감**이 가능하다. 상용 툴에 버금가는 분류 정확도를 프롬프트 엔지니어링으로 확보하면서도, 비용은 사실상 무시할 수 있는 수준이 되었다.

## OutfitAI란

OutfitAI는 의류 이미지를 입력받아 **색상·카테고리·드레스코드·시즌 정보를 JSON 형식으로 반환**하는 Python 라이브러리다. CLI와 라이브러리 방식을 모두 지원한다.

### 분류 기준

| 항목 | 분류 값 |
|---|---|
| **색상** | white, gray, black, red, orange, yellow, green, blue, indigo, purple, other |
| **카테고리** | tops, bottoms, outerwear, dresses, shoes, bags, hats, accessories, other |
| **드레스코드** | casual wear, business attire, campus style, date night outfit, travel wear, wedding attire, loungewear, resort wear, other |
| **시즌** | spring, summer, fall, winter (복수 선택 가능) |

### 지원 기능

- **멀티 프로바이더**: OpenAI(gpt-4o-mini)와 Google Gemini 모두 지원
- **입력 방식**: 로컬 파일 경로와 이미지 URL 모두 지원
- **단일/배치 처리**: 이미지 한 장 또는 디렉토리 전체를 비동기로 처리
- **사용 방식**: CLI 명령어와 Python 라이브러리 모두 지원
- **PyPI 배포**: `pip install outfitai`로 즉시 설치 가능

### 출력 예시

![clothes_example](/assets/img/project/outfitai/introduction/clothes_example.jpg)

위의 이미지를 입력했을때 아래와 같은 JSON이 반환된다.

```json
[
  {
    "image_path": "image/clothes_1.jpg",
    "color": "red",
    "category": "tops",
    "dress_code": "casual wear",
    "season": [
      "spring",
      "fall",
      "winter"
    ]
  }
]
```

## 프로젝트 구조

```
src/outfitai/
├── __init__.py              # 공개 API: Settings, ClassifierFactory
├── __main__.py              # python -m outfitai 진입점
├── cli.py                   # CLI 인터페이스
├── classifier/
│   ├── base.py              # 추상 베이스 클래스 (BaseClassifier)
│   ├── openai_classifier.py # OpenAI 구현체
│   ├── gemini_classifier.py # Gemini 구현체
│   └── factory.py           # 프로바이더에 따라 구현체를 반환하는 팩토리
├── config/
│   └── settings.py          # pydantic-settings 기반 설정 관리
├── utils/
│   ├── image_processor.py   # 이미지 입력 추상화 및 프로바이더별 처리
│   └── logger.py            # 로거 설정
└── error/
    └── exceptions.py        # 도메인 예외 정의
```

각 모듈의 역할은 다음과 같다.

- **`classifier/`**: 분류 로직의 핵심. `BaseClassifier`를 추상 클래스로 두고 OpenAI와 Gemini 구현체가 각각 이를 상속한다. `ClassifierFactory`는 설정값을 읽어 적절한 구현체를 반환한다.
- **`utils/image_processor.py`**: 로컬 파일과 URL이라는 서로 다른 입력 방식을 `ImageSource`로 통일하고, 프로바이더별 처리 방식의 차이를 캡슐화한다.
- **`config/settings.py`**: `pydantic-settings` 기반으로 환경 변수, `.env` 파일, 코드 인자를 모두 지원하며, 프로바이더에 맞는 API 키 유효성을 초기화 시점에 검증한다.
- **`error/exceptions.py`**: `ValidationError`, `ClothingClassifierError` 등 도메인 예외를 정의하여, 호출 코드에서 세분화된 에러 처리가 가능하다.

## 사용 방법

### 라이브러리로 사용

```python
from outfitai import Settings, ClassifierFactory
import asyncio

# 환경 변수 또는 .env 파일에서 설정을 자동으로 로드
classifier = ClassifierFactory.create_classifier()

async def main():
    # 로컬 파일 분류
    result = await classifier.classify_single("path/to/image.jpg")

    # URL 분류
    result = await classifier.classify_single("https://example.com/image.jpg")

    # 배치 처리
    results = await classifier.classify_batch("path/to/images/")

asyncio.run(main())
```

### CLI로 사용

![outfitai_cli_example](/assets/img/project/outfitai/introduction/outfitai_cli_example.png)

```bash
# 단일 이미지 분류
outfitai path/to/image.jpg

# 결과를 파일로 저장
outfitai path/to/image.jpg --output result.json

# 디렉토리 내 전체 이미지 배치 처리
outfitai path/to/images/ --batch
```

## 마치며

OutfitAI는 비용 문제를 해결하기 위한 단순한 목적으로 시작됐지만, 개발 과정에서 멀티 프로바이더 추상화, 이미지 입력 레이어 설계, pydantic 기반 설정 관리, PyPI 배포 자동화까지 다양한 설계 포인트를 다루게 되었다.

이후 글들에서는 이러한 설계의 배경과 세부적인 구현 방식을 다룰 예정이다.
