# Google Maps Platform APIs Test Suite

A comprehensive test suite and Jupyter Notebook for exploring and validating Google Maps Platform Web Service APIs and the modern Places API (New v1).

## 📌 Features & Supported APIs

- **Geocoding API**: Address to lat/lng coordinates, address components, location type, and viewports.
- **Reverse Geocoding API**: Coordinates to human-readable structured address and political boundaries.
- **Places API (New)**:
  - Text Search (`places:searchText`)
  - Nearby Search (`places:searchNearby`)
  - Place Details (`places/{placeId}`) with wildcard fieldmask (`*`)
- **Places Autocomplete API**: Real-time search predictions and place suggestions.
- **Directions API**: Turn-by-turn routing, alternatives, and real-time traffic durations.
- **Distance Matrix API**: Multi-origin to multi-destination distance & travel duration matrices.
- **Elevation API**: Altitude measurements for specific coordinates.
- **Time Zone API**: UTC offsets and Daylight Saving Time (DST) information.
- **Geolocation API**: Network & IP-based device location estimation.

## 🚀 Getting Started

### 1. Prerequisites & Environment Setup

This project uses [`uv`](https://github.com/astral-sh/uv) for fast Python dependency management.

```bash
# Clone the repository
git clone https://github.com/shins777/google_apis.git
cd google_apis

# Install dependencies and sync environment
uv sync

# Register Jupyter kernel
uv run python -m ipykernel install --user --name google_apis_env --display-name "Python (google_apis_env)"
```

Alternatively, with standard `pip` and `venv`:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
python -m ipykernel install --user --name google_apis_env --display-name "Python (google_apis_env)"
```

### 2. API Key Configuration

Create a `.env` file in the root directory (based on `.env.example`):

```env
GOOGLE_MAPS_API_KEY=your_google_maps_api_key_here
```

Ensure the required APIs are enabled in your [Google Cloud Console](https://console.cloud.google.com/apis/dashboard).

### 3. Running the Notebook

Open and run `google_map/google_maps_api_test.ipynb` in VS Code or JupyterLab.
