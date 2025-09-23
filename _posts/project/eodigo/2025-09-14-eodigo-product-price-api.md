---
title: "어디GO 상품 가격 탐색 API 개발기"
date: 2025-09-14T09:00:00.000Z
categories: [Project, 어디GO]
tags: [spring-boot, api]
---

> 이 글은 어디GO의 상품 가격 탐색 API를 완성하기까지의 여정을 다룬다. Mock API 구현부터 실제 비즈니스 로직을 구현하며 마주한 기술적 과제들, 그리고 예상치 못한 요구사항 변경에 대응했던 경험에 대해 이야기한다.

## 시작하며: API-First 전략

'어디GO'의 실질적인 개발 기간은 4주로, 짧은 기간동안 안드로이드 팀원과의 긴밀한 협업이 필수적이었다. 이러한 상황 속에서 'API-First' 개발 전략을 따르기로 했다. 이는 백엔드 개발자가 로직을 완성할 때까지 안드로이드 개발자가 기다리는 대신, 먼저 서버와 클라이언트 간의 API 명세를 정의하고, 해당 형식의 Mock 데이터를 반환하는 API를 우선적으로 제공하는 것이다. 이로 인해 안드로이드 개발자가 UI 개발과 네트워크 로직 구현을 즉시 시작할 수 있게 하여, 전체 개발 기간을 단축할 수 있다.


## API 설계

우선 내가 담당한 API는 다음과 같다.

|![hierarchy-1](/assets/img/project/eodigo/product-price-api/product_hierarchy-1.png)|![hierarchy-2](/assets/img/project/eodigo/product-price-api/product_hierarchy-2.png)|![ranking-1](/assets/img/project/eodigo/product-price-api/product_ranking-1.png)|![ranking-2](/assets/img/project/eodigo/product-price-api/product_ranking-2.png)|![trends](/assets/img/project/eodigo/product-price-api/product_trends.png)
|1-1|1-2|2-1|2-2|3-1|

- **1. 전체 상품 목록 조회 (`GET /api/v1/products/hierarchy`)**
    - 사용자가 선택할 농수산물의 전체 목록을 부류-품목-품종의 계층 구조로 제공한다.
- **2. 상품별 가격 랭킹 조회 (`GET /api/v1/products/{productId}/rankings`)**
    - 최신 데이터 기준으로 특정 상품의 지역별 가격을 저렴한 순으로 정렬하여 제공한다.
- **3. 상품별 가격 추이 조회 (`GET /api/v1/products/{productId}/trends`)**
    - 특정 상품의 최근 10년간 연도별 전국 평균 가격 정보를 제공한다.

제일 먼저 Notion에 API 명세서를 정의하였다. 전체 상품 목록 조회(`/hierarchy`), 가격 추이 조회(`/trends`), 가격 랭킹 조회(`/rankings`) 세 가지 핵심 API의 요청/응답 형식을 구체적으로 명시하고 프론트엔드 팀원들에게 공유했다.

![notion_api_doc.png](/assets/img/project/eodigo/product-price-api/notion_api_doc.png)
_API 명세서 공유_

## Mock API 구현

그 후 실제 DB 연동이나 비즈니스 로직 없이, 오직 이 명세만을 기반으로 API의 뼈대를 구현했다. `ProductService` 클래스 내부에 하드코딩된 Mock 데이터를 생성하고, `ProductController`는 이 데이터를 받아 약속된 DTO 형식으로 반환하도록 했다.

**주요 코드**  
```kotlin
// ProductService.kt
@Service
class ProductService {
    // TODO: 실제 Repository를 주입받아 DB 조회 로직으로 변경

    fun getProductRanking(productId: Long): ProductRankingResponse {
        // --- Mock 데이터 생성 시작 ---
        val rankingData =
            listOf(
                RegionalPriceInfo(rank = 1, regionName = "천안", price = 2850),
                RegionalPriceInfo(rank = 2, regionName = "김해", price = 2900),
                // ...
            )
        // --- Mock 데이터 생성 끝 ---

        return ProductRankingResponse(
            productId = productId,
            productName = "등심",
            surveyDate = LocalDate.now(),
            ranking = rankingData,
        )
    }
    // getProductHierarchy(), getProductTrend() ...
}
```

이렇게 해서 실제 동작하는 엔드포인트가 만들어졌다. 곧바로 Springdoc OpenAPI(Swagger UI)를 적용하여 안드로이드 팀에게 API 문서를 공유했다. 이제 안드로이드 팀은 이 Mock API를 호출하며 화면 개발을 진행하고, 나는 데이터 수집 파이프라인과 실제 비즈니스 로직 구현에 집중할 수 있게 되었다.

![slack_mock_api.png](/assets/img/project/eodigo/product-price-api/slack_mock_api.png)
_Swagger UI 명세서 공유_

## API 재설계

실제 로직 구현에 앞서 초기 API 명세를 검토하던 중, 클라이언트 입장에서 불필요하거나 비효율적인 필드가 존재함을 발견했다. 구현이 더 진행되기 전에 바로잡는 것이 옳다고 판단하여 팀 스레드를 통해 개선안을 제안했다.

![slack_api_discussion.png](/assets/img/project/eodigo/product-price-api/slack_api_discussion.png)
_API 명세 변경 제안_

이후 팀원들과 논의를 거쳐 아래와 같이 최종적으로 API 명세를 확정하게 되었다.

**주요 변경 사항:**

1. **응답 데이터 간소화:** 요청에 이미 포함된 `productId`나, 배열의 순서로 대체 가능한 `rank` 필드 등 중복되거나 불필요한 정보를 제거하자.
2. **데이터 정렬 기준 통일:** 가격 추이 데이터(`annualData`)를 최신순이 아닌, 시간의 흐름을 직관적으로 파악할 수 있는 오름차순(과거 -> 최신)으로 변경하자.
3. **예외 처리 정책 명확화:** 데이터가 아예 없는 경우는 `404 Not Found` 에러로, 일부만 있는 경우는 `200 OK`와 빈 배열/필터링된 데이터로 응답하는 기준을 명확히 하자.

**최종 확정된 응답 구조 예시 (가격 랭킹 조회)**

```json
{
  // "productId": 78, (제거)
  "productName": "사과",
  // "surveyDate": "2025-08-20", (제거)
  "ranking": [
    {
      // "rank": 1, (제거)
      "regionName": "부산",
      "price": 3280
    }
    // ...
  ]
}
```

## 비즈니스 로직 구현

개선된 명세가 확정되고, Spring Batch로 구현한 데이터 수집 파이프라인이 DB에 데이터를 채우기 시작했다. 이제 Mock 로직을 실제 DB 조회 로직으로 대체할 차례였다. 이 과정에서 두 가지 기술적 문제를 마주했다.

### 문제 1: Entity에서 계층형 DTO로의 변환

전체 상품 목록 조회 API(`GET /api/v1/products/hierarchy`)는 `부류-품목-품종`의 3단계 또는 `부류-품목`의 2단계 계층 구조로 상품 목록을 제공해야 했다. 하지만 DB에 저장된 `Product` 엔티티는 이 모든 정보를 평면적으로 가지고 있었다.

**product 테이블**

![db_product_table.jpg](/assets/img/project/eodigo/product-price-api/db_product_table.jpg)

**Product 엔티티**  
```kotlin
@Entity
class Product(
    val categoryName: String, // 부류
    val categoryCode: String,
    val itemName: String,     // 품목
    val itemCode: String,
    val kindName: String?,    // 품종 (Nullable)
    // ...
)
```

**응답 데이터**

```json
[
  {
    "categoryName": "채소류",
    "categoryCode": "200", // UI 아이콘 매칭용
    "items": [
      {
        "itemName": "배추",
        "kinds": [ // '품종'이 있는 3단계 계층
          { "productId": 42, "kindName": "봄" },
          { "productId": 45, "kindName": "여름(고랭지)" }
        ]
      }
    ]
  },
  {
    "categoryName": "과자류",
    "categoryCode": "700",
    "items": [
      { // '품종'이 없는 2단계 계층
        "productId": 110,
        "itemName": "새우깡"
        // 'kinds' 필드 자체가 없음
      }
    ]
  }
]
```

JPA Repository에서 조회한 `List<Product>`를 API 명세에 맞는 계층형 DTO `List<CategoryInfo>`로 변환해야 했다. 이를 위해 Kotlin의 강력한 컬렉션 처리 함수들을 활용했다.

**핵심 아이디어**

1. 모든 Product를 `categoryCode` 기준으로 1차 그룹화한다. (`Map<String, List<Product>>`)
2. 그룹화된 각 카테고리 내에서, 다시 `itemCode` 기준으로 2차 그룹화한다.
3. 최종적으로 각 품목 그룹 내의 상품 목록을 분석하여, 품종이 있는지(`kindName != null`) 여부에 따라 2단계/3단계 `ItemInfo` DTO를 생성한다.

**주요 코드**  
```kotlin
// ProductService.kt
fun getProductHierarchy(): List<CategoryInfo> {
    val allProducts = productRepository.findAllByOrderByIdAsc()

    return allProducts
        .groupBy { it.categoryCode } // 1. 부류(category)로 그룹화
        .map { (categoryCode, productsInCategory) ->
            CategoryInfo(
                categoryName = productsInCategory.first().categoryName,
                categoryCode = categoryCode,
                items = productsInCategory
                    .groupBy { it.itemCode } // 2. 품목(item)으로 그룹화
                    .map { (_, productsInItem) ->
                        // 3. 품종(kind) 유무에 따라 분기 처리
                        if (productsInItem.size == 1 && productsInItem.first().kindName == null) {
                            // 2단계 계층 DTO 생성
                            ItemInfo(itemName = productsInItem.first().itemName, productId = productsInItem.first().id, kinds = null)
                        } else {
                            // 3단계 계층 DTO 생성
                            ItemInfo(itemName = productsInItem.first().itemName, productId = null, kinds = productsInItem.map { ... })
                        }
                    }
            )
        }
}
```

### 문제 2: 데이터 부재 시 NoSuchElementException 발생

단위 테스트를 작성하던 중, 가격 추이(`trends`) 데이터가 아예 없는 상품을 조회하는 테스트 케이스에서 예상치 못한 에러를 만났다. `ProductTrendNotFoundException`을 기대했지만, 실제로는 `NoSuchElementException`이 발생했다.

```json
{
    "status": 500,
    "code": "C003",
    "message": "서버 내부에서 오류가 발생했습니다."
}
```

```
2025-08-19T23:55:16.867+09:00 ERROR 89941 --- [eodigo] [nio-8080-exec-2] c.e.c.exception.GlobalExceptionHandler   : Unhandled Exception

java.util.NoSuchElementException: List is empty.
  at kotlin.collections.CollectionsKt___CollectionsKt.last(_Collections.kt:418) ~[kotlin-stdlib-1.9.25.jar:1.9.25-release-852]
  at com.eodigo.domain.product.service.ProductService.getProductTrend(ProductService.kt:102) ~[main/:na]
  at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
  at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77) ~[na:na]
  at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
  at java.base/java.lang.reflect.Method.invoke(Method.java:569) ~[na:na]
  at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:360) ~[spring-aop-6.2.9.jar:6.2.9]
  at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:196) ~[spring-aop-6.2.9.jar:6.2.9]
```

**원인**  
DB 조회 결과가 빈 리스트(`emptyList`)일 경우를 고려하지 않고, `allAnnualPrices.first()`를 호출하여 첫 번째 원소에 접근하려고 시도했기 때문이었다. 예외 처리 로직에 도달하기 전에 프로그램이 비정상 종료된 것이다.

```kotlin
// 수정 전 코드
fun getProductTrend(productId: Long): ProductTrendResponse {
    val allAnnualPrices = repository.findByProductIdOrderBySurveyYearAsc(productId)

    // allAnnualPrices가 비어있으면 여기서 에러 발생!
    val latestYear = allAnnualPrices.last().surveyYear

    // 이 코드는 실행되지 못한다.
    if (last10YearsPrices.isEmpty()) {
        throw ProductTrendNotFoundException()
    }
    // ...
}
```

**해결 방안**  
DB를 조회한 직후, 리스트가 비어있는지(`isEmpty()`)를 가장 먼저 확인하는 방어 코드를 추가했다. 이 순서 변경만으로 코드는 훨씬 더 안정적으로 동작하게 되었다.

```kotlin
// 수정 후 코드
fun getProductTrend(productId: Long): ProductTrendResponse {
    val allAnnualPrices = repository.findByProductIdOrderBySurveyYearAsc(productId)

    // [수정] 데이터 존재 여부를 가장 먼저 확인
    if (allAnnualPrices.isEmpty()) {
        throw ProductTrendNotFoundException()
    }

    // 여기서는 리스트가 비어있지 않음이 보장된다.
    val latestYear = allAnnualPrices.last().surveyYear
    // ...
}
```

### 문제 3: 테스트 코드의 Null ID로 인한 NullPointerException

상품 목록 계층 조회 로직을 테스트(`getProductHierarchy_Success`)하는 과정에서 `NullPointerException`이 발생했다.

**원인**  
테스트를 위해 생성한 Product 객체의 id는 null 상태였지만, DTO 변환 로직에서 `product.id!!` 와 같이 id가 null이 아님을 단정했기 때문이었다.

```kotlin
// ProductService.kt
KindInfo(
    productId = product.id!!, // 문제 지점
    kindName = product.kindName ?: "알 수 없음"
)
```

**해결 방안**  
이 문제를 해결하기 위해 서비스 로직을 다음과 같이 개선했다.

```kotlin
// ProductService.kt
KindInfo(
    productId =
        requireNotNull(product.id) {
            "Persisted Product.id must not be null"
        },
    kindName =
        product.kindName ?: UNKNOWN_KIND_NAME,
)
```

위험한 `!!` 연산자 대신 `requireNotNull`을 사용하여 "id는 null일 수 없다"는 비즈니스 규칙을 명시하고, 위반 시 `IllegalStateException`이 발생하도록 변경했다.

## 긴급 요구사항: 상품 검색 API 개발

|![product_search-1.png](/assets/img/project/eodigo/product-price-api/product_search-1.png)|![product_search-2.png](/assets/img/project/eodigo/product-price-api/product_search-2.png)|![product_search-3.png](/assets/img/project/eodigo/product-price-api/product_search-3.png)|

어디GO는 원하는 상품을 카테고리에서 선택하는 기능 외에도, 검색창에서 검색하는 기능도 제공한다. 당초 프론트엔드 팀원과 이 기능의 구현 방식에 대해 논의를 했을때 클라이언트 사이드에서 처리하는 것으로 결론이 났었다.

![slack_search_feature_discussion](/assets/img/project/eodigo/product-price-api/slack_search_feature_discussion.jpg)
_검색 기능 구현 방식 논의_

하지만 프로젝트 마감 직전에 예상치 못한 변수가 발생했다. 마감을 4일 앞둔 날 안드로이드 팀원으로부터 기한내 구현이 어려울 것 같다는 연락을 받았다. 해당 시점에 나는 백엔드의 주요 작업을 마무리하여 시간적인 여유가 있었기에 검색 기능은 서버에서 API를 제공하는 방식으로 변경하였다.

![slack_issue_search_api.png](/assets/img/project/eodigo/product-price-api/slack_issue_search_api.png)
_검색 기능 관련 이슈_

즉시 `GET /api/v1/products/search` 엔드포인트를 설계하고 구현에 착수했다.

**주요 코드**

```kotlin
// ProductController.kt
@GetMapping("/search")
fun searchProducts(@RequestParam("keyword") keyword: String): ResponseEntity<List<ProductSearchResponse>> {
    val searchResult = productService.searchProducts(keyword)
    return ResponseEntity.ok(searchResult)
}

// ProductService.kt
fun searchProducts(keyword: String): List<ProductSearchResponse> {
    return productRepository.findByNameContaining(keyword).map { product ->
        ProductSearchResponse.from(product)
    }
}

// ProductRepository.kt
interface ProductRepository : JpaRepository<Product, Long> {
    fun findByNameContaining(keyword: String): List<Product>
}
```

### Containing 쿼리 관련 문제

한 가지 문제가 있었다. 검색 기능은 Spring Data JPA의 `findByNameContaining`을 사용하여 구현되었는데, 만약 사용자가 아무것도 입력하지 않은 채(`keyword=""`)로 검색을 요청하면, `findByNameContaining`은 SQL의 `LIKE '%%'`와 동일하게 동작하여 **DB의 모든 상품을 반환**했다. 이는 불필요한 부하를 유발할 수 있었다.

이 문제를 해결하기 위해, Service 계층에 검색어가 비어있거나 공백으로만 이루어진 경우 DB를 조회하지 않고 즉시 **빈 리스트를 반환**하는 방어 코드를 추가했다.

```kotlin
// ProductService.kt
fun searchProducts(keyword: String): List<ProductSearchResponse> {
    // [추가] 비어있는 검색어에 대한 처리
    if (keyword.isBlank()) {
        return emptyList()
    }

    return productRepository.findByNameContaining(keyword).map { product ->
        ProductSearchResponse.from(product)
    }
}
```

## 마치며

'어디GO'의 상품 API 개발 과정은 API-First 전략을 통한 협업, 소통을 통한 설계 개선, 그리고 테스트 과정에서 발견된 엣지 케이스 해결 등 다양한 경험을 할 수 있는 기회였다. 또한 예상치 못한 요구사항 변경에 대응하며 방어적 코드 설계의 중요성도 배울 수 있었다.
