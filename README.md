# 🗺️ Google Maps Platform & Gemini AI 엔터프라이즈 통합 스위트

Google Maps Platform의 핵심 Web Service API(Places New 포함 12종) 및 **Google Gemini 3.5 Flash LLM**을 결합하여 서울 주요 랜드마크 데이터 기반의 기능 검증과 지능형 맛집·카페·여행 코스 추천 시스템을 제공하는 파이썬 개발 툴킷입니다.

---

## 📂 프로젝트 구성 및 핵심 노트북

```
google_apis/
├── .agents/
│   └── skills/
│       ├── map_api_test/
│       │   └── SKILL.md                 # 12개 Google Maps Platform API 테스트 스킬 정의서
│       └── map_scenario/
│           └── SKILL.md                 # Maps API + Gemini 3.5 Flash 5단계 추천 스킬 정의서
├── google_map/
│   ├── map_api_check.ipynb              # 🗺️ 12개 Maps API 입출력 전수 검증 마스터 노트북
│   └── recommendation_w_map.ipynb      # 🍽️ Gemini 3.5 Flash 기반 지능형 맛집 & 여행 코스 플래너
├── pyproject.toml                       # Python 3.13+, googlemaps, google-genai 등 패키지 설정
├── uv.lock                              # uv 패키지 종속성 고정 락파일
├── README.md                            # 프로젝트 종합 설명서 (본 파일)
└── .env                                 # API 키 보안 관리 파일 (Git 제외)
```

---

## 📓 1. 제공되는 주피터 노트북 상세

### 1️⃣ [`google_map/map_api_check.ipynb`](google_map/map_api_check.ipynb)
**"12개 Google Maps Platform Web Service API 전수 기능 & 입출력 검증"**
- 서울 중심 실무 시나리오(강남파이낸스센터 GFC, 명동교자, N서울타워, 코엑스, 청계천, 한강공원 등) 적용.
- 최신 **Places API (New v1)**의 와일드카드 `*` 필드마스크를 통한 50+ 상세 속성, 고객 리뷰 5건(원문/한국어 번역본) 및 AI 에디토리얼 요약 검증.
- 다중 경유지 최적 경로 탐색(TSP), $N \times M$ 배차 거리 행렬, 해발 고도 프로파일, Wi-Fi 기반 Geolocation, 도로 스냅(Snap to Roads) 등 전체 API 입출력 검증 및 `pandas.DataFrame` 표 시각화.

### 2️⃣ [`google_map/recommendation_w_map.ipynb`](google_map/recommendation_w_map.ipynb)
**"Google Maps Platform + Gemini 3.5 Flash AI 지능형 음식점 분석 & 맞춤 여행 코스 추천"**
- 사용자 지정 음식점(예: '명동교자 본점')을 기준으로 5단계 AI 파이프라인을 자동 수행합니다.

```
┌────────────────────────────────────────────────────────────────────────┐
│ [사용자 입력: TARGET_RESTAURANT (예: '명동교자 본점')]                  │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 1️⃣ 음식점 기본 정보 & 편의시설 (Places API New Details `*`)             │
│    • 도로명 주소 & 우편번호, 전화번호, Google Maps 바로가기 URL         │
│    • 요일별 영업시간 & 브레이크타임                                    │
│    • 편의시설 표: 유아의자, 화장실, 단체석, 주차, 예약, 포장, 배달    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 2️⃣ 리뷰 기반 인기 메뉴 분석 (Places Reviews ➡️ Gemini 3.5 Flash)        │
│    • 실제 수집된 방문자 리뷰 + 에디토리얼 요약 전달                    │
│    • Gemini 3.5 Flash가 대표 인기 메뉴 Top 3, 맛/식감 포인트,         │
│      첫 방문자 꿀조합 및 팁 추출                                      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 3️⃣ 비슷한 맛집 추천 (Places SearchNearby ➡️ Gemini 3.5 Flash)          │
│    • 동일 카테고리/반경 2km 이내 맛집 후보 검색                        │
│    • Gemini 3.5 Flash가 맛의 결(육수/면발 등)과 방문 장점 비교 추천     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 4️⃣ 식사 후 추천 근처 카페 (Places SearchNearby ➡️ Gemini 3.5 Flash)   │
│    • 도보 5~10분 (반경 700m) 내 평점 4.0+ 카페/베이커리 거리순 탐색   │
│    • 하버사인 공식 기반 실제 보행 시간(분) 계산                        │
│    • Gemini 3.5 Flash가 식후 입가심 디저트/커피 페어링 큐레이션 제공   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 5️⃣ 음식점 근처 맞춤 여행 코스 (Directions API + Places + Gemini AI)    │
│    🚗 차량 드라이브 코스: 반경 15km 내 뷰포인트/야경 드라이브 코스     │
│    🚶 도보 산책 코스: 식후 20~40분 소화 겸 도심 문화/산책로 가이드     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 2. 지원 API 및 서울 테스트 시나리오 요약

| # | API 서비스명 | 주요 입력 (Inputs) | 주요 출력 (Outputs) | 서울 테스트 시나리오 |
| :---: | :--- | :--- | :--- | :--- |
| **01** | **Geocoding** | `address`, `components: {"country": "KR"}` | `formatted_address`, `location`, `viewport`, `address_components` | 강남파이낸스센터(GFC) 도로명 주소 정제 및 우편번호(06236) 검증 |
| **02** | **Reverse Geocoding** | `latlng`, `result_type`, `location_type: ["ROOFTOP"]` | 계층형 주소 목록 (`street_address` ~ `country`) | 역삼동 GPS 좌표 ➡️ 테헤란로 도로명 주소 변환 (배달/택시 픽업지) |
| **03** | **Places (New) Search** | `textQuery`, `locationRestriction`, `minRating: 4.0` | `places.id`, `displayName`, `rating`, `userRatingCount`, `location` | 명동/강남역 "영업중 + 평점 4.0+" 맛집 및 테헤란로 반경 1.5km 카페 검색 |
| **04** | **Places (New) Details** | `placeId`, `X-Goog-FieldMask: *` | 50+ 속성 (영업시간, 휠체어 접근성, EV충전, 주차, 결제수단 등) | 코엑스/더현대 매장 전체 프로필 및 무장애/편의시설 조회 |
| **05** | **Places (New) Reviews** | `placeId`, `X-Goog-FieldMask: reviews,editorialSummary` | 리뷰어, 평점(1~5), 한국어 번역본, 원문, 상대시간, `editorialSummary` | 명동교자 고객 리뷰 상세 분석 및 AI 매장 소개 요약 |
| **06** | **Places Autocomplete** | `input: "남산"`, `sessionToken`, `origin: 서울시청` | `placePrediction`, `mainText`, `distanceMeters`, `placeId` | 서울시청 기준 "남산" 실시간 자동완성 및 거리(km) 표기 |
| **07** | **Directions** | `origin`, `dest`, `waypoints`, `optimize_waypoints=True` | `waypoint_order`, `legs(distance, duration)`, `steps(turn-by-turn)` | 서울역 ➡️ 명동, 남산, 잠실 다중 경유지 최적 순서 배송 (TSP) |
| **08** | **Distance Matrix** | `origins[]` (3개역), `destinations[]` (2개 거점) | $N \times M$ 행렬 (`distance`, `duration`, `duration_in_traffic`) | 서울역/용산역/청량리역 ➡️ 강남역/더현대 최적 배차 매트릭스 |
| **09** | **Elevation** | `locations[]` (4개 랜드마크), `path[]` (등산로) | `elevation`(해발 고도 m), `resolution`, 경로 고도 프로파일 | 북한산/남산/롯데월드타워 고도 비교 & 서울역 ➡️ 남산타워 고도 변화 |
| **10** | **Time Zone** | `location: 서울`, `timestamp` | `timeZoneId` (`Asia/Seoul`), `rawOffset` (+9h), `dstOffset` (0h) | 대한민국 서울 표준시(KST) 및 해외 도시 시차/서머타임 비교 |
| **11** | **Geolocation** | `consider_ip=True` | `location(lat, lng)`, `accuracy`(오차 반경 m) | 지하철/실내 GPS 음영지역 기지국 및 Wi-Fi 기반 위치 추정 |
| **12** | **Roads** | `path[]` (GPS 궤적), `interpolate=True` | `snappedPoints(location, placeId)` | 강남 테헤란로 빌딩 숲 차량 GPS 궤적 도로 스냅 보정 |

---

## 🚀 3. 빠른 시작 가이드 (Getting Started)

### 1단계. 환경 설정 및 패키지 설치

본 프로젝트는 초고속 패키지 관리자 [`uv`](https://github.com/astral-sh/uv) 및 표준 `pip` 가상환경을 모두 지원합니다.

#### ⚡ Option A. `uv` 사용 (권장)
```bash
# 저장소 클론 및 이동
git clone https://github.com/shins777/google_apis.git
cd google_apis

# 의존성 설치 및 가상환경(.venv) 동기화
uv sync

# Jupyter 커널 등록
uv run python -m ipykernel install --user --name google_apis_env --display-name "Python (google_apis_env)"
```

#### 🐍 Option B. 표준 `pip` / `venv` 사용
```bash
# 가상환경 생성 및 활성화
python3 -m venv .venv
source .venv/bin/activate

# 패키지 설치
pip install -e .

# Jupyter 커널 등록
python -m ipykernel install --user --name google_apis_env --display-name "Python (google_apis_env)"
```

---

### 2단계. API 키 설정 (`.env`)

프로젝트 루트 디렉토리에 `.env` 파일을 생성하고 발급받은 Google Maps Platform API 키와 Google Gemini API 키를 입력합니다:

```env
# Google Maps Platform Web Service API Key
GOOGLE_MAPS_API_KEY=AIzaSy...your_actual_maps_api_key_here

# Google Gemini API Key (Gemini 3.5 Flash 호출용)
GEMINI_API_KEY=AIzaSy...your_actual_gemini_api_key_here
```

> [!IMPORTANT]
> 1. Google Maps Platform API들이 [Google Cloud Console](https://console.cloud.google.com/apis/dashboard)에서 활성화되어 있어야 정상 호출됩니다.
> 2. `.env` 파일은 `.gitignore`에 등록되어 있어 원격 저장소에 절대 커밋되지 않습니다.

---

### 3단계. 주피터 커널 선택 및 노트북 실행

VS Code 또는 JupyterLab에서 노트북을 실행할 때:
1. 노트북 우측 상단의 **[커널 선택 (Select Kernel)]** 클릭
2. **[Python Environments...]** ➡️ **`.venv (Python 3.13)`** 또는 **`google_apis_env`** 선택
3. 원하는 노트북을 열고 실행:
   - **[`google_map/map_api_check.ipynb`](google_map/map_api_check.ipynb)**
   - **[`google_map/recommendation_w_map.ipynb`](google_map/recommendation_w_map.ipynb)**

---

## 🤖 4. 에이전트 스킬 명세 (Agent Skills)

본 저장소는 AI 에이전트가 코드를 자동으로 유지보수하고 확장할 수 있도록 표준 스킬을 제공합니다:

- **[`map_api_test`](.agents/skills/map_api_test/SKILL.md)**: 12개 Google Maps Platform API의 입출력 명세, 서울 데이터셋 시나리오, DataFrame 시각화 표준 레시피.
- **[`map_scenario`](.agents/skills/map_scenario/SKILL.md)**: Places API (New) + Gemini 3.5 Flash를 연계한 5단계 음식점 및 맞춤 도보/차량 여행 코스 설계 가이드.

---

## 🛡️ 5. 보안 및 기여 가이드

- **Secret Key 노출 방지**: 커밋 및 푸시 전 모든 diff에서 API 키(`AIzaSy...`), 토큰, 비공개 인증 정보가 포함되어 있지 않은지 자동 검증을 수행합니다.
- **기여 및 라이선스**: 본 프로젝트는 Google APIs를 활용하는 개발자를 위한 예제 및 도구 모음입니다.
