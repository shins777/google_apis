---
name: map_api_test
description: Google Maps Platform API 테스트 노트북(map_api_check.ipynb) 생성 및 유지보수를 위한 간결한 요구사항 가이드.
---

# 🗺️ Google Maps API 테스트 노트북 생성 가이드 (`map_api_test`)

본 가이드는 `google_map/map_api_check.ipynb` 노트북을 일관성 있고 깔끔하게 생성/유지보수하기 위한 핵심 요구사항과 표준 레시피를 정의합니다.

---

## 🎯 1. 핵심 요구사항 (Core Requirements)

1. **서울 중심 데이터셋 사용**: 서울 주요 랜드마크(강남파이낸스센터 GFC, 명동교자, N서울타워, 코엑스, 서울시청 등)를 테스트 데이터로 활용.
2. **Places API (New) 완전 지원**:
   - 텍스트/주변 검색 (`searchText`, `searchNearby`)
   - 전체 상세 조회 (`places/{placeId}` with `*` 와일드카드)
   - 고객 리뷰 분석 (작성자, 평점, 한국어 자동 번역본, 지도 URL) 및 AI 요약 (`editorialSummary`)
   - 자동완성 (`places:autocomplete`) 및 세션 토큰 (`sessionToken`)
3. **전체 입출력 속성 검증**: 주요 파라미터를 적용하고 결과를 `pandas.DataFrame` 표 형식으로 직관적으로 시각화.
4. **보안 및 예외 처리**:
   - API 키는 `.env`에서 로드하며 절대 코드에 노출하지 않음.
   - 미활성화 API 호출 시 콘솔 활성화 링크 안내.

---

## 📊 2. API별 입출력 및 시나리오 요약

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

## 🏗️ 3. 노트북 구성 표준 (Notebook Layout)

1. **개요 및 실행 표 (Markdown)**: 12개 API 요약 테이블.
2. **환경 설정 가이드 (Markdown)**: `uv sync` / `pip` 설치법 & `.env` 설정 안내.
3. **라이브러리 및 헬퍼 함수 (Code)**: `googlemaps`, `requests`, `pandas`, `print_json()` 정의.
4. **API별 테스트 블록 (Markdown + Code 쌍)**:
   - 마크다운: 입력/출력 및 시나리오 설명.
   - 코드: API 호출 및 `pandas.DataFrame` 표 시각화, `try...except` 예외 처리.
5. **종합 요약 및 레퍼런스 (Markdown)**: 요약 표 및 공식 문서 링크.

---
