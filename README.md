
# ESSL Docker Attendance System

A containerized attendance tracking system with:

- **Flask API** service for managing employees and attendance events
- **Dash Dashboard** for visualizing daily presence and **worked hours**
- **PostgreSQL** database (local via Docker Compose; cloud via Neon/Render)

This repo is production‑ready for free deployment on **Render** with a free **Neon** Postgres instance.

---

## 📁 Project Structure

```
essl-docker-attendance-system/
├── docker-compose.yml                 # Local development only
├── app/                               # Flask API service
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app.py                         # Endpoints, summaries, workhours
│   ├── database.py                    # SQLAlchemy models & engine
│   └── fake_data.py                   # Demo data seeding
└── dashboard/                         # Dash dashboard service
    ├── Dockerfile
    ├── requirements.txt
    └── dashboard.py                   # Visualizations & latest events
```

---

## 🚀 Features

- Employees CRUD (minimal create/list)
- Attendance events ingestion (**IN / OUT**) with ISO8601 timestamps
- **Daily summary** of IN events
- **Worked hours per day** (pairs IN→next OUT, clamps outliers)
- Health check endpoint for platform readiness
- Timezone aware grouping (default **Asia/Kolkata**, configurable via `TZ`)

**Work Hours Logic**
- Pairs each `IN` with the **immediately following** `OUT` for same employee
- Groups by **local date of IN** (`DATE(in_time AT TIME ZONE TZ)`)
- Ignores open sessions (IN without subsequent OUT)
- Clamps each IN→OUT segment to **16 hours** by default; configurable via `max_hours`

---

## 🔌 API Endpoints

Base URL (local): `http://localhost:8000`

| Method | Path                     | Description |
|--------|--------------------------|-------------|
| GET    | `/health`                | Service health check |
| GET    | `/employees`             | List employees |
| POST   | `/employees`             | Create employee `{name, department?}` |
| GET    | `/attendance`            | List attendance events (optional `?employee_id`) |
| POST   | `/attendance`            | Create event `{employee_id, event_type: IN|OUT, event_time: ISO8601}` |
| GET    | `/summary/daily`         | IN counts per day per employee |
| GET    | `/summary/workhours`     | Worked hours per day per employee. Query params: `days`, `max_hours` |

**Example**
```bash
curl "http://localhost:8000/summary/workhours?days=7&max_hours=12"
```

---

## 🧪 Quick Start (Local with Docker Compose)

> Use this for local development/testing. Cloud deploy (Render+Neon) is below.

1. **Prerequisites**: Docker & Docker Compose installed
2. **Build images**
   ```bash
   docker compose build
   ```
3. **Start services**
   ```bash
   docker compose up -d
   ```
4. **Seed demo data** (employees + last 7 days attendance)
   ```bash
   docker compose exec app python /app/fake_data.py
   ```
5. **Test API**
   ```bash
   curl http://localhost:8000/health
   curl http://localhost:8000/employees
   curl http://localhost:8000/summary/workhours?days=7
   ```
6. **Open Dashboard**: http://localhost:8050

**Stop/Cleanup**
```bash
docker compose down          # stop containers
docker compose down -v       # also remove DB volume
```

---

## 🌐 Free Cloud Deployment: Render + Neon

This project is designed to run free online using **Render** (API + dashboard) and **Neon** (managed Postgres).

### 1) Create a Neon Postgres
- Sign up: https://neon.tech
- Create a project (PostgreSQL 16)
- Copy connection string:
  `postgresql://<user>:<password>@<host>/<db>?sslmode=require`
- Note values: `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_PORT=5432`

### 2) Deploy API service on Render
- Sign up: https://render.com and connect your GitHub repo
- Create **New → Web Service**
- **Root Directory**: `app/`
- **Environment**: Docker (autodetects `Dockerfile`)
- **Port**: `8000`
- **Environment Variables**:
  ```
  DB_HOST=<neon host>
  DB_PORT=5432
  DB_NAME=<neon database>
  DB_USER=<neon user>
  DB_PASSWORD=<neon password>
  TZ=Asia/Kolkata
  PGSSLMODE=require
  ```
- Deploy → after successful deploy, open **Shell** in the service and run:
  ```bash
  python fake_data.py
  ```

### 3) Deploy Dashboard service on Render
- Create **New → Web Service**
- **Root Directory**: `dashboard/`
- **Environment**: Docker
- **Port**: `8050`
- **Environment Variables**: same as API
- Deploy and visit the Render URL (e.g., `https://attendance-dashboard.onrender.com`)

### 4) Verify
- API Health: `https://<api-service>.onrender.com/health`
- Workhours Summary: `https://<api-service>.onrender.com/summary/workhours?days=7`
- Dashboard: `https://<dashboard-service>.onrender.com`

> **Free-tier notes:** Render free instances may sleep after inactivity (~15 min), causing a 30–60s cold start. Keep images lean (`python:3.11-slim` already).

---

## ⚙️ Configuration

**Environment Variables**
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`: Postgres connection
- `PGSSLMODE=require`: ensure SSL when using Neon
- `TZ`: timezone for grouping by date (default `Asia/Kolkata`)
- `PORT`: optional override of service port (API=8000, Dashboard=8050)

**Database URL construction** (handled in `database.py`):
`postgresql+psycopg2://<user>:<pass>@<host>:<port>/<db>`

> If your provider requires explicit SSL in URL, append `?sslmode=require`.

---

## 🛠️ Development Notes

- **CORS**: If you switch the dashboard to call API endpoints (instead of direct DB queries), add CORS:
  ```bash
  pip install flask-cors
  ```
  ```python
  from flask_cors import CORS
  CORS(app)
  ```
- **Migrations**: For complex schemas, add Alembic.
- **Auth**: Protect endpoints using JWT or a reverse proxy (NGINX) for production.
- **Observability**: Add structured logging and `/metrics` if you use Prometheus.

---

## 🧩 Troubleshooting

- **Render build fails**: Check Dockerfile path and Root Directory setting per service
- **DB connection error**: Verify `PGSSLMODE=require`, host, port, and credentials; ensure Neon project is running
- **No data on dashboard**: Run `python fake_data.py` on the API service to seed demo rows
- **Timezone mismatch**: Set `TZ` env var to your locality (e.g., `Asia/Kolkata`)

---

## 📜 License

MIT License — feel free to use and adapt.

---

## 📦 Contributing

PRs are welcome. Please open issues for bugs or feature requests.

