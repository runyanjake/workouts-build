# Instructions

## General Principles
- Act as a competent engineer with domain knowledge in Ktor backend development, Vite + React frontend development, and DevOps (Docker/Traefik).
- Always ensure frontend and backend stay in sync on shared interfaces (API contracts, request/response types).
- Prefer simple, working solutions over over-engineered abstractions.

## Repository Boundaries
- This repo (workouts-build) owns build scripts and deployment configuration only.
- Frontend code changes belong in `workouts-frontend/`.
- Backend code changes belong in `workouts-backend/`.
- Each sub-repo contains its own `Dockerfile` and `docker-compose.yml`.

## Backend (Ktor / Kotlin)
- Use Ktor idioms and conventions for routing, serialization, and authentication.
- Connect to PostgreSQL for all persistent storage.
- Use UUIDs for all primary keys.
- Enums should encode the DB column name in the table and the human-readable name in entries.
- Implement token-based auth with short-lived access tokens and refresh support.
- Scope all data queries to the authenticated user.

## Frontend (Vite + React)
- Use React functional components with hooks.
- All API calls must be authenticated (attach token from auth flow).
- Keep views focused: Login, Signup, Tracker, Analytics, Exercises, Settings.

## Build & Deployment
- Build scripts must support WSL on Windows, Ubuntu, and MacOS.
- Use Docker Compose to orchestrate containers (frontend, backend, Postgres).
- Use Traefik as the reverse proxy for routing and TLS.
- Distinguish between local development and production configurations.

## Code Quality
- Write clear, maintainable code. Favor readability over cleverness.
- Validate inputs at system boundaries (API endpoints, user-facing forms).
- Do not introduce security vulnerabilities (SQL injection, XSS, token leakage).
