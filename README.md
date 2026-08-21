# 🗺️ Google Maps Platform APIs 테스트 스위트 (Test Suite)

Google Maps Platform의 핵심 Web Service API 및 최신 **Places API (New v1)**의 모든 기능, 입력 옵션, 반환 속성(음식점 리뷰 및 AI 요약 포함)을 서울 주요 랜드마크를 중심으로 검증할 수 있는 종합 테스트 스위트 및 주피터 노트북입니다.

---

## 📌 주요 기능 및 지원 API 목록

- **Geocoding API**: 주소 ➡️ 좌표 지오코딩, 한국 우편번호(06236) 컴포넌트 필터링, 지도 뷰포트 피팅.
- **Reverse Geocoding API**: GPS 좌표 ➡️ 도로명 주소 역지오코딩, `ROOFTOP` 정밀 필터링.
- **Places API (New)**:
  - 텍스트 검색 (`places:searchText`): 평점(4.0+) 및 영업 여부 조건부 검색.
  - 주변 검색 (`places:searchNearby`): 서울 강남 테헤란로 반경 1.5km 매장 탐색.
  - 장소 상세 정보 (`places/{placeId}`): 와일드카드 `*` 필드마스크를 통한 50+ 상세 속성(접근성, EV충전, 주차, 결제수단 등) 조회.
  - 음식점 리뷰 & AI 요약: 고객 리뷰 5건(작성자, 평점, 한국어 자동 번역, 원문, 지도URL) 및 AI 에디토리얼 소개 요약(`editorialSummary`).
- **Places Autocomplete API**: 서울시청 기준 "남산", "코엑스" 실시간 자동완성 및 직선거리(km) 산출, `sessionToken` 과금 최적화.
- **Directions API**: 서울역 ➡️ 명동, 남산, 잠실, 코엑스 다중 경유지 최적 순서 배송(TSP, `optimize_waypoints=True`), 실시간 교통 반영 경로 탐색.
- **Distance Matrix API**: 서울역/용산역/청량리역 ➡️ 강남역/더현대 서울 간 N x M 최적 배차 거리 및 소요시간 행렬.
- **Elevation API**: 북한산, 남산타워, 롯데월드타워, 한강공원 해발 고도 비교 및 서울역 ➡️ 남산타워 등산로 경로 고도 프로파일.
- **Time Zone API**: 대한민국 서울(KST, UTC+9), 제주도, 독도 표준시 확인 및 글로벌 타임존/서머타임(DST) 비교.
- **Geolocation API**: 서울 시내 지하철/실내 등 GPS 음영지역 기지국 및 Wi-Fi AP 기반 위치 추정.
- **Roads API**: 강남 테헤란로 빌딩 숲 주행 차량 GPS 궤적 도로 스냅 보정(Snap to Roads).

---

## 🚀 빠른 시작 가이드 (Getting Started)

### 1. 환경 설정 및 패키지 설치

본 프로젝트는 초고속 패키지 관리자 [`uv`](https://github.com/astral-sh/uv) 및 표준 `pip` 환경을 모두 지원합니다.

#### ⚡ Option A. `uv` 사용 (권장)
```bash
# 저장소 클론 및 이동
git clone https://github.com/shins777/google_apis.git
cd google_apis

# 의존성 설치 및 환경 동기화
uv sync

# Jupyter 커널 등록
uv run python -m ipykernel install --user --name google_apis_env --display-name "Python (google_apis_env)"
```

#### 🐍 Option B. 표준 `pip` / `venv` 사용
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
python -m ipykernel install --user --name google_apis_env --display-name "Python (google_apis_env)"
```

---

### 2. API 키 설정 (`.env`)

프로젝트 루트 디렉토리에 `.env` 파일을 생성하고 발급받은 Google Maps API 키를 입력합니다:

```env
GOOGLE_MAPS_API_KEY=AIzaSy...your_actual_api_key_here
```

> [!NOTE]
> 대상 API들이 [Google Cloud Console API 라이브러리](https://console.cloud.google.com/apis/dashboard)에서 활성화되어 있어야 정상적으로 호출됩니다.

---

### 3. 노트북 실행

VS Code 또는 JupyterLab에서 아래 노트북을 열고 실행합니다:
1. **[`google_map/map_api_check.ipynb`](google_map/map_api_check.ipynb)**: 12개 Google Maps Platform API의 입출력 및 서울 랜드마크 전수 검증
2. **[`google_map/recommendation_w_map.ipynb`](google_map/recommendation_w_map.ipynb)**: Gemini 3.5 Flash + Maps API 결합 맛집/리뷰/유사식당/식후카페/도보·차량 여행 코스 추천

---

### 📁 에이전트 스킬 (Agent Skills)

본 프로젝트에는 노트북 생성 및 유지보수 명세를 담은 전용 스킬이 정의되어 있습니다:
- [`.agents/skills/map_api_test/SKILL.md`](.agents/skills/map_api_test/SKILL.md): 12개 API의 입력/출력 속성, 서울 비즈니스 시나리오 가이드.
- [`.agents/skills/map_scenario/SKILL.md`](.agents/skills/map_scenario/SKILL.md): Google Maps + Gemini 3.5 Flash 결합 5단계 음식점 및 맞춤 여행 코스 추천 가이드.
