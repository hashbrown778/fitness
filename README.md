# Fitness App MVP

A gamified AI fitness app (Duolingo for fitness) built with React, FastAPI, and Claude API. Right now, it will be a website just because I am learning how to develop things like this but eventually i want to turn it into an app. 

## Tech Stack
- Frontend: React + Tailwind CSS + Recharts
- Backend: FastAPI + PostgreSQL
- AI: Anthropic Claude API

## Setup (Local Development)

### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
python -m uvicorn main:app --reload
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## Status
- [ ] Week 1-2: Auth + database
- [ ] Week 3: Workout logging
- [ ] Week 4: Claude coaching
- [ ] Week 5: React frontend
- [ ] Week 6: Streaks & badges
- [ ] Week 7: Charts
- [ ] Week 8: Deploy