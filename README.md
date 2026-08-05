# RepoPulse AI — Developer Productivity & Repository Health Platform

**Engineering management dashboard** modeled after LinearB, Swarmia, and Jellyfish.  
Connects to live GitHub repositories to track team velocity, analyze code review bottlenecks, measure developer sentiment, and predict pull request (PR) risk using machine learning.

```
┌─────────────────────────┐       WebSockets / REST      ┌─────────────────────────┐
│     Vue 3 Dashboard     │ ◄──────────────────────────► │     Node.js API Gateway │
│ (Pinia, ECharts, Tailwind)                            │   (OAuth, BullMQ, Redis) │
└─────────────────────────┘                              └────────────┬────────────┘
                                                                      │ Async RPC
                                                                      ▼
┌─────────────────────────┐     GitHub REST / GraphQL    ┌─────────────────────────┐
│     PostgreSQL DB       │ ◄──────────────────────────► │    Python ML Service    │
│  (PRs, Metrics, Users)  │                              │ (FastAPI, Pandas, SKLearn)
└─────────────────────────┘                              └─────────────────────────┘
```

## Features

- **GitHub OAuth 2.0** authentication
- **Background ingestion** with BullMQ + Redis (rate-limit aware)
- **DORA Metrics**: Deployment Frequency, Change Failure Rate, MTTR, Lead Time
- **PR Risk Prediction**: RandomForestClassifier scoring (Low / Medium / High)
- **Sentiment Analysis**: HuggingFace DistilBERT on PR/Issue comments
- **Burnout Risk Scoring**: Late-night commits + review lag + backlog
- **Contributor Heatmaps** (GitHub-style via Apache ECharts)
- **Live WebSocket** sync status
- **Multi-tenant** ready architecture
- **Pre-loaded demo repos** (vuejs/core, fastapi/fastapi)

## Tech Stack

| Tier              | Technology                                      |
|-------------------|-------------------------------------------------|
| Frontend          | Vue 3 (Composition API), Pinia, ECharts, Tailwind, Vite |
| Backend / Gateway | Node.js, Express, Prisma, BullMQ, Redis, Socket.io |
| ML Service        | Python, FastAPI, Pandas, Scikit-learn, joblib, HuggingFace |
| Database          | PostgreSQL                                      |
| Infra             | Docker Compose, Nginx                           |

## Quick Start (Docker)

```bash
# 1. Clone & enter
cd RepoPulse-AI

# 2. Copy environment files
cp backend/.env.example backend/.env
cp ml-service/.env.example ml-service/.env
# Edit .env files with your GitHub OAuth App credentials

# 3. Start everything
docker compose up --build

# 4. Open dashboard
open http://localhost:5173   # or http://localhost (via Nginx)
```

### Required Environment Variables

**backend/.env**
```
DATABASE_URL=postgresql://repopulse:repopulse@postgres:5432/repopulse
REDIS_URL=redis://redis:6379
GITHUB_CLIENT_ID=your_github_oauth_app_id
GITHUB_CLIENT_SECRET=your_github_oauth_app_secret
GITHUB_CALLBACK_URL=http://localhost:3000/auth/github/callback
JWT_SECRET=change-me-to-a-long-random-string
ML_SERVICE_URL=http://ml-service:8000
PORT=3000
```

**ml-service/.env**
```
# Optional: model path overrides
MODEL_PATH=./models/pr_risk_model.joblib
```

## Local Development (without full Docker)

### 1. PostgreSQL + Redis
```bash
docker compose up postgres redis -d
```

### 2. Backend
```bash
cd backend
npm install
npx prisma migrate dev
npm run dev
```

### 3. ML Service
```bash
cd ml-service
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

### 4. Frontend
```bash
cd frontend
npm install
npm run dev
```

## Project Structure

```
RepoPulse-AI/
├── backend/                 # Node.js API Gateway + Ingestion
│   ├── prisma/
│   ├── src/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── workers/
│   │   └── ...
│   └── package.json
├── ml-service/              # Python FastAPI + ML
│   ├── app/
│   ├── models/
│   └── requirements.txt
├── frontend/                # Vue 3 Dashboard
│   ├── src/
│   │   ├── components/
│   │   ├── views/
│   │   ├── stores/
│   │   └── ...
│   └── package.json
├── docker-compose.yml
└── README.md
```

## 4-Week Implementation Roadmap (Completed)

- **Week 1**: GitHub OAuth + BullMQ ingestion of PRs/commits/comments → PostgreSQL
- **Week 2**: FastAPI `/predict-pr-risk`, RandomForest model, sentiment pipeline
- **Week 3**: Vue 3 + Pinia + ECharts DORA cards, PR Risk table + drawer, heatmaps
- **Week 4**: Full Docker Compose + demo data seeding

## API Overview

### Backend (Node.js :3000)

| Method | Endpoint                     | Description                     |
|--------|------------------------------|---------------------------------|
| GET    | /auth/github                 | Start OAuth                     |
| GET    | /auth/github/callback        | OAuth callback                  |
| GET    | /api/repos                   | List connected repositories     |
| POST   | /api/repos/:owner/:name/sync | Trigger background sync         |
| GET    | /api/repos/:id/metrics       | DORA + health metrics           |
| GET    | /api/repos/:id/prs           | PRs with risk scores            |
| GET    | /api/repos/:id/contributors  | Contributor metrics + burnout   |
| WS     | /socket.io                   | Live sync status                |

### ML Service (FastAPI :8000)

| Method | Endpoint              | Description                          |
|--------|-----------------------|--------------------------------------|
| POST   | /predict-pr-risk      | Score a single PR                    |
| POST   | /batch-predict        | Score multiple PRs                   |
| POST   | /sentiment            | Analyze comment sentiment            |
| GET    | /health               | Health check                         |

## License

MIT — built as a portfolio / demo project.
