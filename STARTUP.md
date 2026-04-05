# Startup Guide — ISL Recognition System

This guide walks you through starting the full stack from scratch every time you want to run the app.

---

## What You Need Running

| Service | Port | Purpose |
|---|---|---|
| Redis | 6379 | Message broker for Celery |
| Backend (uvicorn) | 8000 | FastAPI + WebSocket server |
| Celery Worker | — | Async video processing |
| Frontend (Vite) | 5173 | React UI |

Open **4 separate terminal windows** and follow the steps below in order.

---

## Terminal 1 — Redis

Redis must start first. Use WSL2 (recommended on Windows):

```
wsl
sudo service redis-server start
```

Verify it's running:
```
redis-cli ping
```
Expected output: `PONG`

> If you get an error, try: `sudo service redis-server restart`

---

## Terminal 2 — Backend (FastAPI)

Open a **PowerShell** terminal in the project folder:

```
cd "k:\AI project\isl-recognition\backend"
venv\Scripts\activate
uvicorn app.main:app --reload --port 8000
```

Wait until you see:
```
INFO  Dynamic model loaded. Classes: 296
INFO  Sentence model loaded.
INFO  Application startup complete.
INFO  Uvicorn running on http://0.0.0.0:8000
```

> If models show ✅ in the frontend, the backend is ready.

API docs available at: **http://localhost:8000/api/docs**

---

## Terminal 3 — Celery Worker

Open another **PowerShell** terminal:

```
cd "k:\AI project\isl-recognition\backend"
venv\Scripts\activate
celery -A app.workers.celery_app worker --loglevel=info --pool=solo
```

Wait until you see:
```
[tasks]
  . app.workers.tasks.process_video

[config]
  .> broker: redis://localhost:6379/1

ready.
```

> `--pool=solo` is required on Windows. Do not remove it.

---

## Terminal 4 — Frontend (React + Vite)

Open another **PowerShell** terminal:

```
cd "k:\AI project\isl-recognition\frontend"
pnpm dev
```

Wait until you see:
```
  VITE v5.x.x  ready in xxx ms

  ➜  Local:   http://localhost:5173/
```

Open **http://localhost:5173** in your browser.

---

## Startup Order Summary

```
1. WSL2 terminal     →  sudo service redis-server start
2. PowerShell        →  cd backend && venv\Scripts\activate && uvicorn app.main:app --reload --port 8000
3. PowerShell        →  cd backend && venv\Scripts\activate && celery -A app.workers.celery_app worker --loglevel=info --pool=solo
4. PowerShell        →  cd frontend && pnpm dev
```

Then open: **http://localhost:5173**

---

## Stopping Everything

| Terminal | How to stop |
|---|---|
| Redis | `sudo service redis-server stop` in WSL2 |
| Backend | `Ctrl + C` |
| Celery | `Ctrl + C` |
| Frontend | `Ctrl + C` |

---

## Troubleshooting

### Models show ❌ in the frontend

The model files are not included in the repo. You need to train them first:

```
cd "k:\AI project\isl-recognition\ml\training"
# Activate your Python environment first
python train_dynamic.py
```

See the main [README.md](README.md) for full training instructions.

### Camera is black / not working

- Make sure your browser has **camera permission** granted for `localhost:5173`
- Check in Chrome: address bar → lock icon → Site settings → Camera → Allow
- Try a hard refresh: `Ctrl + Shift + R`

### Backend crashes on startup

- Check that your `.env` file exists: copy `.env.example` to `.env`
- Check that `DYNAMIC_MODEL_PATH` in `.env` points to the correct folder
- Make sure the `venv` is activated before running uvicorn

### Redis connection refused

- Make sure WSL2 is running: open a WSL2 terminal and run `sudo service redis-server start`
- Or use Docker: `docker run -d -p 6379:6379 redis:7-alpine`

### Celery PermissionError on Windows

Always use `--pool=solo` on Windows:
```
celery -A app.workers.celery_app worker --loglevel=info --pool=solo
```

### Port already in use

```
# Find and kill process on port 8000
netstat -ano | findstr :8000
taskkill /PID <PID> /F

# Find and kill process on port 5173
netstat -ano | findstr :5173
taskkill /PID <PID> /F
```

### pnpm not found

Install pnpm globally:
```
npm install -g pnpm
```

---

## First Time Setup (one-time only)

If this is your first time running the project, do this before the startup steps above:

### Backend dependencies

```
cd "k:\AI project\isl-recognition\backend"
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

### Frontend dependencies

```
cd "k:\AI project\isl-recognition\frontend"
pnpm install
```

### Environment file

```
cd "k:\AI project\isl-recognition"
copy .env.example .env
```

Edit `.env` and set the model paths if needed (defaults work if you trained locally).

---

## Quick Health Check

Once everything is running, verify:

| Check | URL |
|---|---|
| Backend health | http://localhost:8000/api/health |
| API docs | http://localhost:8000/api/docs |
| Frontend | http://localhost:5173 |

The health endpoint returns model load status:
```json
{
  "status": "ok",
  "models": {
    "dynamic": true,
    "sentence": true,
    "static": false
  }
}
```
