---
title: "OutfitAI LLM 출력 일관성 확보: 고정 단어 목록과 JSON 모드 활용"
date: 2025-03-07T09:00:00.000Z
categories: [Project, OutfitAI]
tags: [prompt-engineering]
---

## 배경

멀티모달 AI 모델은 이미지를 보고 자유롭게 텍스트를 생성한다. 이 자유로움은 장점이지만, 서비스에서 AI 출력을 **데이터베이스에 저장하거나 후속 로직에서 활용**하려면 오히려 독이 된다.

![clothes_example](/assets/img/project/outfitai/schema-design/clothes_example.webp)

OutfitAI에서 분류해야 하는 항목은 색상, 카테고리, 드레스코드, 시즌이다. 모델에게 단순히 "이 옷의 색상을 알려줘"라고 물으면 어떻게 될까?

```
"navy blue"
"dark navy"
"indigo"
```

동일한 옷 사진에 대해서도 이런 식으로 제각각인 응답이 돌아온다. 이를 그대로 저장하면 같은 옷이 앱 내 옷장에서 전혀 다른 색상으로 분류되고, 색상 기반의 필터링이나 추천 기능은 동작 자체를 할 수 없게 된다.

이 글에서는 자유 텍스트 출력을 서비스에서 바로 활용할 수 있는 구조화된 JSON으로 제약하기까지의 설계 과정을 다룬다.

## 실패한 첫 번째 시도: 헥스 코드

최초 구현에서는 색상을 **헥스 코드로 받도록** 설계했다. `#1A2B3C`처럼 정확한 색상값을 받으면 UI에서 바로 활용할 수 있을 것 같았다.

결과는 기대와 달랐다. 모델이 반환하는 헥스 코드는 같은 옷에 대해서도 매번 달라졌다. `#1C2D4A`, `#1E3055`, `#172848` 처럼 미묘하게 다른 값들이 나왔고, 이를 기반으로 "비슷한 색상의 옷 찾기" 같은 기능을 구현하는 것이 불가능했다.

근본적인 문제는 **색상 공간이 너무 넓다**는 것이었다. 사람이 옷을 고를 때 실제로 사용하는 색상 범주는 "파란색", "검정", "베이지" 정도의 큰 단위이지, RGB 값이 아니다. 이 도메인에서 의미 있는 색상 분류는 제한된 단어 목록에 맵핑하는 것이 맞다는 결론에 도달했다.

## 스키마 우선 설계

핵심 원칙은 **출력 형식을 먼저 정의하고, 모델이 그 형식에 맞춰 응답하도록 제약**하는 것이다. 이를 위해 세 가지를 동시에 설계했다.

1. **분류 단어 목록**: 각 항목에서 허용하는 값을 고정된 리스트로 정의
2. **JSON 강제 출력**: API 레벨에서 모델이 반드시 JSON으로 응답하도록 설정
3. **응답 검증**: 모델 출력이 허용된 값 안에 있는지 애플리케이션에서 한 번 더 검증

### 분류 단어 목록 설계

각 항목의 단어 목록은 **서비스의 실제 사용 목적을 기준**으로 설계했다.

**색상 (11종)**

```python
color_values = [
    "white", "gray", "black", "red", "orange",
    "yellow", "green", "blue", "indigo", "purple", "other"
]
```

일상적으로 옷을 묘사할 때 쓰는 기본 색상 범주를 기준으로 삼았다. 분홍(pink)은 red에, 하늘색(sky blue)은 blue에 흡수시키는 방식으로 범주를 단순화했다. 유사한 색상들을 하나의 범주로 묶더라도 필터링과 추천 기능에는 충분히 유효하다.

**카테고리 (9종)**

```python
category_values = [
    "tops", "bottoms", "outerwear", "dresses",
    "shoes", "bags", "hats", "accessories", "other"
]
```

이 분류에서는 **사용자가 옷을 등록하는 목적**을 고려했다. 단순히 의류를 분류하는 것이 아니라, 앱에서 "오늘 뭘 입을까"를 결정하는 데 쓰이는 단위다. 따라서 코디를 구성하는 시각적 단위(상의/하의/아우터)와 패션 소품(신발/가방/모자)을 분리해 정의했다.

**드레스코드 (9종)**

```python
dress_code_values = [
    "casual wear", "business attire", "campus style", "date night outfit",
    "travel wear", "wedding attire", "loungewear", "resort wear", "other"
]
```

옷의 속성 중 모델이 가장 주관적으로 판단하는 항목이다. "이 재킷은 캐주얼인가, 비즈니스인가"는 사람마다 다를 수 있다. 때문에 정의가 모호한 중간 범주는 의도적으로 두지 않고, **대표성 있는 상황** 위주로 목록을 구성했다. 어느 범주에도 속하기 어려운 경우를 위한 `other`도 포함했다.

**시즌 (4종, 복수 선택)**

```python
season_values = ["spring", "summer", "fall", "winter"]
```

다른 항목과 달리 시즌만 **배열로 받는다**. 얇은 가디건처럼 봄과 가을 모두에 어울리는 옷이 실제로 존재하기 때문이다. 단일 값으로 제약하면 이런 옷을 절반의 계절에만 추천하는 오류가 생긴다.

### 프롬프트 설계

단어 목록이 확정되면 프롬프트는 이를 그대로 명시하는 방식으로 단순하게 구성한다. 이 프롬프트는 `BaseClassifier._create_prompt()`에서 생성되며 OpenAI, Gemini 구현체 모두 동일하게 사용한다.

```python
# base.py
def _create_prompt(self) -> str:
    return f"""
    Analyze the clothing item in the image and classify it according to these rules.
    Return a JSON object with these keys:
    - 'color': 1 value from {self.color_values}
    - 'category': 1 value from {self.category_values}
    - 'dress_code': 1 value from {self.dress_code_values}
    - 'season': 1+ values from {self.season_values} (array)
    """
```

모델에게 맥락을 장황하게 설명하는 대신, **허용되는 값의 목록을 직접 나열**한다. "이 중에서만 골라라"는 제약을 명시적으로 주는 것이 가장 효과적이었다.

### JSON 강제 출력

프롬프트로 JSON을 요청해도, 모델이 마크다운 코드 블록으로 감싸거나 부연 설명을 덧붙이면 파싱이 실패한다. 이를 API 레벨에서 막는다.

OpenAI는 `response_format` 파라미터를 사용한다.

```python
# openai_classifier.py
response = await self.client.chat.completions.create(
    model=self.settings.OPENAI_MODEL,
    messages=[...],
    max_tokens=self.settings.OPENAI_MAX_TOKENS,
    response_format={"type": "json_object"}  # JSON 모드 강제 활성화
)
```

Gemini는 `response_mime_type`으로 동일한 효과를 낸다.

```python
# gemini_classifier.py
response = self.client.models.generate_content(
    model=self.settings.GEMINI_MODEL,
    contents=[self.prompt_text, image_part],
    config={
        'response_mime_type': 'application/json',  # JSON 응답 강제
    },
)
```

두 프로바이더 모두 API 레벨의 JSON 모드를 지원하며, 이를 활성화하면 모델은 유효한 JSON 외의 출력을 생성하지 않는다.

### 응답 검증 레이어

JSON으로 파싱에 성공했다고 해서 끝이 아니다. 모델이 JSON은 반환했지만 `color` 값으로 허용되지 않은 `"navy"` 를 넣거나, `season`을 배열이 아닌 문자열로 반환할 수도 있다.

`BaseClassifier._validate_response()`는 이를 애플리케이션 레벨에서 한 번 더 걸러낸다.

```python
# base.py
def _validate_response(self, data: Dict[str, Any]) -> None:
    required_keys = ["color", "category", "dress_code", "season"]

    for key in required_keys:
        if key not in data:
            raise ValidationError(f"Missing required key: {key}")

    if data["color"] not in self.color_values:
        raise ValidationError(f"Invalid category: {data['color']}")

    if data["category"] not in self.category_values:
        raise ValidationError(f"Invalid category: {data['category']}")

    if data["dress_code"] not in self.dress_code_values:
        raise ValidationError(f"Invalid dress_code: {data['dress_code']}")

    if not isinstance(data["season"], list):
        raise ValidationError("Season must be a list")

    for season in data["season"]:
        if season not in self.season_values:
            raise ValidationError(f"Invalid season: {season}")
```

이 검증 레이어가 있으면 `ValidationError`가 발생할 때 명확한 원인을 알 수 있다. 만약 특정 프로바이더나 모델에서 비허용 값이 자주 나온다면, 해당 케이스를 프롬프트 수정으로 피드백할 수 있다. 호출 코드에서 세분화된 예외를 처리할 수 있다는 점도 장점이다.

## 스키마의 변천

완성된 스키마가 처음부터 이 모습이었던 것은 아니다. 커밋 이력을 보면 몇 차례 수정이 있었다.

- **색상 헥스 코드 → 네임드 컬러**: 앞서 설명한 이유로 초기 헥스 코드 방식을 폐기하고 고정 단어 목록으로 변경했다.
- **분류 항목 수정**: 해당 서비스의 요구사항이 구체화되면서 드레스코드 항목이 실제 사용 시나리오에 맞게 조정되었다.
- **`dresscode` → `dress_code`**: JSON 키 명칭을 Python 변수 네이밍 컨벤션(snake_case)에 맞게 통일했다. 이 변경은 호출 코드에 영향을 주는 브레이킹 체인지였기 때문에 메이저 버전 업데이트와 함께 진행했다.

설계 초기에 도메인 요구사항을 충분히 파악하지 못한 채 구현에 착수하면 이런 수정이 생긴다. 특히 다른 서비스에서 라이브러리를 쓰고 있는 상황에서 스키마가 바뀌면 연쇄적인 변경이 필요하므로, JSON 키 이름과 허용 단어는 초기에 신중하게 정의하는 것이 중요하다.

## 마치며

AI 모델의 출력을 서비스에서 활용하려면 "잘 응답하길 바라는" 접근보다, **출력 공간을 명시적으로 제약**하는 설계가 필요하다. OutfitAI의 경우 세 가지 레이어를 조합해 이를 실현했다.

1. **단어 목록 고정**: 허용하는 값을 프롬프트에 직접 명시
2. **API 레벨 JSON 모드**: 프로바이더가 제공하는 구조화된 출력 기능 활용
3. **애플리케이션 레벨 검증**: 허용되지 않은 값이 내려왔을 때 명확한 예외 발생

다음 글에서는 이 분류기를 OpenAI와 Gemini 모두 지원하도록 확장하면서 도입한 **멀티 프로바이더 팩토리 패턴**을 다룬다.
