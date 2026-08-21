---
name: map_api_test
description: Google Maps Platform API 종합 테스트 주피터 노트북(google_maps_api_test.ipynb) 생성을 위한 요구사항, 아키텍처 명세, 입력/출력 속성 및 코드 생성 레시피 가이드.
---

# Google Maps Platform API 테스트 노트북 생성 스킬 (`map_api_test`)

본 스킬은 **Google Maps Platform API 종합 테스트 노트북**(`google_map/google_maps_api_test.ipynb`)을 구축, 검증 및 유지보수하기 위한 전체 요구사항, 아키텍처 표준, API별 입력/출력 속성 명세 및 코드 생성 가이드를 정의합니다.

---

## 🎯 1. 목적 및 핵심 목표 (Core Objectives)

`google_maps_api_test.ipynb` 노트북은 Google Maps Platform API의 실무 적용성과 기능적 완전성을 검증하기 위한 상용 수준의 대화형 테스트 스위트이자 기술 참조 문서입니다.

### 핵심 목표:
1. **입력(Input) 및 출력(Output) 속성의 전수 검증**: 모든 실무 파라미터(필터, 컴포넌트, 바이어스, 세션 토큰, 필드 마스크)를 적용하고, 반환되는 지오메트리, 주소 분해 컴포넌트, 리뷰, 영업시간, 접근성, 교통 모델 등 모든 속성을 파싱합니다.
2. **서울 및 대한민국 중심 데이터 현지화**: 강남파이낸스센터(GFC), 명동교자 본점, N서울타워, 코엑스, 서울시청, 롯데월드타워, 북한산 등 실제 서울 핵심 랜드마크 데이터를 활용합니다.
3. **최신 Places API (New / Modern v1) 집중 검증**: 텍스트 검색(`searchText`), 주변 검색(`searchNearby`), 와일드카드(`*`) 전체 상세 조회, 고객 리뷰(`reviews`) 및 AI 에디토리얼 요약(`editorialSummary`), 세션 토큰 기반 자동완성(`autocomplete`)을 전면 구현합니다.
4. **국내 규제 및 예외 상황 대응 (Resilience)**: 한국 공간정보 국외반출 제한으로 인한 국내 운전(`driving`) 경로 제약 시 대중교통(`transit`) 모드 지원 및 친절한 안내 메시지를 제공하고, 미활성화 API 호출 시 Google Cloud Console 활성화 링크를 안내합니다.
5. **구조화된 데이터 시각화**: `pandas.DataFrame` 표 형식 및 가독성 높은 JSON 포맷팅 헬퍼(`print_json`)를 통해 직관적으로 출력합니다.

---

## 🛠️ 2. 환경 및 의존성 명세 (Environment & Dependencies)

### 파이썬 런타임 및 필수 라이브러리
- **Python 버전**: 3.10 이상 (Python 3.12 권장)
- **필수 패키지**:
  - `googlemaps>=4.10.0`: 공식 Google Maps Python 클라이언트 SDK
  - `requests>=2.31.0`: 최신 Places API (New) 및 REST v1 엔드포인트 호출용
  - `pandas>=2.0.0`: 주소 컴포넌트, 리뷰, 거리 행렬 등의 표 형식 시각화
  - `python-dotenv>=1.0.0`: `.env` 환경변수 자동 로드
  - `ipykernel>=6.29.0`: Jupyter 커널 실행 지원

### 보안 및 시크릿 관리 수칙 (Security Guardrails)
- **API 키 하드코딩 절대 금지**: `python-dotenv`를 통해 `os.getenv("GOOGLE_MAPS_API_KEY")`로만 로드해야 합니다.
- **동적 대화형 입력 지원**: 환경변수가 없을 경우 `getpass.getpass()`를 통해 안전하게 입력을 유도합니다.
- **Git 커밋 방지**: `.env` 파일은 `.gitignore`에 등록되어야 하며, 출력 로그에는 마스킹된 키(`AIzaSy...xxxx`)만 표시합니다.

---

## 📋 3. 상세 API별 요구사항 및 실무 시나리오 명세

노트북은 아래 12개 API 기능에 대한 전용 마크다운 설명과 실행 코드 블록을 포함해야 합니다:

### 📍 1. Geocoding API (`gmaps.geocode`)
- **입력 파라미터 (Inputs)**:
  - `address`: 전체 도로명 주소 (예: `"서울특별시 강남구 테헤란로 152 강남파이낸스센터"`)
  - `components`: 필수 제한 필터 (예: `{"country": "KR", "postal_code": "06236"}`)
  - `region`: 국가 코드 바이어스 (`"kr"`)
  - `language`: 응답 언어 (`"ko"`)
- **출력 속성 (Outputs)**:
  - `formatted_address`: 표준 도로명 주소
  - `place_id`: 고유 장소 식별자
  - `geometry`: `location` (`lat`, `lng`), `location_type` (`ROOFTOP`, `RANGE_INTERPOLATED`), `viewport`
  - `address_components`: 우편번호, 행정구, 도로명, 동별 세부 분해 배열
- **실무 시나리오**: 국내 배송 주소 표준화 정제, 5자리 국가기초구역번호(우편번호) 검증, 지도 줌 뷰포트 자동 피팅.

### 🔄 2. Reverse Geocoding API (`gmaps.reverse_geocode`)
- **입력 파라미터 (Inputs)**:
  - `latlng`: 대상 좌표 튜플 `(37.50005, 127.0365)`
  - `result_type`: 특정 주소 유형 필터 (`['street_address', 'premise']`)
  - `location_type`: 위치 정밀도 필터 (`['ROOFTOP']`)
  - `language`: `"ko"`
- **출력 속성 (Outputs)**:
  - 건물/도로명 단위부터 동(역삼동)/구(강남구)/시(서울특별시) 단위까지 계층화된 주소 매칭 후보 목록.
- **실무 시나리오**: GPS 좌표를 도로명 주소로 실시간 변환, 배달/택시 앱 픽업 위치 자동 인식.

### 🔍 3. Places API (New) - 검색 (`searchText` & `searchNearby`)
- **입력 파라미터 (Inputs)**:
  - `textQuery`: 검색어 (예: `"명동교자 본점"`, `"강남역 맛집"`)
  - `includedTypes`: 카테고리 필터 (예: `["cafe", "korean_restaurant", "bakery"]`)
  - `locationRestriction`: 원형 반경 (강남 중심 `(37.50005, 127.0365)` + 반경 `1500.0` 미터)
  - `minRating`: 최소 평점 (`4.0`)
  - `X-Goog-FieldMask`: 선별 필드 (`places.id,places.displayName,places.formattedAddress,places.rating,places.userRatingCount,places.location,places.primaryType,places.businessStatus,places.regularOpeningHours`)
- **출력 속성 (Outputs)**:
  - 매장명, 평점, 리뷰 수, 영업 상태, 카테고리, 좌표, Place ID 목록.
- **실무 시나리오**: 조건부 매장 큐레이션("영업중 + 평점 4.0 이상"), 반경 1.5km 내 긴급 편의시설 거리순 정렬.

### 🏢 4. Places API (New) - 상세 속성 전체 조회 (`places/{placeId}`)
- **입력 파라미터 (Inputs)**:
  - `placeId`: 대상 장소 Place ID
  - `X-Goog-FieldMask`: `*` (50개 이상의 전체 필드 요청 와일드카드)
  - `languageCode`: `"ko"`
- **출력 속성 (Outputs)**:
  - 기본/위치: `id`, `displayName`, `formattedAddress`, `location`, `googleMapsUri`, `types`
  - 연락처/웹: `internationalPhoneNumber`, `nationalPhoneNumber`, `websiteUri`
  - 영업시간: `regularOpeningHours`, `currentOpeningHours`
  - 다이닝/서비스: `dineIn`, `delivery`, `takeout`, `curbsidePickup`, `reservable`, `servesVegetarianFood`, `servesBeer`, `servesWine`
  - 분위기/시설: `outdoorSeating`, `liveMusic`, `goodForChildren`, `goodForGroups`, `allowsDogs`
  - 접근성(Accessibility): `wheelchairAccessibleParking`, `wheelchairAccessibleEntrance`, `wheelchairAccessibleRestroom`
  - 주차/전기차: `parkingOptions`, `evChargeOptions`
  - 결제수단: `paymentOptions`
- **실무 시나리오**: 포털 매장 상세 프로필 카드 렌더링, 무장애(휠체어 배리어프리) 관광 지도, EV 충전소 매장 검색.

### 💬 5. Places API (New) - 음식점 고객 리뷰 & AI 소개 요약
- **입력 파라미터 (Inputs)**:
  - `placeId`: 음식점 Place ID
  - `X-Goog-FieldMask`: `displayName,rating,userRatingCount,reviews,editorialSummary,googleMapsUri`
  - `languageCode`: `"ko"` (외국어 리뷰 한국어 자동 번역)
- **출력 속성 (Outputs)**:
  - `editorialSummary`: 식당 분위기 및 시그니처 메뉴에 대한 1~2문장 AI 요약.
  - `reviews` 배열 (최대 5건):
    - `authorAttribution`: 리뷰어 이름, 지도 프로필 URL, 프로필 사진 URL
    - `rating`: 별점 점수 (1 ~ 5)
    - `text`: 한국어로 자동 번역된 리뷰 본문 및 언어코드
    - `originalText`: 원본 작성 리뷰 본문 및 원본 언어코드
    - `relativePublishTimeDescription`: 상대적 작성 시점 (예: `"2주 전"`, `"1달 전"`)
    - `publishTime`: ISO 8601 UTC 타임스탬프
    - `googleMapsUri`: Google 지도 내 해당 리뷰 바로가기 URL
- **실무 시나리오**: 음식점 평판 및 고객 감성 분석, 외국인 관광객 다국어 리뷰 자동 번역, 지도 리뷰 원문 링크 연동.

### ✍️ 6. Places Autocomplete API (`places:autocomplete`)
- **입력 파라미터 (Inputs)**:
  - `input`: 타이핑 검색어 (예: `"남산"`, `"코엑스"`)
  - `sessionToken`: 과금 최적화용 UUID v4 토큰
  - `origin`: 기준점 좌표 (서울시청 `(37.5665, 126.9780)`)
  - `includedRegionCodes`: `["kr"]`
- **출력 속성 (Outputs)**:
  - `placePrediction`: `mainText`, `secondaryText`, `distanceMeters`(서울시청 기준 직선거리 m), `placeId`, `types`.
- **실무 시나리오**: 검색창 실시간 인스턴트 자동완성 및 실시간 거리 표시, Session Token 기반 API 과금 절감.

### 🚗 7. Directions API (`gmaps.directions`)
- **입력 파라미터 (Inputs)**:
  - `origin`: `"서울특별시 중구 한강대로 405 서울역"`
  - `destination`: `"서울특별시 강남구 영동대로 513 코엑스"`
  - `waypoints`: `["명동대성당", "남산서울타워", "롯데월드몰"]`
  - `optimize_waypoints`: `True` (다중 경유지 방문 순서 최적화 TSP 엔진)
  - `mode`: `"transit"` (국내 대중교통) / `"driving"` (한국 운전 모드 제약 시 대체 안내)
  - `departure_time`: `"now"`
  - `traffic_model`: `"best_guess"`
- **출력 속성 (Outputs)**:
  - `waypoint_order`, `legs` (`distance`, `duration`, `duration_in_traffic`), `steps` (턴바이턴 주행 가이드 및 동작), `overview_polyline`.
- **실무 시나리오**: 당일 퀵/배송 다중 경유지 최단 순서 정렬(TSP), 출퇴근 실시간 교통 반영 ETA 산출.

### 📏 8. Distance Matrix API (`gmaps.distance_matrix`)
- **입력 파라미터 (Inputs)**:
  - `origins`: `["서울역", "용산역", "청량리역"]`
  - `destinations`: `["강남역", "여의도 더현대 서울"]`
  - `mode`: `"driving"` / `"transit"`
  - `departure_time`: `"now"`
- **출력 속성 (Outputs)**:
  - $N \times M$ 거리 행렬: `distance`, `duration`, `duration_in_traffic`, `status`.
- **실무 시나리오**: 라이드헤일링/배달 기사 최단 도착 시간 자동 배차, 물류 허브 거점 최적 입지 분석.

### ⛰️ 9. Elevation API (`gmaps.elevation` & `elevation_along_path`)
- **입력 파라미터 (Inputs)**:
  - `locations`: 북한산 백운대(836m), 남산타워(243m), 롯데월드타워, 여의도 한강공원
  - `path` + `samples`: 서울역 ➡️ 남산타워 등산로 경로 샘플링 ($N=5$)
- **출력 속성 (Outputs)**:
  - `elevation`(해발 고도 m), `resolution`(해상도), 경로 고도 프로파일.
- **실무 시나리오**: 등산/사이클 코스 경사도 및 획득고도 분석, 드론/UAM 도심 항공 비행 안전고도 검증.

### ⏰ 10. Time Zone API (`gmaps.timezone`)
- **입력 파라미터 (Inputs)**:
  - `location`: 서울 `(37.5665, 126.9780)`, 제주도, 독도, 뉴욕
  - `timestamp`: UTC 타임스탬프
- **출력 속성 (Outputs)**:
  - `timeZoneId` (`Asia/Seoul`), `timeZoneName` (`한국 표준시`), `rawOffset` (+9시간), `dstOffset` (서머타임 0시간).
- **실무 시나리오**: 글로벌 항공/호텔 예약 시스템 현지 시간 계산, IoT 기기 GPS 기반 시계 동기화.

### 📶 11. Geolocation API (`gmaps.geolocate`)
- **입력 파라미터 (Inputs)**:
  - `consider_ip=True`, 주변 Wi-Fi AP 및 기지국 신호.
- **출력 속성 (Outputs)**:
  - `location` (`lat`, `lng`), `accuracy` (정확도 반경 m).
- **실무 시나리오**: 지하철 및 코엑스몰 등 GPS 음영지역 실내 측위, 저전력 IoT 화물 위치 추적.

### 🛣️ 12. Roads API (`gmaps.snap_to_roads`)
- **입력 파라미터 (Inputs)**:
  - `path`: 테헤란로 주행 중 수집된 GPS 궤적
  - `interpolate=True` (도로 형상 자동 보간)
- **출력 속성 (Outputs)**:
  - `snappedPoints`: 테헤란로 도로 중심선 보정 좌표, `originalIndex`, `placeId`.
- **실무 시나리오**: 빌딩 숲 GPS 반사 궤적 도로 보정, 법정 제한속도 준수율 관제.

---

## 🏗️ 4. 노트북 구조 및 UX 표준 (Notebook Layout)

노트북을 생성할 때는 아래 표준 순서를 엄격히 준수합니다:

1. **표지 및 실행 요약 매트릭스 (Markdown)**: 12개 API 전체 서비스, 메소드, 입출력 속성, 서울 실무 시나리오 종합 표.
2. **환경 설정 및 패키지 설치 가이드 (Markdown)**: `uv sync` 및 `pip`/`venv` 명령어, `.env` 가이드.
3. **패키지 임포트 및 헬퍼 함수 (Code)**: 라이브러리 임포트, `.env` 자동 탐색(`find_dotenv()`), `print_json()` 함수.
4. **API 키 로드 및 클라이언트 초기화 (Code)**: 안전한 키 로드 및 `googlemaps.Client` 생성.
5. **API별 번호 매겨진 테스트 섹션 (Markdown + Code 쌍)**:
   - Markdown: 지원 입력 파라미터, 반환 출력 속성, 실무 시나리오 명시.
   - Code: 직관적인 호출 및 `pandas.DataFrame` 테이블 렌더링.
   - 예외 처리: `try...except`로 감싸고 미활성화 시 Cloud Console 활성화 링크 안내.
6. **종합 속성 매트릭스 & 공식 개발자 레퍼런스 (Markdown)**: 요약 표 및 공식 문서 링크.

---

## 🤖 5. 파이썬 코드 생성 레시피 (Automation Recipe)

노트북(`.ipynb`) 파일은 문자열 치환이나 정규식이 아닌, 파이썬의 `json.dump()`와 구조화된 딕셔너리를 사용하여 `nbformat 4.5` 표준에 맞게 생성해야 합니다:

```python
import json
import uuid

def make_cell(cell_type: str, source: str) -> dict:
    lines = [l if l.endswith("\n") else l + "\n" for l in source.split("\n")]
    if lines and lines[-1] == "\n":
        lines.pop()
    cell = {
        "cell_type": cell_type,
        "id": uuid.uuid4().hex[:8],
        "metadata": {},
        "source": lines
    }
    if cell_type == "code":
        cell["execution_count"] = None
        cell["outputs"] = []
    return cell

# 셀 조립 및 json 저장
notebook_dict = {
    "cells": cells,
    "metadata": {
        "kernelspec": {"display_name": "Python 3 (.venv)", "language": "python", "name": "python3"},
        "language_info": {"name": "python", "version": "3.12.0"}
    },
    "nbformat": 4,
    "nbformat_minor": 5
}
with open("google_map/google_maps_api_test.ipynb", "w", encoding="utf-8") as f:
    json.dump(notebook_dict, f, indent=2, ensure_ascii=False)
```

---

## 🔒 6. 보안 및 시크릿 검증 체크리스트
- [ ] 마크다운이나 코드 셀에 실제 API 키나 토큰이 포함되어 있지 않은가?
- [ ] `.env` 파일이 `.gitignore`에 등록되어 Git 추적에서 제외되었는가?
- [ ] 커밋 및 푸시 전 정규식 기반 시크릿 스캐너가 통과되었는가?
- [ ] Git 작성자 정보가 `ForusOne` (`forusone777@gmail.com`)과 일치하는가?
