---
title: "Google Maps API"
date: 2023-10-11
categories: [Project, etc_p]
tags: [til, api]
---

# 개요

구글 맵스(google maps) API는 다양한 지도와 지리위치 기능을 개발자들이 자신들의 애플리케이션이나 웹사이트, 서비스 등에 적용하도록 구글에서 제공하는 웹서비스와 도구들을 일컫는다.

# 주요 기능

- **Mapping Services**
  - 상호작용이 가능한 지도
- **Geocoding**
  - 주소나 장소명을 지리 좌표(geographical coordinates)(위도, 경도)로 변환하는 과정
- **Reverse Geocoding**
  - 지오코딩의 반대 개념.
  - 경위도 좌표를 사람이 읽을 수 있는 주소로 변환하는 과정
- **Directions and Routing**
  - 경로 및 네비게이션 서비스
- **Place Search**
  - 사용자가 특정 기준에 맞게 장소, 사업체, 관심지점(POI)를 검색할 수 있다.
    - 관심지점(POI, points of interests)
      - 사람들이 특정 지역에서 관심있어 하는 장소나 랜드마크
- **Distance Matrix**
  - 여러 장소 사이의 거리나 이동 시간을 계산
- **Elevation Data**
  - 특정 장소의 고도 데이터
- **Street View**
  - 거리뷰 이미지
- **Traffic Data**
  - 실시간 교통 데이터

# 활용 방법

- Geolocation Services
  - 특정 지역의 지리 좌표(경위도)를 통해 장소 기반의 서비스를 제공할 수 있다.
- Geospatial Analysis
  ![Untitled](/assets/img/project/google-maps-api/Untitled.png)
  - 공간 데이터를 분석하여 도시 계획, 환경 모니터링, 시장 분석 등에 활용할 수 있다.
- Mapping and Navigation Apps
  - 네비게이션 앱이나 여행 앱, 장소 기반 서비스에 활용할 수 있다.
- Location-Based Marketing
  ![Untitled](/assets/img/project/google-maps-api/Untitled%201.png)
  - 사용자의 물리적 위치에 기반하여 맞춤 광고를 할 수 있다.
- Data Visualization
  ![Untitled](/assets/img/project/google-maps-api/Untitled%202.png)
  [미국의 지역별 최저 임금 미만 노동자를 시각화한 자료]
  - 지도상에 데이터를 효과적으로 담아 시각화 할 수 있다.
- Geofencing and Alerts
  ![Untitled](/assets/img/project/google-maps-api/Untitled%203.png)
  - 지오펜싱(가상의 경계)을 설정하여 누군가 해당 위치를 나가거나 들어올 때 경보를 작동시킬 수 있다.
- Local Search and Recommendations
  - 지역 식당이나 장소에 대한 추천을 받을 수 있다.
- Real-Time Location Tracking
  - 차량 또는 사람에 대한 실시간 위치 추적에 활용할 수 있다.
- Traffic Analysis
  - 실시간 또는 누적된 교통 데이터를 분석할 수 있다.
- Custom Map Creation
  - 관광업, 이벤트 계획, 지리 조사 등에 사용할 지도를 직접 제작할 수 있다.

# 참고

- content
  - [https://github.com/googlemaps/google-maps-services-python](https://github.com/googlemaps/google-maps-services-python)
  - [https://blog.hubspot.com/website/google-maps-api](https://blog.hubspot.com/website/google-maps-api)
  - [https://www.ancoris.com/blog/do-more-with-google-maps-apis](https://www.ancoris.com/blog/do-more-with-google-maps-apis)
- images
  - [https://www.safegraph.com/guides/geospatial-data-analytics](https://www.safegraph.com/guides/geospatial-data-analytics)
  - [https://medium.com/@dunstconsulting/what-do-you-need-to-know-about-location-based-marketing-20075d62d6be](https://medium.com/@dunstconsulting/what-do-you-need-to-know-about-location-based-marketing-20075d62d6be)
  - [https://www.tableau.com/learn/articles/interactive-map-and-data-visualization-examples](https://www.tableau.com/learn/articles/interactive-map-and-data-visualization-examples)
  - [https://www.areusdev.com/what-is-geo-fencing-and-why-so-many-companies-are-starting-to-use-its-advantages/](https://www.areusdev.com/what-is-geo-fencing-and-why-so-many-companies-are-starting-to-use-its-advantages/)
