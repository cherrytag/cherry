# 🍒 Cherry Rewards

A mobile-first rewards app with QR scanning, MQTT real-time messaging, geolocation store finder, and a Flask/Postgres backend — deployable to [Render.com](https://render.com) in minutes.

---

## Files

| File | Purpose |
|---|---|
| `cherry_backend.py` | Flask backend — REST API + MQTT client |
| `cherry-rewards.html` | Frontend — single HTML file, no build step |
| `geotags.txt` | Store locations, seeded into Postgres on first boot |
| `requirements.txt` | Python dependencies |
| `render.yaml` | Render Blueprint — deploys web service + Postgres in one click |
| `.env.example` | Template for local environment variables |

---

## Deploy to Render.com (recommended)

### Option A — Blueprint (one-click, creates service + DB together)

1. Push this repo to GitHub.
2. Go to [dashboard.render.com](https://dashboard.render.com) → **New** → **Blueprint**.
3. Connect your GitHub repo — Render will detect `render.yaml` automatically.
4. Click **Apply** — it provisions the web service and a free Postgres database and wires `DATABASE_URL` automatically.
5. Once the deploy is live, copy your service URL (e.g. `https://cherry-rewards.onrender.com`).
6. In `cherry-rewards.html` find the line:
   ```js
   const GEOTAG_API = 'https://your-backend.onrender.com/geotags';
   ```
   Replace with your actual Render URL, commit, and push. Render will auto-redeploy.

### Option B — Manual setup

1. **Web Service**
   - Render dashboard → **New Web Service** → connect your repo.
   - Build command: `pip install -r requirements.txt`
   - Start command: `gunicorn cherry_backend:app`
   - Environment: **Python 3**

2. **Database**
   - Render dashboard → **New PostgreSQL** → create a free instance.
   - Copy the **Internal Database URL** and paste it as the `DATABASE_URL` environment variable on your web service.

3. **Environment variables** (set in Render dashboard → Environment tab)

   | Variable | Value |
   |---|---|
   | `DATABASE_URL` | Render internal Postgres URL |
   | `MQTT_BROKER` | `broker.hivemq.com` (or your broker) |
   | `MQTT_PORT` | `1883` |
   | `MQTT_USER` | *(leave blank for public HiveMQ)* |
   | `MQTT_PASS` | *(leave blank for public HiveMQ)* |
   | `MQTT_CLIENT_ID` | `cherry_backend_prod` |

4. On first boot the backend will:
   - Create `geotags`, `users`, and `rewards` tables.
   - Seed stores from `geotags.txt`.

---

## Run locally

```bash
# 1. Clone and enter the repo
git clone https://github.com/YOUR_USERNAME/cherry-rewards.git
cd cherry-rewards

# 2. Create a virtual environment
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment
cp .env.example .env
# Edit .env — at minimum set DATABASE_URL to a local Postgres instance

# 5. Start the backend
python cherry_backend.py

# 6. Open the frontend
# Open cherry-rewards.html in your browser directly, or serve it:
python -m http.server 8080
# Then visit http://localhost:8080/cherry-rewards.html
```

---

## Add / edit store locations

Edit `geotags.txt` — one store per line:

```
lat, lng, Store Name
-36.8485, 174.7633, Cherry – Auckland City Centre
-33.8688, 151.2093, Cherry – Sydney CBD
```

Lines starting with `#` are ignored. Re-deploy to seed new stores (uses `ON CONFLICT DO NOTHING`, so existing stores won't duplicate).

---

## MQTT topics

| Topic | Direction | Payload |
|---|---|---|
| `cherry/qr_scan` | Frontend → Backend | `{ data, user, timestamp }` |
| `cherry/login` | Frontend → Backend | `{ email, mode, timestamp }` |
| `user_id/rewards` | Backend → Frontend | `{ title, desc, exp }` |
| `111aabc/email` | Backend → Frontend | `{ user_id, email }` |

---

## Health check

```
GET https://your-backend.onrender.com/
→ { "status": "ok", "service": "Cherry Rewards Backend" }

GET https://your-backend.onrender.com/geotags?lat=-36.8485&lng=174.7633&radius=8000
→ [ { "name": "...", "lat": ..., "lng": ..., "distance_m": ... }, ... ]
```
