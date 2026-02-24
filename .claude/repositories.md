# Workouts Repositories

## workouts-build (this repo)

The top-level build and orchestration repository. Contains the frontend and backend as subdirectories and holds Docker Compose configuration for local development.

**Structure:**
- `workouts-frontend/` — React web application
- `workouts-backend/` — Go API server
- `docker-compose.yml` — Postgres 17 + Redis 7 for local dev
- `db/init.sql` — Database schema initialization
- `.env` — Local dev credentials (gitignored)
- `.claude/` — project documentation and Claude Code settings

## workouts-backend

Go REST API server. Uses stdlib `net/http` for routing, `lib/pq` for PostgreSQL, `go-redis` for Redis, and `golang-jwt` for JWTs.

**Tech:** Go 1.24+, stdlib + lib/pq + go-redis/v9 + golang-jwt/jwt/v5 + golang.org/x/crypto

**Structure:**
- `main.go` — entrypoint: connects to DB + Redis, creates services, registers all route groups, wraps mux with auth middleware
- `model/` — domain structs with JSON tags
  - `user.go` — `User`
  - `weight.go` — `WeightEntry`
  - `exercise.go` — `Exercise` (name, type, muscle_groups, description, notes)
  - `workout.go` — `Workout` (name, created_at, sets []Set)
  - `set.go` — `Set` (exercise_id, date, reps *int, weight *float64, time *int, notes)
- `repository/` — data access layer, one struct per domain
  - `user.go` — `UserRepo`
  - `weight.go` — `WeightRepo`
  - `exercise.go` — `ExerciseRepo` (uses pq.Array for muscle_groups)
  - `workout.go` — `WorkoutRepo`
  - `set.go` — `SetRepo`
  - `token.go` — `TokenRepo` (Redis: store/check/delete access + refresh tokens)
- `handler/` — HTTP handlers using closures to capture repos/services
  - `auth.go` — login, register, refresh, logout
  - `user.go` — get, update, delete user
  - `weight.go` — list and log weight entries
  - `exercise.go` — CRUD exercises
  - `workout.go` — CRUD workouts (GET by ID embeds sets)
  - `set.go` — create/update/delete sets nested under `/workout/{workoutID}/set`
  - `middleware.go` — `RequireAuth(authSvc)` wraps entire mux
  - `auth_context.go` — `currentUserID(r)` / `withUserID(ctx, id)` context helpers
  - `health.go` — healthcheck
  - `response.go` — `writeJSON`, `writeError` helpers
- `service/auth.go` — `AuthService`: JWT generation/validation, token lifecycle
- `db/db.go` — PostgreSQL connection pool setup
- `db/redis.go` — Redis client setup
- `cmd/seed/main.go` — seed script: inserts demo users, exercises, workouts, and sets
- `go.mod` — module `github.com/workouts/workouts-backend`
- `.claude/api.md` — full API endpoint documentation

**Run:** `go run main.go` (default port 8080, configurable via `PORT` env var)

**Endpoints:** 23 total across 6 domains — health, auth, user, weight, exercises, workouts, sets. See `workouts-backend/.claude/api.md` for the full contract.

## workouts-frontend

React single-page application for the workout tracker web UI.

**Tech:** React 19, Vite 6, TypeScript 5.7, Tailwind CSS v4, React Router v6, Recharts

**Structure:**
- `src/main.tsx` — entry: BrowserRouter + AuthProvider
- `src/App.tsx` — route definitions
- `src/index.css` — `@import "tailwindcss"` + minimal reset
- `src/types/api.ts` — TypeScript interfaces for all API shapes
- `src/types/auth.ts` — `AuthStatus`, `AuthState` types
- `src/lib/api.ts` — typed fetch wrapper with auto token refresh on 401; exports `authApi`, `userApi`, `weightApi`, `exerciseApi`, `workoutApi`, `setApi`
- `src/lib/storage.ts` — localStorage helpers (access/refresh token + expiry)
- `src/context/AuthContext.tsx` — `AuthProvider` + `useAuth` hook
- `src/pages/` — `LandingPage`, `LoginPage`, `RegisterPage`, `DashboardPage`, `NotFoundPage`
- `src/components/layout/` — `Header`, `Footer`, `ProtectedRoute`
- `src/components/auth/` — `LoginForm`, `RegisterForm`
- `src/components/dashboard/` — `WorkoutEditor`, `WorkoutList`, `WorkoutCard`, `WeightLogger`
- `src/components/analytics/` — `AnalyticsPanel`, `WeightChart`, `WorkoutFrequencyChart`, `WorkoutTypeChart`
- `vite.config.ts` — Tailwind plugin + dev proxy (`/auth`, `/api`, `/user`, `/workout`, `/exercise`, `/health` → `localhost:8080`)

**Run:** `npm run dev` (localhost:5173)

**Routes:**
- `/` — LandingPage (public)
- `/login` — LoginPage (public; → /dashboard if authed)
- `/register` — RegisterPage (public; → /dashboard if authed)
- `/dashboard?tab=workouts` — workout editor + list (protected, default tab)
- `/dashboard?tab=analytics` — weight + frequency + top workouts charts (protected)
