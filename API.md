# API Reference — VisualizerWeb Backend

This document describes the Flask backend API exposed by the project.

> Base URL (dev): `http://127.0.0.1:5000`

## Endpoints

### POST `/compute`
Runs the selected machine-learning algorithm on the points from the canvas and returns results for visualization.

- **Content-Type:** `application/json`
- **CORS:** Enabled via `Flask-Cors`

#### Request body (generic shape)
```json
{
  "mlAlgoName": "Regression Tree",          // algorithm to run (examples below)
  "mlAlgoOptions": { ... },                  // hyperparameters; keys come from schema/*.json
  "points": [                                // dataset drawn in the UI (screen coords)
    {"x": 1200, "y": 360, "id": "p0", "strokeColor":"black"},
    {"x": 1180, "y": 380, "id": "p1", "strokeColor":"black"}
  ],
  "predictionData": []                       // optional, used by some tools
}
```

> **Where these fields are parsed:** `backend/algo/compute.py`  
> **Where options are defined:** `backend/schema/*.json` (e.g., `decision_tree_schema.json`, `hierarchical_schema.json`).

#### Example: Linear Regression
```json
{
  "mlAlgoName": "Linear Regression",
  "mlAlgoOptions": { "Degree": 1 },            // (example) degree for polynomial fit
  "points": [{"x": 1200, "y": 360}, {"x": 1180, "y": 380}]
}
```

#### Example: Regression Tree (Decision Tree)
```json
{
  "mlAlgoName": "Regression Tree",
  "mlAlgoOptions": {
    "Depth": 3,
    "Samples": 5
  },
  "points": [{"x": 1200, "y": 360}, {"x": 1180, "y": 380}]
}
```

#### Example: Hierarchical Clustering
```json
{
  "mlAlgoName": "Hierarchical",
  "mlAlgoOptions": {
    "Linkage": "ward",           // or: "complete", "average", "single"
    "Clusters": 3                // or: "DistanceThreshold": 12.0 (depending on schema)
  },
  "points": [{"x": 520, "y": 110}, {"x": 600, "y": 150}, {"x": 700, "y": 200}]
}
```

> **Note on coordinates:** The frontend sends **screen coordinates**. The backend converts/normalizes them for sklearn (see helpers in `backend/algo` or `backend/compute.py`).

#### Response (generic)
```json
{
  "specific": "Regression Tree",      // echoes the algorithm category/name
  "labels": [...],                    // algorithm-specific labels/series for drawing
  "prediction": ""                    // algorithm-specific; may be empty or a series
}
```

The exact structure varies by algorithm. Inspect the frontend drawing code (`frontend/src`) to see how it consumes `labels`/`prediction` for each model.

---

## Errors
- **405 Method Not Allowed** — wrong HTTP method (use POST).
- **415 Unsupported Media Type** — missing or wrong `Content-Type` (set `application/json`).
- **400 Bad Request** — malformed body or missing required options; check PyCharm console for the stack trace.

---

## Dev Notes
- Single route defined in `backend/backend.py`:
  ```python
  @app.route('/compute', methods=['POST'])
  def post():
      return app.response_class(
          response=json.dumps(compute(request.get_json())),
          status=200,
          mimetype='application/json')
  ```
- All dispatching happens in `backend/algo/compute.py`.
- Parameters exposed in the UI are described in `backend/schema/*.json`.
