# SIH26015 Backend — GEE + FastAPI

## 1. Setup (5 min)

```bash
cd sih26015-backend
pip install -r requirements.txt --break-system-packages   # drop the flag if not needed on your machine
earthengine authenticate                                  # opens Google login, one-time
```

Verify:
```bash
python -c "import ee; ee.Initialize(); print('Earth Engine connected!')"
```

## 2. Run

```bash
uvicorn app.main:app --reload
```

- API: http://127.0.0.1:8000
- Interactive docs (test without any frontend): http://127.0.0.1:8000/docs

## 3. Verify it works — open `test.html` directly in your browser

This is a **standalone page**, independent of your teammates' frontend. It calls your
running backend and renders the NDVI layer on a Leaflet map with a status line
showing vegetation/water classification. Use this to prove the backend works
*before* touching the real frontend integration — saves you debugging two things
at once.

## 4. Connecting to your teammates' actual frontend

Once you know their stack, the only two things that matter:

1. **The endpoint**: `POST http://127.0.0.1:8000/analyze`
   ```json
   {
     "latitude": 17.2403,
     "longitude": 78.4294,
     "start_date": "2026-06-01",
     "end_date": "2026-09-01",
     "radius_meters": 5000
   }
   ```

2. **The response** — the key field is `tile_url`, a ready XYZ template string.
   Any map library (Leaflet, Mapbox GL, MapLibre) can add it directly as a raster layer:
   ```json
   {
     "status": "success",
     "image_count": 12,
     "center": { "lat": 17.2403, "lon": 78.4294 },
     "ndvi": { "mapid": "...", "token": "...", "tile_url": "https://earthengine.googleapis.com/.../{z}/{x}/{y}" },
     "ndwi": { "mapid": "...", "token": "...", "tile_url": "https://earthengine.googleapis.com/.../{z}/{x}/{y}" },
     "analysis": { "ndvi_mean": 0.43, "ndwi_mean": 0.12, "vegetation": "Moderate", "water_moisture": "Low" }
   }
   ```

   In Leaflet: `L.tileLayer(data.ndvi.tile_url).addTo(map)`
   In Mapbox GL / MapLibre: add as a `raster` source with `tiles: [data.ndvi.tile_url]`

If their frontend expects a different field naming or GeoJSON instead of tiles,
tell me what they've got (a fetch call, a form, an API client file) and I'll
adjust `routes.py` to match — much faster than changing their code.

## 5. If you're deploying (not just running on localhost for the demo)

Swap `ee.Authenticate()` for a **service account** — local user auth won't work
on a server. Ask me for this if deployment comes up; for a same-laptop demo at
10AM, localhost is fine.

## Common issues

- **"No cloud-free Sentinel-2 images found"** → widen `start_date`/`end_date`,
  or raise the `CLOUDY_PIXEL_PERCENTAGE` threshold in `earth_engine.py` (default 20).
- **CORS errors in browser console** → already handled (`allow_origins=["*"]`),
  but double check the frontend is calling `http://127.0.0.1:8000`, not `localhost`
  (browsers treat these as different origins sometimes) — use whichever one is in `API_URL`.
- **`ee.Initialize()` fails** → re-run `earthengine authenticate`, make sure your
  Google account is EE-enabled at https://code.earthengine.google.com/
