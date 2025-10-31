# 🧭 Project Setup Guide — *VisualizerWeb Thesis 2*

This guide explains how to set up and run the **VisualizerWeb** application locally without Docker.

It reproduces the working environment tested on **Windows 10** with  
**Python 3.10** and **Node.js 16.20.2**.

---

## 📁 Repository
```
https://github.com/MariamFech/visualizerweb.git
```

Cloned into:
```
C:\Users\salem\PycharmProjects\visualizerweb-thesis2
```

---

## 🧱 1. Backend Setup (Flask)

> **Important:** Use **Python 3.10**, not 3.12.

### Steps
```cmd
cd C:\Users\salem\PycharmProjects\visualizerweb-thesis2\backend
py -3.10 -m venv .venv
.\.venv\Scripts\activate
python -m pip install --upgrade pip setuptools wheel
pip install -r requirements.txt
```

### Run the server
```cmd
set FLASK_APP=backend.py
set FLASK_ENV=development
python -m flask run
```

**Expected output:**
```
* Running on http://127.0.0.1:5000/
```

Keep this terminal open — it serves the backend API.

---

## 🌐 2. Frontend Setup (Quasar / Vue)

> **Requirement:** Node.js ≥ 14 and < 18 (tested on 16.20.2)

### Steps
```cmd
cd C:\Users\salem\PycharmProjects\visualizerweb-thesis2\frontend
npm install --legacy-peer-deps
```

### Backend connection fix
Open `frontend/quasar.config.js` (or `quasar.conf.js` in older versions) and set:
```js
env: {
  BACKENDURL: "http://localhost:5000"
}
```

### Run the frontend
```cmd
npx quasar dev
```

**Expected output:**
```
App running at: http://localhost:8080/
```

---

## 🔗 3. Access the Application
Open a browser and visit:

👉 **http://localhost:8080**

The web interface should load and connect successfully to your Flask backend at **http://localhost:5000**.

---

## 🧩 4. Troubleshooting

| Problem | Fix |
|----------|-----|
| `Cannot import 'setuptools.build_meta'` or `No module named flask` | Use Python 3.10, not 3.12 |
| `flask --app` not recognized | Old Flask (v1.1.2) — use `set FLASK_APP=backend.py` |
| `npm ERR! ERESOLVE` when installing | Run `npm install --legacy-peer-deps` |
| `Unknown command dev` | Dependencies not installed — reinstall as above |
| Frontend can’t reach backend | Check `BACKENDURL` = `http://localhost:5000` |
| Port conflict | Run backend on `python -m flask run -p 5001` and update `BACKENDURL` |

---

## ✅ Environment summary

| Component | Version | Notes |
|------------|----------|-------|
| Python | 3.10.x | Required for old scientific stack |
| Flask | 1.1.2 | Old CLI syntax |
| Node.js | 16.20.2 | Compatible with Quasar v2 |
| Quasar | 2.0.0-beta.9 | Legacy dev build |
| Backend Port | 5000 | Flask |
| Frontend Port | 8080 | Quasar |
