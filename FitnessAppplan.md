# Fitness App MVP — 8-Week Build Plan

**Project Name:** [Your Choice — e.g., Gainz, StreakFit, LevelUp]

**Vision:** A gamified AI fitness app (Duolingo for fitness) where users log workouts, get personalized Claude API coaching feedback, and track progress through streaks, badges, and charts.

---

## Tech Stack

| Layer | Tools |
|-------|-------|
| **Frontend** | React (Vite) + Tailwind CSS + Recharts |
| **Backend** | Python 3.10+ + FastAPI + SQLAlchemy |
| **Database** | PostgreSQL (local) → Render/Railway (production) |
| **Auth** | JWT tokens + bcrypt |
| **AI** | Anthropic Claude API |
| **Hosting** | Vercel (frontend) + Render/Railway (backend) |

---

## Timeline (15-18 hrs/week)

### **Pre-Week 1: Setup**
**Time: 2-3 hours**

**Tasks:**
- [ ] Install PostgreSQL (local development)
- [ ] Install Python 3.10+
- [ ] Install Node.js + npm
- [ ] Install VSCode extensions (Python, REST Client or Postman)
- [ ] Create GitHub repo (public, for portfolio)
- [ ] Set up Anthropic API key in environment variables
- [ ] Pick project name & domain

**Resources:**
- PostgreSQL: https://www.postgresql.org/download/
- FastAPI: https://fastapi.tiangolo.com/
- Node.js: https://nodejs.org/

---

### **Weeks 1-2: Database + Backend Auth**
**Time: 15-20 hours**

**Goal:** Get FastAPI running with user authentication and database connected.

**Tasks:**
- [ ] Learn FastAPI basics (~2-3 hours)
  - Routes, request/response models, dependency injection
  - Official tutorial: https://fastapi.tiangolo.com/tutorial/

- [ ] Create backend folder structure:
  ```
  /backend
    /routes
      __init__.py
      auth.py
      workouts.py
      achievements.py
    /schemas
      __init__.py
      user.py
      workout.py
    __init__.py
    main.py
    models.py
    database.py
    config.py
  ```

- [ ] Design and implement database schema (see below)

- [ ] Set up SQLAlchemy ORM with PostgreSQL

- [ ] Implement user authentication:
  - Signup endpoint: `POST /auth/signup`
  - Login endpoint: `POST /auth/login`
  - Password hashing (bcrypt)
  - JWT token generation
  - Protected routes (require valid JWT)

- [ ] Test authentication with Postman/REST Client

**Database Schema:**

```sql
-- Users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Exercises (seed with ~50 common exercises)
CREATE TABLE exercises (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    category VARCHAR(50)  -- strength, cardio, mobility, etc.
);

-- Workouts
CREATE TABLE workouts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    exercise_id INTEGER REFERENCES exercises(id),
    sets INTEGER,
    reps INTEGER,
    weight FLOAT,  -- in lbs or kg
    rpe FLOAT,  -- Rate of Perceived Exertion (1-10 scale)
    notes TEXT,
    logged_at TIMESTAMP DEFAULT NOW()
);

-- Coaching Feedback
CREATE TABLE feedback (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    workout_id INTEGER REFERENCES workouts(id),
    feedback_text TEXT,
    generated_at TIMESTAMP DEFAULT NOW()
);

-- Streaks
CREATE TABLE streaks (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    current_streak INTEGER DEFAULT 0,
    longest_streak INTEGER DEFAULT 0,
    last_workout_date DATE
);

-- Achievements/Badges
CREATE TABLE achievements (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    achievement_type VARCHAR(100),  -- "5_day_streak", "new_pr", "100_workouts", etc.
    unlocked_at TIMESTAMP DEFAULT NOW()
);
```

---

### **Week 3: Workout Logging API**
**Time: 10-12 hours**

**Goal:** Users can log, retrieve, and manage workouts via API.

**Tasks:**
- [ ] Create workout endpoints:
  - `POST /workouts` — log a new workout
  - `GET /workouts` — fetch user's workout history (paginated)
  - `GET /workouts/{id}` — get single workout details
  - `PUT /workouts/{id}` — edit a workout
  - `DELETE /workouts/{id}` — delete a workout

- [ ] Create exercise endpoints:
  - `GET /exercises` — list all exercises (paginated)
  - Seed database with common exercises (squat, deadlift, bench, row, OHP, etc.)

- [ ] Input validation:
  - Sets/reps must be positive integers
  - Weight must be positive
  - RPE must be 1-10
  - Exercise must exist

- [ ] Test all endpoints with sample data (Postman/REST Client)

**Example Request:**
```json
POST /workouts
{
    "exercise_id": 1,
    "sets": 4,
    "reps": 8,
    "weight": 225,
    "rpe": 8
}
```

---

### **Week 4: Claude API Integration**
**Time: 8-10 hours**

**Goal:** Claude analyzes user workout data and generates personalized coaching feedback.

**Tasks:**
- [ ] Learn Claude API basics (~1 hour)
  - Official docs: https://docs.anthropic.com/
  - Make test API call

- [ ] Design coaching prompt:
  - Fetch user's last 30 days of workouts
  - Calculate: volume trends, intensity, consistency, progression
  - Send to Claude with context
  - Example prompt: "Analyze this user's workouts and give 2-3 sentences of personalized coaching feedback"

- [ ] Implement `/workouts/{id}/feedback` endpoint:
  - Fetch recent workout data for context
  - Call Claude API with structured prompt
  - Store feedback in database
  - Return feedback to frontend

- [ ] Error handling:
  - If Claude API fails, return graceful error
  - Cache feedback (don't regenerate if already exists)

- [ ] Test: Log a few workouts, get coaching feedback

**Example Claude Prompt:**
```
You are a fitness coach. Analyze this user's recent workouts and provide 1-2 sentences of personalized, encouraging coaching feedback. Focus on progression, volume trends, and form considerations.

Recent workouts (last 30 days):
- Squat: 225 lbs x 8 reps (RPE 7) — 3 sessions
- Squat: 225 lbs x 8 reps (RPE 8) — 2 sessions
- Squat: 235 lbs x 5 reps (RPE 9) — 1 session

Feedback:
```

---

### **Week 5: React Frontend — Core UI**
**Time: 12-15 hours**

**Goal:** Build login, workout logger, and basic dashboard.

**Tasks:**
- [ ] Set up React app:
  ```bash
  npm create vite@latest frontend -- --template react
  cd frontend
  npm install
  npm install -D tailwindcss postcss autoprefixer
  npm install axios react-router-dom
  ```

- [ ] Folder structure:
  ```
  /frontend/src
    /pages
      AuthPage.jsx
      DashboardPage.jsx
      LogWorkoutPage.jsx
    /components
      Navbar.jsx
      WorkoutForm.jsx
      WorkoutHistory.jsx
    /api
      auth.js
      workouts.js
    /utils
      auth.js  (token storage)
      constants.js
    App.jsx
    main.jsx
  ```

- [ ] Build pages:
  - **AuthPage:** Signup/login form with email/password
  - **DashboardPage:** Stub for now (will add streaks, badges, charts later)
  - **LogWorkoutPage:** Form to log a workout (exercise dropdown, sets, reps, weight, RPE)

- [ ] API integration:
  - Create `api/auth.js` with signup/login functions
  - Create `api/workouts.js` with workout CRUD functions
  - Handle JWT tokens (store in localStorage, attach to headers)

- [ ] Routing:
  - Public routes: `/auth` (signup/login)
  - Protected routes: `/dashboard`, `/log-workout` (require JWT)

- [ ] Test: Signup, login, basic workout form submission

---

### **Week 6: Streak + Achievement Logic**
**Time: 10-12 hours**

**Goal:** Track streaks, unlock badges, display on dashboard.

**Tasks:**
- [ ] Backend streak logic:
  - Calculate current streak (consecutive days with a workout)
  - Reset if >1 day gap
  - Track longest streak
  - Endpoint: `GET /streaks` — fetch user's current + longest streak

- [ ] Backend achievement system:
  - Define achievement types:
    - `5_day_streak`, `7_day_streak`, `30_day_streak`
    - `new_pr_` (personal record per exercise)
    - `50_workouts`, `100_workouts`
    - `3_workouts_this_week`
  - Create endpoint: `POST /achievements/check` — checks if user unlocked any new achievements
  - Trigger automatically after each workout

- [ ] Frontend dashboard:
  - Display current streak (big, prominent number)
  - Show unlocked badges as cards/icons
  - Show next badge goal ("3 more workouts for 5-day streak!")
  - Simple achievement gallery

- [ ] Test: Log workouts across multiple days, watch streak increment, unlock badges

---

### **Week 7: Charts + Progress Visualization**
**Time: 10-12 hours**

**Goal:** Show strength gains, volume trends, and consistency over time.

**Tools:** Recharts (React charting library)

**Tasks:**
- [ ] Install recharts:
  ```bash
  npm install recharts
  ```

- [ ] Create backend endpoints for chart data:
  - `GET /stats/strength?exercise_id={id}` — estimated 1RM over time for a specific exercise
  - `GET /stats/volume?weeks={n}` — total weekly volume (sets × reps × weight)
  - `GET /stats/consistency?months={n}` — workouts per day (for heatmap)

- [ ] Build chart components:
  - **StrengthProgressChart:** Line chart showing 1RM estimates over time
    - X-axis: Date
    - Y-axis: Estimated 1RM
    - One line per main lift (squat, deadlift, bench)

  - **VolumeChart:** Bar chart of weekly volume
    - X-axis: Week
    - Y-axis: Total volume
    - Shows trend over last 8-12 weeks

  - **ConsistencyHeatmap:** Calendar view
    - Green = workout day, gray = rest day
    - Shows month-by-month workout consistency

- [ ] Add to dashboard:
  - Tab or section switcher (Strength / Volume / Consistency)
  - Time range filters (last 4 weeks, 3 months, all time)

- [ ] Test: Log varied workouts, see charts update in real time

---

### **Week 8: Polish + Deployment**
**Time: 12-15 hours**

**Goal:** Make it production-ready, deploy everywhere, prepare for sharing.

**Tasks:**
- [ ] Frontend polish:
  - [ ] Responsive design (mobile-first, test on phone)
  - [ ] Loading states (spinners while fetching)
  - [ ] Error messages (clear, actionable)
  - [ ] Button states (disabled during submit)
  - [ ] Better styling (colors, spacing, typography)
  - [ ] Dark mode (optional, but nice)

- [ ] Backend polish:
  - [ ] Input validation (all endpoints)
  - [ ] Error responses (consistent format)
  - [ ] Rate limiting (prevent abuse)
  - [ ] Logging (track issues)

- [ ] Deployment:
  - [ ] **Backend:** Deploy to Render or Railway
    - Push to GitHub
    - Connect to Render/Railway
    - Set environment variables (DATABASE_URL, ANTHROPIC_API_KEY, SECRET_KEY)
    - Test live API endpoints
  
  - [ ] **Frontend:** Deploy to Vercel
    - Push to GitHub
    - Connect to Vercel
    - Update API URLs to point to live backend
    - Test live website

- [ ] Testing:
  - [ ] Create new account on live site
  - [ ] Log 5+ workouts
  - [ ] Check streaks working
  - [ ] Unlock a badge
  - [ ] View charts
  - [ ] Test on mobile

- [ ] Documentation:
  - [ ] Write README with:
    - Project overview
    - How to set up locally (clone, install deps, run)
    - How to use the app (screenshots)
    - Tech stack
    - Future features
  - [ ] Add screenshots to GitHub (workout logging, dashboard, charts)

- [ ] Portfolio prep:
  - [ ] Make GitHub repo public (if not already)
  - [ ] Polish GitHub README
  - [ ] Add live demo link in GitHub
  - [ ] Consider a short demo video (optional)

---

## Optional Features (if time allows)

- [ ] Apple Health sync (skip for MVP, add later)
- [ ] Workout programs (pre-made routines like "5/3/1", "PPL")
- [ ] Social sharing (share progress on Twitter/Instagram)
- [ ] Community leaderboards
- [ ] Rest day tracking & recovery recommendations
- [ ] Exercise form tips (from Claude)
- [ ] Workout history export (CSV/PDF)

---

## Skills to Learn (Est. 15 hours)

| Topic | Time | Resource |
|-------|------|----------|
| FastAPI | 3-4 hrs | https://fastapi.tiangolo.com/tutorial/ |
| SQLAlchemy | 2-3 hrs | https://docs.sqlalchemy.org/ |
| JWT Auth | 1-2 hrs | https://fastapi.tiangolo.com/advanced/security/ |
| Claude API | 1-2 hrs | https://docs.anthropic.com/ |
| Recharts | 1-2 hrs | https://recharts.org/en-US/guide |
| React Routing | 1-2 hrs | https://reactrouter.com/en/main |
| Tailwind CSS | 1-2 hrs | https://tailwindcss.com/docs |

**Total:** ~15 hours (spread across 8 weeks, overlap with building)

---

## Getting Started (Right Now)

1. **Create GitHub repo:**
   ```bash
   git init
   git remote add origin https://github.com/YOUR_USERNAME/fitness-app.git
   ```

2. **Clone & setup folders:**
   ```bash
   mkdir backend frontend
   cd backend && python -m venv venv
   source venv/bin/activate  # (Windows: venv\Scripts\activate)
   pip install fastapi uvicorn sqlalchemy psycopg2-binary python-dotenv pydantic
   ```

3. **Follow Pre-Week 1 setup checklist above**

4. **Start Week 1:** Follow FastAPI tutorial, build auth

---

## Weekly Checklist

- [ ] **Pre-Week 1:** Setup tools, GitHub, API key
- [ ] **Week 1-2:** Auth + database working locally
- [ ] **Week 3:** Can log workouts via API (test with Postman)
- [ ] **Week 4:** Claude gives coaching feedback
- [ ] **Week 5:** Can sign up, login, log workout in React
- [ ] **Week 6:** Streaks & badges display on dashboard
- [ ] **Week 7:** Charts show progress
- [ ] **Week 8:** Deployed to live URLs, portfolio-ready

---

## Success Criteria

By end of week 8, you should have:

✅ A live website at a public URL  
✅ User can signup/login with email  
✅ User can log workouts (exercise, sets, reps, weight, RPE)  
✅ Claude API generates personalized coaching feedback  
✅ Streak counter increments and resets correctly  
✅ Achievement badges unlock on milestones  
✅ Charts show strength/volume/consistency progress  
✅ Responsive design (works on phone)  
✅ Clean GitHub repo with good README  
✅ Portfolio-ready (can show to recruiters)  

---

## Resources & Links

**Documentation:**
- FastAPI: https://fastapi.tiangolo.com/
- SQLAlchemy: https://docs.sqlalchemy.org/
- PostgreSQL: https://www.postgresql.org/docs/
- Claude API: https://docs.anthropic.com/
- Recharts: https://recharts.org/
- Tailwind CSS: https://tailwindcss.com/docs/

**Hosting:**
- Backend: https://render.com/ or https://railway.app/
- Frontend: https://vercel.com/

**Tools:**
- API Testing: https://www.postman.com/ or VSCode REST Client
- Database GUI: https://www.pgadmin.org/ or DBeaver

---

**Questions?** Ask along the way. Good luck building! 🚀
