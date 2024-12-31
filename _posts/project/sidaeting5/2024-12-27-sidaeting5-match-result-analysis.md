---
title: "로지스틱 회귀 분석을 통한 시대팅 매칭 결과 분석"
date: 2024-12-27T08:00:00.000Z
categories: [Project, 시대팅5]
tags: [data-analysis, logistic-regression]
math: true
---

## 서론

이 글에서는 시대팅 시즌5의 매칭 결과에 영향을 미친 주요 요인들을 분석한 내용을 다룬다. 특히 이번 학기 고급프로그래밍 강의에서 배운 로지스틱 회귀를 활용하여 분석을 진행하였는데, 이를 통해 알고리즘 개선을 위한 유용한 인사이트를 도출하고자 했다.

## 로지스틱 회귀 분석

### 기본 개념

**로지스틱 회귀(Logistic Regression)**는 **종속 변수가 이진 값**을 가질 때 사용하는 회귀 분석 기법이다. 로지스틱 회귀는 선형 회귀를 기반으로 하지만, 결과를 확률 값으로 변환하기 위해 **로지스틱 함수(Sigmoid function)**를 사용한다. 로지스틱 함수는 다음과 같은 형태를 가진다.

$$
\sigma(z) = \frac{1}{1 + e^{-z}}
$$

여기서 $z$는 선형 함수로, 보통 독립 변수들의 가중치 합으로 표현된다. 이 확률 값을 바탕으로 특정 사건이 일어날 확률을 예측할 수 있으며, 결과값은 0과 1 사이에 위치한다.

### 선택 이유

![biniary classification](/assets/img/project/sidaeting5/07-match-result-analysis/binary-classification-example.png)
_이진 분류 예시_

이 분석 기법을 선택한 이유는 **타겟 변수(매칭 성공 여부)가 이진 분류(성공 또는 실패)**였기 때문이다. 로지스틱 회귀는 선형 회귀와 유사하지만 출력값이 확률로 변환되기 때문에 이진 분류 문제에 적합하다. 또한, 변수 간의 관계를 명확하게 해석할 수 있어 매칭 성공률에 영향을 미치는 요인들을 파악하는 데 강점을 가진다.

### 분석 데이터 소개

분석에 사용한 데이터는 **1대1 미팅을 신청한 남성** 유저들의 정보이다(여성은 전원 매칭되어 제외). 이 데이터에서 매칭이 성공했는지 여부(`is_matched`)는 타겟 변수로, 팀 ID(`team_id`), 나이(`age`), 키(`height`)는 독립 변수로 설정하였다.

### 분석 과정

먼저, 데이터를 불러온 뒤 확인하고자 하는 변수를 선택했다. 이후 데이터를 훈련용 데이터와 테스트용 데이터로 나누고, `statsmodels`의 `Logit` 함수를 사용하여 로지스틱 회귀 모델을 학습시켰다. 마지막으로 회귀 분석 결과를 출력하여 변수들의 영향도를 확인했다.

```python
import pandas as pd
import statsmodels.api as sm
from sklearn.model_selection import train_test_split

df = pd.read_csv('./data/single_male.csv')

# 주요 변수 선택
features = ['team_id', 'age', 'height']
target = 'is_matched'

X = df[features]
y = df[target]

# 모델 학습을 위한 데이터 준비
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# statsmodels 로지스틱 회귀
X_train_transformed = sm.add_constant(X_train)
X_test_transformed = sm.add_constant(X_test)

# 로지스틱 회귀 모델 학습
logit_model = sm.Logit(y_train, X_train_transformed)
result = logit_model.fit()

# 결과 출력
print(result.summary())
```

### 분석 결과

로지스틱 회귀 모델을 사용하여 매칭 성공률에 영향을 미치는 요인들을 분석한 결과는 다음과 같다.

```
Optimization terminated successfully.
         Current function value: 0.318716
         Iterations 8
                           Logit Regression Results                           
==============================================================================
Dep. Variable:             is_matched   No. Observations:                  432
Model:                          Logit   Df Residuals:                      428
Method:                           MLE   Df Model:                            3
Date:                Tue, 10 Dec 2024   Pseudo R-squ.:                  0.4980
Time:                        09:06:03   Log-Likelihood:                -137.69
converged:                       True   LL-Null:                       -274.28
Covariance Type:            nonrobust   LLR p-value:                 6.329e-59
==============================================================================
                 coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------
const        -55.4298      7.937     -6.984      0.000     -70.986     -39.874
team_id        0.0090      0.001      9.981      0.000       0.007       0.011
age            0.0484      0.064      0.752      0.452      -0.078       0.174
height         0.2587      0.040      6.401      0.000       0.179       0.338
==============================================================================
```

**주요 변수 해석**

- **팀 ID (`team_id`)**: 팀 ID는 매칭 성공률에 중요한 영향을 미쳤다. 회귀 계수(coef)는 0.009로, 팀 ID가 증가할수록 매칭 성공 확률이 증가한다는 것을 의미한다. p-value는 0.000으로 매우 유의미한 변수이다.
- **키 (`height`)**: 키는 매칭 성공률에 큰 영향을 미쳤다. 회귀 계수(coef)는 0.2587로, 키가 증가할수록 매칭 성공 확률이 높아진다. p-value는 0.000으로 매우 유의미한 변수이다.
- **나이 (`age`)**: 나이는 매칭 성공률에 유의미한 영향을 미치지 않았다. 회귀 계수(coef)는 0.0484로 미미한 영향을 주지만, p-value가 0.452로 통계적으로 유의미하지 않다.

**결과 해석**

로지스틱 회귀 분석을 통해, **팀 ID와 키가 매칭 성공률에 중요한 영향**을 미친다는 결과를 얻을 수 있었다. 특히 팀 ID는 매칭 성공률과의 상관관계가 강하게 나타났는데, 이는 신청 시점이 성공률에 영향을 미쳤음을 시사한다. 키 또한 매칭 성공률에 중요한 요소로 작용했는데, 여성 참가자가 핵심 선호 항목으로 키를 선택한 것이 원인으로 보인다.

## 매칭 성공률 시각화

로지스틱 회귀 분석 결과 유의미한 변수로 밝혀진 팀 ID와 키에 대해 시각화를 진행하였다.

### 팀 ID

![matching_success_rate_by_team_id.png](/assets/img/project/sidaeting5/07-match-result-analysis/matching_success_rate_by_team_id.png)

팀 ID를 구간별로 살펴본 결과, 뒤로 갈수록 매칭 성공률이 가파르게 상승한다는 특징을 발견했다. 즉, 신청기간 후반부에 신청한 유저들의 성공률이 초반부에 비해 월등히 높았다.

### 키

![matching_success_rate_by_height.png](/assets/img/project/sidaeting5/07-match-result-analysis/matching_success_rate_by_height.png)

키에 따른 성공률을 살펴본 결과, 180cm 근방의 유저들의 성공률이 높게 나타났다. 특히 180~184cm 구간에서 매칭 성공률이 가장 높았다.

## 결론

이번 로지스틱 회귀 분석은 매칭 성공률에 영향을 미치는 주요 변수들을 확인하고, 이를 기반으로 매칭 알고리즘의 성능 개선 방향을 탐색하는 데 의미가 있었다. 특히 **팀 ID와 키는 매칭 성공률과의 상관관계가 뚜렷했으며**, 이를 통해 신청 시점과 키를 고려한 매칭 정책 설정의 필요성을 느꼈다.

분석을 통해 이번 매칭 알고리즘의 문제를 알 수 있었다. **매칭 성공 유저가 신청기간 후반부에 편중**된 현상이었는데, 이는 Hospital-Resident 알고리즘의 입력 데이터 순서 의존성과 관련있는 것으로 보인다. 이러한 문제는 매칭의 공정성 측면에서 개선의 여지가 있다.

이번 분석 과정은 이번 학기에 배운 통계 기법인 로지스틱 회귀를 실제 데이터에 적용해보는 소중한 기회였다. 이 과정을 통해 데이터 기반 의사결정의 중요성을 실감했고, 분석 결과를 앞으로의 서비스 개선 방향에 반영할 수 있었다.

## 참고 자료

- [Binary Classification, Explained - Sharp Sight](https://www.sharpsightlabs.com/blog/binary-classification-explained/)
