# Workouts Repositories

## workouts-build (this repo)

The top-level build and orchestration repository. Contains the frontend and backend as subdirectories and will eventually hold Docker Compose configuration to build and run them together. This is the integration point — it does not contain application logic itself.

**Structure:**
- `workouts-frontend/` — React web application
- `workouts-backend/` — Go API server
- `.claude/` — project documentation and Claude Code settings

## workouts-backend

Go REST API server using only the standard library (`net/http`, `encoding/json`). No external dependencies.

**Tech:** Go 1.23+, stdlib only

**Structure:**
- `main.go` — entrypoint, registers all route groups on `http.ServeMux`
- `handler/` — one file per domain (health, auth, user, weight, workout)
- `go.mod` — module `github.com/workouts/workouts-backend`
- `.claude/api.md` — full API endpoint documentation

**Run:** `go run main.go` (default port 8080, configurable via `PORT` env var)

**Endpoints:** 15 total across 5 domains — health, auth, user, weight, workouts. Currently returning stub/mock responses. See `workouts-backend/.claude/api.md` for the full contract.

## workouts-frontend

React single-page application for the workout tracker web UI.

**Tech:** React 19, Vite 6, TypeScript 5.7

**Structure:**
- `src/App.tsx` — root component
- `src/main.tsx` — React DOM entry point
- `index.html` — Vite HTML entry
- `vite.config.ts` — Vite config (host bound to 127.0.0.1 for Windows compatibility)

**Run:** `npm run dev` (localhost:5173)

**Status:** Scaffolded with minimal UI. Will be built out to consume the backend API.
