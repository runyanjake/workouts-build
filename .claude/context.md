# Project Context

## What is this?

A workout tracking application. Users can register, log in, track their workouts, and record their weight over time.

## Architecture

- **Backend:** Go REST API (stdlib only, no frameworks) serving JSON on port 8080
- **Frontend:** React SPA (Vite + TypeScript) on port 5173
- **iOS app:** Planned for the future (Swift/Xcode), will consume the same REST API
- **Build:** This repo (`workouts-build`) will use Docker to compose and deploy the frontend and backend together

## Current State

The project is in early development:

1. **API contract is defined** — 15 endpoints across 5 domains (health, auth, user, weight, workouts), all returning hardcoded stub responses. The full specification is in `workouts-backend/.claude/api.md`.
2. **Frontend is scaffolded** — minimal React + Vite + TypeScript app, not yet wired to the backend.
3. **No database yet** — stubs will be replaced with real persistence (likely PostgreSQL) in a future phase.
4. **No real auth yet** — login/register/refresh return fake tokens. JWT signing and validation will be added later.
5. **No Docker setup yet** — the build repo will eventually contain Docker Compose to run everything together.

## Tech Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Backend language | Go | Industry standard for performant APIs, simple concurrency, small binaries |
| Backend framework | stdlib (`net/http`) | Go 1.22+ has method-based routing built in, no need for third-party routers |
| Frontend framework | React + Vite | Industry standard, large ecosystem, TypeScript support |
| API style | REST + JSON | Simple, well-understood, easy to generate iOS clients from OpenAPI spec later |

## What's Next

- Wire up real JWT authentication
- Add PostgreSQL database and replace stubs with real data access
- Build out the React frontend to consume the API
- Docker Compose setup for local development and deployment
- iOS app (Swift) consuming the same API
