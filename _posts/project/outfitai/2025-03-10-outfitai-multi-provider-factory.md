---
title: "OutfitAI 멀티 프로바이더 팩토리 패턴 구현기: OpenAI·Gemini 통합"
date: 2025-03-10T09:00:00.000Z
categories: [Project, OutfitAI]
tags: [design-pattern, factory-pattern]
---

| ![openai_logo](/assets/img/project/outfitai/multi-provider-factory/openai_logo.png) | ![gemini_logo](/assets/img/project/outfitai/multi-provider-factory/gemini_logo.png) |

## 배경

OutfitAI 개발 초기에는 AI 프로바이더로 OpenAI만을 지원하였다. 함수 하나에 OpenAI API 호출 로직이 담겨 있었고, 분류 결과를 반환하는 단순한 구조였다. 이 시점에는 "다른 프로바이더도 지원해야 할 이유"가 없었기 때문이다.

그러나 개발을 진행하면서 두 가지 필요가 생겼다.

첫째, **비용과 성능 비교**다. OpenAI와 Google의 모델들 중 의류 이미지 분류에 더 적합한 모델을 실험하려면 동일한 입력에 대해 두 모델의 결과를 비교할 수 있어야 했다.

둘째, **라이브러리 사용자의 선택권**이다. 이미 OpenAI API 키를 가진 사용자도 있고, Google 생태계를 선호하는 사용자도 있다. `pip install outfitai` 한 줄로 설치한 뒤 환경 변수 하나만 바꿔서 원하는 프로바이더를 사용할 수 있어야 한다.

이 두 가지 요구를 충족하면서, 기존 호출 코드가 전혀 바뀌지 않아야 한다는 조건을 함께 만족시키는 설계가 필요했다.

## 초기 구조의 문제

Gemini를 단순하게 추가한다면 어떻게 됐을까?

```python
# 단순 추가 방식 (채택하지 않은 구조)
if provider == "openai":
    client = openai.AsyncOpenAI(api_key=openai_key)
    response = await client.chat.completions.create(...)
    result = json.loads(response.choices[0].message.content)
elif provider == "gemini":
    client = genai.Client(api_key=gemini_key)
    response = client.models.generate_content(...)
    result = json.loads(response.text)
```

`if/elif` 분기로 구현하면 당장은 동작하지만, 문제가 명확하다.

- **분기 확산**: 이미지 처리, 프롬프트 구성, 에러 처리 등 모든 단계마다 `if/elif`가 필요해진다.
- **변경의 파급 범위**: 새 프로바이더를 추가할 때 분기가 존재하는 모든 곳을 찾아서 수정해야 한다.
- **테스트 어려움**: 특정 프로바이더를 모킹하거나 격리하기 어렵다.

세 번째 프로바이더가 추가된다고 가정하면, 이 구조는 금방 감당하기 어려워진다.

## 설계: 추상 클래스와 팩토리 패턴

채택한 구조는 두 가지 패턴의 조합이다.

- **템플릿 메서드 패턴**: 공통 흐름은 추상 베이스 클래스에 두고, 프로바이더별로 달라지는 부분만 서브클래스에서 구현한다.
- **팩토리 패턴**: 어떤 구현체를 생성할지 결정하는 책임을 별도 클래스에 위임한다.

```
BaseClassifier (추상 클래스)
├── OpenAIClassifier
└── GeminiClassifier

ClassifierFactory
└── create_classifier(settings) → BaseClassifier
```

호출 코드는 `BaseClassifier` 인터페이스만 알면 된다. 실제로 어떤 구현체가 동작하는지는 알 필요가 없다.

## 구현

### BaseClassifier: 공통 로직의 집합소

`BaseClassifier`는 두 가지 역할을 한다. 첫째, 프로바이더에 무관하게 동일한 로직(분류 단어 목록 정의, 프롬프트 생성, 응답 검증, 배치 처리)을 한 곳에서 관리한다. 둘째, 서브클래스가 반드시 구현해야 하는 인터페이스를 `@abstractmethod`로 강제한다.

```python
# classifier/base.py
from abc import ABC, abstractmethod

class BaseClassifier(ABC):
    def __init__(self, settings: Settings):
        self.settings = settings
        self.image_processor = ImageProcessor(self.settings)
        self._init_constants()

    def _init_constants(self):
        self.color_values = ["white", "gray", "black", ...]
        self.category_values = ["tops", "bottoms", "outerwear", ...]
        self.dress_code_values = ["casual wear", "business attire", ...]
        self.season_values = ["spring", "summer", "fall", "winter"]

    def _create_prompt(self) -> str:
        # 단어 목록을 프롬프트에 직접 주입
        return f"""
        Analyze the clothing item in the image and classify it according to these rules.
        Return a JSON object with these keys:
        - 'color': 1 value from {self.color_values}
        - 'category': 1 value from {self.category_values}
        - 'dress_code': 1 value from {self.dress_code_values}
        - 'season': 1+ values from {self.season_values} (array)
        """

    @abstractmethod
    async def classify_single(self, image_source) -> Dict[str, Any]:
        pass

    async def classify_batch(self, image_paths, batch_size=None) -> List[Dict[str, Any]]:
        # asyncio.gather를 활용한 배치 처리 공통 구현
        ...
```

`classify_single`만 추상 메서드로 선언하고 `classify_batch`는 이를 호출하는 형태로 구현했다. 새 프로바이더를 추가하더라도 `classify_single`만 구현하면 배치 처리는 자동으로 따라온다.

### OpenAIClassifier와 GeminiClassifier: 프로바이더별 구현

두 구현체의 `classify_single`은 이미지를 API에 맞는 형식으로 전달하는 방식과, 응답을 파싱하는 방식만 다르다.

**OpenAI**는 `messages` 배열 안에 `image_url` 타입 콘텐츠를 포함시키고, `response_format`으로 JSON 출력을 강제한다.

```python
# classifier/openai_classifier.py
async def classify_single(self, image_source) -> Dict[str, Any]:
    image_data = await self.image_processor.process_image(image_source)

    response = await self.client.chat.completions.create(
        model=self.settings.OPENAI_MODEL,
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": self.prompt_text},
                {"type": "image_url", "image_url": {"url": image_data, "detail": "low"}}
            ]
        }],
        max_tokens=self.settings.OPENAI_MAX_TOKENS,
        response_format={"type": "json_object"}
    )

    result = json.loads(response.choices[0].message.content)
    self._validate_response(result)
    return {"image_path": ..., **result}
```

**Gemini**는 `contents` 리스트에 텍스트와 이미지 파트를 함께 전달하고, `response_mime_type`으로 JSON 출력을 강제한다.

```python
# classifier/gemini_classifier.py
async def classify_single(self, image_source) -> Dict[str, Any]:
    image_part = await self.image_processor.process_image(image_source)

    response = self.client.models.generate_content(
        model=self.settings.GEMINI_MODEL,
        contents=[self.prompt_text, image_part],
        config={'response_mime_type': 'application/json'},
    )

    result = json.loads(response.text)
    self._validate_response(result)
    return {"image_path": ..., **result}
```

공통 부분(`_create_prompt`, `_validate_response`, `classify_batch`)은 `BaseClassifier`에 있으므로, 각 구현체는 실질적인 API 호출 로직만 담는다. 두 구현체를 나란히 보면 구조가 대칭적이라는 것을 알 수 있다.

### ClassifierFactory: 구현체 선택의 단일 지점

`ClassifierFactory`는 설정값을 받아 적절한 구현체를 반환하는 단 하나의 역할만 한다.

```python
# classifier/factory.py
class ClassifierFactory:
    _classifiers: Dict[str, Type[BaseClassifier]] = {
        'openai': OpenAIClassifier,
        'gemini': GeminiClassifier
    }

    @classmethod
    def create_classifier(cls, settings=None) -> BaseClassifier:
        if isinstance(settings, dict):
            settings = Settings.from_dict(settings)
        elif settings is None:
            settings = Settings()

        classifier_class = cls._classifiers.get(settings.OUTFITAI_PROVIDER.lower())
        if not classifier_class:
            raise ValueError(f"Invalid API provider: {settings.OUTFITAI_PROVIDER}")

        return classifier_class(settings)
```

구현체 목록은 `_classifiers` 딕셔너리에서 관리한다. 새 프로바이더를 추가할 때 이 딕셔너리에 한 줄을 추가하고 대응하는 서브클래스를 구현하면 된다. 팩토리 메서드 본문은 전혀 수정하지 않아도 된다.

## 호출 코드의 변화

이 구조가 실제로 어떤 차이를 만드는지 보자. 라이브러리 사용자 입장에서 프로바이더를 바꾸는 것은 환경 변수 하나를 변경하는 것으로 끝난다.

```python
# 사용 코드는 프로바이더에 완전히 무관하다
classifier = ClassifierFactory.create_classifier()

result = await classifier.classify_single("path/to/image.jpg")
```

환경 변수 설정만 바꾸면 내부 동작이 달라진다.

```bash
# OpenAI 사용
OUTFITAI_PROVIDER=openai
OPENAI_API_KEY=sk-...

# Gemini로 전환 — 호출 코드 변경 없음
OUTFITAI_PROVIDER=gemini
GEMINI_API_KEY=AI...
```

CLI도 동일하다. `outfitai image.jpg` 명령은 환경 변수에 설정된 프로바이더를 자동으로 사용한다.

## 이 설계가 가져온 것

**벤더 종속 위험 감소**: 특정 프로바이더의 API 정책 변경이나 장애가 발생했을 때 코드 수정 없이 다른 프로바이더로 전환할 수 있다.

**비교 실험의 용이성**: 동일한 이미지 셋에 대해 두 모델의 분류 결과를 비교하는 스크립트를 작성할 때, 프로바이더만 바꿔가며 동일한 인터페이스로 호출할 수 있다.

**확장 비용 최소화**: 세 번째 프로바이더(예: Anthropic의 Claude)를 추가하려면 `BaseClassifier`를 상속한 구현체를 작성하고, `_classifiers`에 한 줄을 추가하면 된다. 기존 코드를 수정하지 않아도 된다.

## 마치며

단일 프로바이더로 시작한 구조에 `if/elif`를 쌓아가는 대신, 추상 클래스와 팩토리 패턴을 도입해 확장에 열려 있는 구조를 만들었다. 구현 비용은 다소 높아지지만, 두 번째 프로바이더를 추가하는 시점에 이미 그 비용을 회수한다.

다음 글에서는 로컬 파일과 URL이라는 서로 다른 입력 방식을 단일 인터페이스로 통합한 **이미지 입력 레이어 설계**를 다룬다.
