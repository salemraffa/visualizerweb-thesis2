# Architecture — VisualizerWeb

This document summarizes how data flows between the **frontend** (Quasar/Vue) and the **backend** (Flask) and where to extend functionality for your thesis.

## High-level flow
```
User draws points / sets params (frontend)
        ↓
Frontend POST /compute with JSON
        ↓
Flask backend (backend/backend.py)
        ↓
compute(request_json) — backend/algo/compute.py
        ↓
Dispatch to algorithm module (backend/algo/*)
        ↓
sklearn computes result
        ↓
Backend returns JSON labels/predictions
        ↓
Frontend renders polylines/segments/labels on canvas
```

## Backend layout
```
backend/
  backend.py                 # Flask entrypoint; defines POST /compute
  compute.py                 # (if present) orchestration helpers
  algo/
    classification/
    regression/
      regressiontree.py      # Decision tree regression (sklearn.tree)
      linear_regression.py
    clustering/
      hierarchical.py        # Agglomerative / linkage logic
  schema/
    decision_tree_schema.json
    hierarchical_schema.json
    ...                      # parameter schemas that drive the UI
  validate.py / helpers.py   # request validation / normalization (if present)
```

### What `compute()` typically does
1. Read JSON: `mlAlgoName`, `mlAlgoOptions`, `points`, `predictionData?`.
2. Map `mlAlgoName` → concrete function (e.g., `regression.regressiontree.run(...)`).
3. Convert screen coordinates into numeric arrays for sklearn.
4. Call sklearn model(s) with the provided options.
5. Convert raw results back into a JSON the frontend understands:
   - `specific` — algorithm name/category
   - `labels` — arrays/segments/series to draw
   - `prediction` — optional predicted series or summary

## Frontend layout (Quasar/Vue)
```
frontend/
  quasar.conf.js (v1) or quasar.config.js (v2)
  src/
    boot/axios.js (or similar)           # Axios instance using BACKENDURL
    pages/                                # Screens that host the canvas + controls
    components/                           # Widgets for toolbar, parameter panels, plots
```

### API calls
Look for code similar to:
```js
this.$axios.post(process.env.BACKENDURL + '/compute', payload)
```
or a wrapped API module. The payload structure mirrors the JSON in the API doc.

## Where to extend for your thesis

### Decision Tree (regression)
- **Backend:** `backend/algo/regression/regressiontree.py`
  - Expose more hyperparameters in the schema (e.g., `max_leaf_nodes`, `min_impurity_decrease`).
  - Return split thresholds, leaf predictions, and tree structure for richer visualization.
- **Schema:** `backend/schema/decision_tree_schema.json`
  - Add fields; ensure IDs/keys match what the backend expects.
- **Frontend:** the component that renders tree segments/regions.
  - Add overlays/labels for thresholds and leaves; provide tooltips.

### Hierarchical clustering
- **Backend:** `backend/algo/clustering/hierarchical.py`
  - Return `children_`, `distances_`, or a reduced summary to render a mini dendrogram or merge order.
- **Schema:** `backend/schema/hierarchical_schema.json`
  - Add `metric`, `linkage`, `distance_threshold` or `n_clusters` controls.
- **Frontend:** add a small dendrogram or merge-step list; tie selection to highlighting points.

## Debugging tips
- Run backend from PyCharm; set breakpoints in `compute()` and in specific algorithm files.
- Use Postman to replay payloads quickly and tweak params.
- Keep `quasar.conf.js` pointing to `http://localhost:5000` for dev.
