---
name: map_scenario
description: Google Maps Platform API와 Gemini Flash 모델을 결합하여 음식점 정보, 리뷰 기반 인기 메뉴 분석, 유사 맛집, 식후 카페, 도보/차량 여행 코스를 자동 생성하는 실무 시나리오 노트북 구축 가이드.
---

# 🍽️ Google Maps + Gemini AI 음식점 & 여행 코스 시나리오 가이드 (`map_scenario`)

본 가이드는 **Google Maps Platform API**와 **Gemini Flash 모델**을 연계하여 특정 음식점에 대한 5단계 심층 분석 및 맞춤형 여행 코스를 생성하는 주피터 노트북(`google_map/restaurant_scenario_recommendation.ipynb`)의 요구사항과 구현 방안을 정의합니다.

---

## 🔑 1. 환경 및 API 키 구성

`.env` 파일에 아래 두 개의 환경변수를 설정하고 `python-dotenv`로 안전하게 로드합니다:

```env
# Google Maps Platform API 키
GOOGLE_MAPS_API_KEY=AIzaSy...your_maps_api_key_here

# Google Gemini API 키
GEMINI_API_KEY=AIzaSy...your_gemini_api_key_here
```

### 필수 라이브러리:
- `googlemaps`: Google Maps Python SDK
- `requests`: 최신 Places API (New v1) REST 호출
- `google-genai` 또는 `google-generativeai`: Gemini Flash 모델 호출
- `pandas`, `python-dotenv`, `ipykernel`

---

## 🎯 2. 5대 핵심 요구사항 및 단계별 구현 방안

```
[사용자 입력: 음식점명/주소]
         │
         ▼ (Step 1: Places API New Details with '*')
1. 음식점 기본 정보 (주소, 영업시간/브레이크타임, 연락처, 편의시설)
         │
         ▼ (Step 2: Reviews + Gemini Flash LLM)
2. 리뷰 기반 인기 메뉴 & 시그니처 요리 추출
         │
         ▼ (Step 3: Nearby Search + Gemini Flash)
3. 비슷한 분위기 & 카테고리의 유사 맛집 추천
         │
         ▼ (Step 4: Nearby Search + Walk Radius)
4. 식사 후 도보 방문 가능한 추천 카페 탐색
         │
         ▼ (Step 5: Tourist Attractions + Directions API + Gemini Flash)
5. 맞춤형 여행 코스 (차량 드라이브 코스 & 도보 산책 코스)
```

---

### Step 1. 음식점 기본 정보 조회 (Restaurant Basic Info)
- **사용 API**: Places API (New) Text Search (`searchText`) ➡️ Place Details (`places/{placeId}`) with `X-Goog-FieldMask: *`
- **추출 속성**:
  1. **주소**: `formattedAddress`, `addressComponents` (도로명 주소, 우편번호)
  2. **영업시간 & 브레이크타임**: `regularOpeningHours`, `currentOpeningHours.periods` (요일별 오픈/마감 시간, 브레이크타임 구간)
  3. **전화번호**: `nationalPhoneNumber`, `internationalPhoneNumber`
  4. **편의 및 부대시설**:
     - 유아의자/키즈: `goodForChildren`, `menuForChildren`
     - 화장실: `restroom`, `wheelchairAccessibleRestroom`
     - 단체이용 & 좌석: `goodForGroups`, `outdoorSeating`
     - 주차: `parkingOptions` (무료/유료 주차, 발렛 등)
     - 예약 및 서비스: `reservable`, `takeout` (포장), `delivery` (배달), `paymentOptions`

---

### Step 2. 리뷰 기반 인기 메뉴 분석 (Popular Menus from Reviews)
- **사용 API**: Places API (New) `reviews` (최상위 리뷰 원문/번역본) + `editorialSummary`
- **Gemini Flash 프롬프트 로직**:
  - 수집된 리뷰 텍스트 및 에디토리얼 요약을 Gemini Flash 모델에 전달.
  - 고객들이 반복적으로 칭찬하는 **대표 시그니처 메뉴**, **인기 조합**, **가격대/맛의 특징(맵기, 양, 식감 등)**을 구조화된 JSON 또는 Markdown 테이블로 추출.

---

### Step 3. 비슷한 맛집 추천 (Similar Restaurants)
- **사용 API**: Places API (New) Nearby Search (`searchNearby`) 또는 Text Search
- **검색 조건**:
  - 기준 음식점의 `primaryType`(예: `korean_restaurant`, `seafood_restaurant`) 및 가격대(`priceLevel`)
  - 반경: 1.5km ~ 3km 이내, `minRating: 4.0`
- **Gemini Flash 역할**:
  - 검색된 3~5개 후보 음식점 중 본점과 맛/분위기가 가장 유사한 곳을 선정하고 비교 이유(예: "동일한 진한 육수 기반 칼국수", "자가제면 방식") 요약 제시.

---

### Step 4. 식사 후 추천 근처 카페 (Post-Meal Nearby Cafes)
- **사용 API**: Places API (New) `searchNearby`
- **검색 조건**:
  - 음식점 좌표 기준 반경 500m ~ 1km 이내 (도보 5~10분 거리)
  - `includedTypes: ["cafe", "coffee_shop", "bakery"]`, `minRating: 4.0`, `openNow: true`
- **출력 구성**:
  - 카페명, 평점, 도보 거리(m), 대표 디저트/커피 강점, 주차 및 야외 좌석 여부.

---

### Step 5. 음식점 근처 맞춤 여행 코스 (Curated Itinerary Planning)
- **사용 API**: Places API (New) 관광지 검색 + Directions API (`gmaps.directions`) + Gemini Flash

#### 🚗 A. 차량 드라이브 코스 (Driving Course)
- 음식점 반경 5~15km 내 주요 관광지/뷰포인트(예: 남산, 북악스카이웨이, 한강공원 등) 2~3곳 선정.
- Directions API (`mode="driving"` 또는 대중교통 경로)로 구간별 이동 시간, 주차 가능 여부(`parkingOptions`) 연계.
- Gemini Flash가 총 소요시간 및 드라이브 추천 시간대(야경/일몰 등) 가이드 제공.

#### 🚶 B. 도보 산책 코스 (Walking Course)
- 음식점 기준 도보 5~20분 내 공원, 역사문화 유적, 쇼핑거리, 산책로(예: 명동성당, 청계천 등) 선정.
- Directions API (`mode="walking"`)로 보행 경로 및 소요시간 계산.
- Gemini Flash가 식후 가볍게 걸을 수 있는 힐링/문화 산책 팁 안내.

---

## 📁 3. 산출물 노트북 표준 구조 (`restaurant_scenario_recommendation.ipynb`)

1. **[Markdown] 프로젝트 소개 & 5단계 아키텍처 개요**
2. **[Code] 라이브러리 임포트 & .env 키 로드 (Maps & Gemini)**
3. **[Code] Step 1: 음식점 검색 & 기본 정보/편의시설 카드 출력**
4. **[Code] Step 2: Google 리뷰 수집 ➡️ Gemini Flash 인기 메뉴 분석**
5. **[Code] Step 3: 반경 3km 내 유사 맛집 검색 & 비교 추천**
6. **[Code] Step 4: 도보 10분 내 식후 추천 카페 리스트업 (거리순)**
7. **[Code] Step 5-A: 차량 드라이브 코스 생성 (경로 & 소요시간)**
8. **[Code] Step 5-B: 도보 산책 코스 생성 (턴바이턴 보행 가이드)**
9. **[Markdown] 5단계 종합 결과 보고서 요약**
