# ESSL Docker Attendance System

![Deploy on Render](https://img.shields.io/badge/Deploy-Render-blue?logo=render)
![Database Neon](https://img.shields.io/badge/Database-Neon-green?logo=postgresql)
![Docker](https://img.shields.io/badge/Container-Docker-blue?logo=docker)

A containerized attendance tracking system with:
- **Flask API** service for managing employees and attendance events
- **Dash Dashboard** for visualizing daily presence and worked hours
- **PostgreSQL** database (local via Docker Compose; cloud via Neon/Render)

---

## 📁 Project Structure
```
essl-docker-attendance-system/
├── .env.example                     # Environment variable template
├── README.md
├── app/                             # Flask API service
└── dashboard/                       # Dash dashboard service
```

---

## ✅ Environment Variables
Use `.env.example` as a reference for required variables:
```
DB_HOST=your-neon-host
DB_PORT=5432
DB_NAME=your-database-name
DB_USER=your-database-user
DB_PASSWORD=your-database-password
TZ=Asia/Kolkata
PGSSLMODE=require
PORT=8000
```

---

## 🌐 Free Cloud Deployment: Render + Neon
1. **Create Neon DB** → copy credentials.
2. **Deploy API service on Render**:
   - Root: `app/`
   - Port: `8000`
   - Add env vars from `.env.example`
3. **Deploy Dashboard service on Render**:
   - Root: `dashboard/`
   - Port: `8050`
4. **Seed data**:
   ```bash
   python fake_data.py
   ```

---

## 📸 Screenshots
*(Add screenshots of your dashboard here after deployment)*
Example:
![Dashboard Screenshot](https://your-screenshot-link)

---

## 🧪 Local Setup
```bash
docker compose build
docker compose up -d
docker compose exec app python /app/fake_data.py
```

Access:
- API: http://localhost:8000
- Dashboard: http://localhost:8050

Stop/Cleanup:
```bash
docker compose down
docker compose down -v
```

---

## 📜 License
MIT License
