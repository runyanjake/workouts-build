# Project Context

## What is this?

A workout tracking application. Users can register, log in, track their workouts, and record their weight over time.

## Architecture

- **Backend:** Go REST API (stdlib + lib/pq + go-redis + jwt) serving JSON on port 8080
- **Frontend:** React SPA (Vite + TypeScript) on port 5173
- **Database:** PostgreSQL 17, run via Docker Compose for local dev
- **Session store:** Redis 7, run via Docker Compose for local dev
- **iOS app:** Planned for the future (Swift/Xcode), will consume the same REST API
- **Build:** This repo (`workouts-build`) orchestrates Docker Compose for local dev and will handle production deployment

## Backend Layering

```
HTTP Request
    ↓
handler/middleware.go  — Auth middleware: validates Bearer token, sets userID in context
    ↓
handler/              — HTTP layer: parse request, call service/repo, write JSON response
    ↓
service/auth.go       — Auth business logic: JWT generation/validation, token lifecycle
    ↓
repository/           — Data access: SQL (Postgres) and Redis queries, results into model structs
    ↓
model/                — Domain structs shared between layers
    ↓
PostgreSQL + Redis
```

- **model/** — Plain Go structs (`User`, `WeightEntry`, `Exercise`, `Workout`, `Set`) with JSON tags. Sensitive fields use `json:"-"`.
- **repository/** — One struct per domain. `UserRepo`, `WeightRepo`, `ExerciseRepo`, `WorkoutRepo`, `SetRepo` hold `*sql.DB`. `TokenRepo` holds `*redis.Client`. All methods accept `context.Context`.
- **service/auth.go** — `AuthService` owns JWT generation/validation and orchestrates `TokenRepo`. The only current service; others will be added when business logic warrants it.
- **handler/** — HTTP handlers using closures to capture repo/service instances. Falls back to stubs when `db == nil`.
- **handler/middleware.go** — `RequireAuth(authSvc)` wraps the entire mux. Public paths (`/health`, `/auth/login`, `/auth/register`) skip validation.
- **handler/auth_context.go** — `currentUserID(r)` reads from request context (set by middleware). `withUserID(ctx, id)` is called by middleware.

## Current State

1. **Full auth flow is implemented** — bcrypt password hashing, JWT access tokens (15min) + refresh tokens (7d) stored in Redis. Login, register (returns tokens), refresh (with rotation), and logout (token revocation) all work.
2. **Auth middleware protects all routes** except `/health`, `/auth/login`, `/auth/register`.
3. **Model + repository layer is built** — Full schema: `users`, `weight_entries`, `exercises`, `workouts`, `sets`. Handlers read/write real data from Postgres when connected, falling back to stubs when not.
4. **Full React frontend is implemented** — Auth, dashboard (workout editor + weight logger), analytics (weight, frequency, top workouts charts), protected routes, auto token refresh.

## Auth Flow

- **Access token:** JWT, HS256, 15 min TTL, stored in Redis as `access:<userID>:<tokenID>`
- **Refresh token:** JWT, HS256, 7 day TTL, stored in Redis as `refresh:<userID>:<tokenID>`, rotated on each use
- **Logout:** deletes all `access:<userID>:*` and `refresh:<userID>:*` keys from Redis
- **Middleware:** validates JWT signature + checks Redis (revocation). Returns 401 if token is missing, malformed, expired, or revoked.
- **Deferred:** rate limiting on login, password reset, email verification, CSRF, refresh token family reuse detection

## Infrastructure

- **Local dev:** `docker compose up -d` starts Postgres (port 5432) and Redis (port 6379)
- **Schema:** `db/init.sql` — `users`, `weight_entries`, `exercises`, `workouts`, `sets` tables
- **Env vars:** `DATABASE_URL`, `REDIS_URL`, `JWT_SECRET`, `PORT` (all have sane defaults for local dev)
- **Credentials:** `.env` at the build repo root (gitignored)

## Tech Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Backend language | Go | Industry standard for performant APIs, simple concurrency, small binaries |
| Backend framework | stdlib (`net/http`) | Go 1.22+ has method-based routing built in, no need for third-party routers |
| Auth tokens | JWT (golang-jwt/jwt v5) | Industry standard, stateless, self-contained claims |
| Token store | Redis (go-redis v9) | Fast, TTL-native, enables revocation without DB queries |
| Password hashing | bcrypt (golang.org/x/crypto) | Official Go crypto, adaptive cost, industry standard |
| Backend layering | handler → service (auth only) → repository | Service layer added where business logic demands it |
| Database | PostgreSQL 17 | Industry standard relational DB |
| DB driver | `lib/pq` | Standard PostgreSQL driver for Go's `database/sql` |
| Local infra | Docker Compose | Matches production setup, simple to start/stop |
| Frontend framework | React + Vite | Industry standard, large ecosystem, TypeScript support |
| API style | REST + JSON | Simple, well-understood, easy to generate iOS clients from OpenAPI spec later |

## What's Next

- Add testcontainers-go for integration tests
- iOS app (Swift) consuming the same API
