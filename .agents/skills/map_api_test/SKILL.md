---
name: map_api_test
description: Requirements, architectural specifications, and code generation recipes for building, testing, and maintaining the comprehensive Google Maps Platform APIs Jupyter Notebook (google_maps_api_test.ipynb).
---

# Google Maps Platform API Test Notebook Generation Skill (`map_api_test`)

This skill defines the end-to-end requirements, architectural standards, input/output attribute specifications, and generation guidelines for the **Google Maps Platform API Comprehensive Test Notebook** (`google_map/google_maps_api_test.ipynb`).

---

## 🎯 1. Purpose & Core Objectives

The `google_maps_api_test.ipynb` notebook serves as a production-grade interactive test suite and reference guide for Google Maps Platform APIs.

### Key Objectives:
1. **Exhaustive Input & Output Coverage**: Demonstrate every practical parameter (filter, bias, bounds, session token, field mask) and parse every response attribute (geometries, address components, reviews, opening hours, accessibility, traffic models).
2. **Seoul & Korea-Centric Localization**: Use realistic, prominent landmarks in Seoul and South Korea (e.g., Gangnam Finance Center, Myeongdong Kyoja, Namsan Seoul Tower, COEX, Seoul City Hall, Lotte World Tower, Bukhansan).
3. **Places API (New / Modern v1) Focus**: Thoroughly exercise modern REST v1 endpoints including Text Search, Nearby Search, Place Details with `*` wildcard FieldMask, Customer Reviews, and Autocomplete with Session Tokens.
4. **Resilience & Graceful Handling**: Account for regional constraints (e.g., Google Maps driving mode limitations in South Korea due to spatial data regulations by supporting `mode="transit"` and informative fallback messages) and handle non-activated APIs with direct Google Cloud Console activation links.
5. **Clean Data Presentation**: Present structured responses using `pandas.DataFrame` and pretty-printed JSON snippets (`print_json`) for immediate clarity.

---

## 🛠️ 2. Environment & Dependency Specifications

### Python Runtime & Packages
- **Python**: 3.10+ (tested on Python 3.12)
- **Required Libraries**:
  - `googlemaps>=4.10.0`: Official Google Maps Services Python Client
  - `requests>=2.31.0`: Direct REST v1 calls for modern Places API & Routes API
  - `pandas>=2.0.0`: Structured tabular presentation of components, matrix, and reviews
  - `python-dotenv>=1.0.0`: Environment variable management
  - `ipykernel>=6.29.0`: Jupyter kernel execution

### Secret Management Guardrails
- **Never Hardcode Keys**: API keys must **always** be loaded via `os.getenv("GOOGLE_MAPS_API_KEY")` with `python-dotenv`.
- **Interactive Fallback**: If the environment variable is missing, provide a safe prompt (`getpass.getpass()`) or display a clear instruction without breaking notebook initialization.
- **Git Safety**: `.env` and `.env.local` files must be tracked in `.gitignore`. Mask API keys (`AIzaSy...xxxx`) when printing logs.

---

## 📋 3. Detailed API Requirements & Scenarios

The notebook must include dedicated, well-documented sections for all 12 API capabilities below:

### 📍 1. Geocoding API (`gmaps.geocode`)
- **Inputs**:
  - `address`: Full street name address (e.g., `"서울특별시 강남구 테헤란로 152 강남파이낸스센터"`)
  - `components`: Mandatory restriction (e.g., `{"country": "KR", "postal_code": "06236"}`)
  - `region`: Country bias (`"kr"`)
  - `language`: `"ko"`
- **Outputs**:
  - `formatted_address`, `place_id`, `types`
  - `geometry`: `location` (`lat`, `lng`), `location_type` (`ROOFTOP`, `RANGE_INTERPOLATED`), `viewport`
  - `address_components`: Breakdown of postal code, administrative district, street, sublocality
- **Scenarios**: Korean address standardization, 5-digit postal code verification, automated map viewport fitting.

### 🔄 2. Reverse Geocoding API (`gmaps.reverse_geocode`)
- **Inputs**:
  - `latlng`: Coordinates tuple `(37.50005, 127.0365)`
  - `result_type`: Filter by `['street_address', 'premise']`
  - `location_type`: Filter by `['ROOFTOP']`
  - `language`: `"ko"`
- **Outputs**:
  - Hierarchical address candidates from building/street address down to administrative district (`sublocality_level_1`, `locality`, `country`).
- **Scenarios**: GPS coordinate-to-address conversion for ride-hailing/delivery pickup point auto-filling.

### 🔍 3. Places API (New) - Search (`searchText` & `searchNearby`)
- **Inputs**:
  - `textQuery`: Search keywords (e.g., `"명동교자 본점"`, `"강남역 맛집"`)
  - `includedTypes`: Filter categories (e.g., `["cafe", "korean_restaurant", "bakery"]`)
  - `locationRestriction`: Circle boundary (e.g., Gangnam center `(37.50005, 127.0365)` + radius `1500.0` meters)
  - `minRating`: Minimum rating threshold (`4.0`)
  - `X-Goog-FieldMask`: Selective fields (e.g., `places.id,places.displayName,places.formattedAddress,places.rating,places.userRatingCount,places.location,places.primaryType,places.businessStatus,places.regularOpeningHours`)
- **Outputs**:
  - List of matching places with ratings, review counts, business status, coordinates, and Place IDs.
- **Scenarios**: Filtered curated store search ("Open now + Rating ≥ 4.0") and radius-based nearby convenience amenity discovery.

### 🏢 4. Places API (New) - Full Details (`places/{placeId}`)
- **Inputs**:
  - `placeId`: Target Place ID
  - `X-Goog-FieldMask`: `*` (Wildcard requesting all 50+ available attributes)
  - `languageCode`: `"ko"`
- **Outputs**:
  - Identification & Geometry: `id`, `displayName`, `formattedAddress`, `location`, `googleMapsUri`, `types`
  - Contact & Web: `internationalPhoneNumber`, `nationalPhoneNumber`, `websiteUri`
  - Opening Hours: `regularOpeningHours`, `currentOpeningHours`
  - Dining & Services: `dineIn`, `delivery`, `takeout`, `curbsidePickup`, `reservable`, `servesVegetarianFood`, `servesBreakfast`, `servesLunch`, `servesDinner`, `servesBeer`, `servesWine`
  - Amenities & Atmosphere: `outdoorSeating`, `liveMusic`, `goodForChildren`, `goodForGroups`, `allowsDogs`
  - Accessibility: `wheelchairAccessibleParking`, `wheelchairAccessibleEntrance`, `wheelchairAccessibleRestroom`, `wheelchairAccessibleSeating`
  - Parking & EV: `parkingOptions`, `evChargeOptions`
  - Payments: `paymentOptions`
- **Scenarios**: Complete business profile card generation, barrier-free accessibility mapping, EV charging station discovery.

### 💬 5. Places API (New) - Restaurant Reviews & AI Summary
- **Inputs**:
  - `placeId`: Target restaurant/venue Place ID
  - `X-Goog-FieldMask`: `displayName,rating,userRatingCount,reviews,editorialSummary,googleMapsUri`
  - `languageCode`: `"ko"` (enables auto-translation of foreign reviews into Korean)
- **Outputs**:
  - `editorialSummary`: 1–2 sentence AI summary of atmosphere, history, and signature dishes.
  - `reviews` Array (up to 5 top reviews):
    - `authorAttribution`: `displayName`, `uri` (Maps profile link), `photoUri` (avatar)
    - `rating`: Star score (1 to 5)
    - `text`: Localized/translated review text and language code
    - `originalText`: Original un-translated review text and source language code
    - `relativePublishTimeDescription`: Human-friendly relative date (e.g., `"2주 전"`, `"1달 전"`)
    - `publishTime`: ISO 8601 UTC timestamp
    - `googleMapsUri`: Direct link to view the review on Google Maps
- **Scenarios**: Restaurant reputation monitoring, multi-language review translation for international tourists, sentiment & star distribution analysis.

### ✍️ 6. Places Autocomplete API (`places:autocomplete`)
- **Inputs**:
  - `input`: Partial keystroke text (e.g., `"남산"`, `"코엑스"`)
  - `sessionToken`: UUID v4 string for billing consolidation
  - `origin`: Coordinates for distance calculation (e.g., Seoul City Hall `(37.5665, 126.9780)`)
  - `includedRegionCodes`: `["kr"]`
- **Outputs**:
  - `placePrediction`: `mainText`, `secondaryText`, `distanceMeters` (straight-line distance from origin), `placeId`, `types`.
- **Scenarios**: Instant search UI with distance display, keystroke billing optimization via Session Tokens.

### 🚗 7. Directions API (`gmaps.directions`)
- **Inputs**:
  - `origin`: `"서울특별시 중구 한강대로 405 서울역"`
  - `destination`: `"서울특별시 강남구 영동대로 513 코엑스"`
  - `waypoints`: `["명동대성당", "남산서울타워", "롯데월드몰"]`
  - `optimize_waypoints`: `True` (Traveling Salesperson Problem route optimizer)
  - `mode`: `"transit"` (for Korea public transit) / `"driving"` (with fallback handling for Korean spatial regulation constraints)
  - `departure_time`: `"now"`
  - `traffic_model`: `"best_guess"`
- **Outputs**:
  - `waypoint_order`, `legs` (`distance`, `duration`, `duration_in_traffic`), `steps` (turn-by-turn HTML and maneuvers), `overview_polyline`.
- **Scenarios**: Multi-stop delivery dispatch order optimization (TSP), public transit guidance, traffic-adjusted ETA.

### 📏 8. Distance Matrix API (`gmaps.distance_matrix`)
- **Inputs**:
  - `origins`: `["서울역", "용산역", "청량리역"]`
  - `destinations`: `["강남역", "여의도 더현대 서울"]`
  - `mode`: `"driving"` / `"transit"`
  - `departure_time`: `"now"`
- **Outputs**:
  - $N \times M$ matrix of `elements`: `distance`, `duration`, `duration_in_traffic`, `status`.
- **Scenarios**: Nearest-driver dispatch calculation, logistics hub candidate evaluation.

### ⛰️ 9. Elevation API (`gmaps.elevation` & `elevation_along_path`)
- **Inputs**:
  - `locations`: Bukhansan Baegundae (836m), Namsan Tower (243m), Lotte World Tower, Yeouido Han River Park.
  - `path` + `samples`: Seoul Station ➡️ Namsan Tower sampled path ($N=5$).
- **Outputs**:
  - `elevation` (meters), `resolution`, elevation slope profile along the trail.
- **Scenarios**: Hiking/cycling elevation gain & slope analysis, drone flight safety clearance.

### ⏰ 10. Time Zone API (`gmaps.timezone`)
- **Inputs**:
  - `location`: Seoul `(37.5665, 126.9780)`, Jeju Island, Dokdo, New York.
  - `timestamp`: UTC epoch timestamp.
- **Outputs**:
  - `timeZoneId` (`Asia/Seoul`), `timeZoneName` (`한국 표준시`), `rawOffset` (+9h), `dstOffset` (0h).
- **Scenarios**: Global flight/hotel booking time calculation, IoT clock sync by GPS.

### 📶 11. Geolocation API (`gmaps.geolocate`)
- **Inputs**:
  - `consider_ip=True`, optional `wifiAccessPoints`, `cellTowers`.
- **Outputs**:
  - `location` (`lat`, `lng`), `accuracy` (radius in meters).
- **Scenarios**: Underground mall / subway positioning where GPS is unavailable, low-power asset tracking.

### 🛣️ 12. Roads API (`gmaps.snap_to_roads`)
- **Inputs**:
  - `path`: GPS coordinates along Gangnam Teheran-ro with artificial noise.
  - `interpolate=True`.
- **Outputs**:
  - `snappedPoints`: Corrected road centerline coordinates, `originalIndex`, `placeId`.
- **Scenarios**: Correcting GPS jitter caused by high-rise urban canyons along Teheran-ro, speed limit compliance auditing.

---

## 🏗️ 4. Notebook Structure & UX Standards

When generating or updating `google_maps_api_test.ipynb`, adhere to the following cell layout:

1. **Title & Executive Matrix (Markdown)**: Comprehensive table of all 12 APIs, methods, input/output attributes, and Seoul scenarios.
2. **Environment & Setup Guide (Markdown)**: Commands for `uv sync` and standard `pip`/`venv`, plus `.env` instructions.
3. **Library Imports & Helper Functions (Code)**: Standard imports, `.env` auto-discovery (`find_dotenv()`), and `print_json()` formatter.
4. **Client Initialization (Code)**: Safe API key retrieval and `googlemaps.Client` setup.
5. **Numbered API Test Sections (Markdown + Code pairs)**:
   - Markdown: Clear list of input parameters, output attributes, and business scenarios.
   - Code: Clean, readable implementation with `pandas.DataFrame` visualization.
   - Exception Handling: `try...except` blocks with descriptive fallback messages and direct Google Cloud Console enablement URLs.
6. **Master Summary Table & Developer References (Markdown)**: Quick reference matrix and links to official documentation.

---

## 🤖 5. Code Generation Recipe (Python Automation)

Always generate `.ipynb` files using Python's `json.dump()` with a structured dictionary rather than string concatenation or regex text replacement to ensure valid `nbformat 4.5` compliance:

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

# Assemble cells array and export to notebook
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

## 🔒 6. Security & Secret Verification Checklist
- [ ] No actual API keys or credentials embedded in markdown or code cells.
- [ ] `.env` is tracked in `.gitignore`.
- [ ] Pre-commit secret scanning executed before every git commit/push.
- [ ] Git commit user matches `ForusOne` (`forusone777@gmail.com`).
