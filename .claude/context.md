# Project Context

## Overview
This is a full-stack workout tracking application split across three repositories, all orchestrated from this top-level **workouts-build** repo.

## Repository Structure
- **workouts-build/** (this repo) — Build scripts and Docker/Traefik deployment configuration
- **workouts-frontend/** — Vite + React frontend application
- **workouts-backend/** — Ktor (Kotlin) backend server with PostgreSQL storage

## Domain Model

### Terminology
- **Workout**: A workout session performed by a user.
- **Set**: A single set of an exercise performed during a workout, recording reps, weight, or time.
- **Exercise**: A defined exercise type with metadata about equipment and muscles targeted.

### Database Tables

#### User
- `user_id` (UUID, primary key)
- Additional fields as needed for auth and profile

#### Exercise
- `exercise_id` (UUID, primary key)
- `name` (string)
- `measurement` (enum: weight, rep, time)
- `interface` (enum: dumbbell, barbell, cable, bodyweight, plate, kettlebell)
- `muscle_group` (enum: arms, back, chest, shoulders, legs, core)
- `muscle_subgroup` (enum: bicep, tricep, lats, lower_back, quads, calves, upper_core, lower_core, obliques)
- `description` (string)
- `form_notes` (array of strings)

#### Sets
- `set_id` (UUID, primary key)
- `user_id` (UUID, foreign key)
- `exercise_id` (UUID, foreign key)
- `date` (date)
- `reps` (integer, nullable)
- `weight` (numeric, nullable)
- `time` (duration, nullable)
- `notes` (string)

### Enum Convention
Enums encode the **database column name** in the table definition and the **human-readable display name** in their entries.

## API Surface

### Backend Endpoints
- `/auth/` — Token-based authorization with short-lived tokens and periodic refresh
- `/user/` — User registration, authentication, deauthentication
- `/user/bio` — Biological data tracking (starting with `/user/weight/`)
- `/workout` — CRUD for workouts, exercise registration, and set recording

## Frontend Views
- **Login** — User authentication
- **Signup** — Account creation
- **Tracker** — View and manage workouts
- **Analytics** — Graphs tracking workout data over time
- **Exercises** — Browse exercise gallery and view exercise details
- **Settings** — Site customization, API keys, OAuth2.0 integrations

## Cross-Repo Contract
The frontend and backend must communicate over shared interfaces. Any API changes in the backend must be reflected in the frontend's API client, and vice versa.
