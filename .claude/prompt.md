I am creating a new project that must include the following Claude Code config files:
- claude_config.yml
- context.md
- instructions.md
- tone.md

Below is a description of the application, which you should use to generate those files.
Important names are between double asterisks.

-----

The agent will act as a competent engineer, who has domain knowledge in **Ktor** server backend development, **Vite + React** frontend development, and **DevOPS** experience when it comes to deploying in a Traefik/Docker environment.

We will be building a complete workout application, including the frontend, backend and build components.
The agent will work in the top level repo, "workouts-build/". Inside this repo it will have access to the 2 cloned repos "workouts-frontend" and "workouts-backend".

The backend will be built using the **Ktor** language which is Java/Kotlin-adjacent.
The frontend will be built using the Vite + React framework.
Each repo will contain its own `Dockerfile` and `docker-compose.yml` files for build. This repo will use these files to build and deploy the services.

The agent must ensure that the frontend and backend communicate over interfaces that are common between the two projects.

# Terminology
This application tracks **Workouts**, which is how we refer to a workout session.
Workouts contain multiple **Sets** of different **Exercises**.

# Build Repo
This repo contains scripts for **WSL on Windows**, **Ubuntu**, and **MacOS** that build and start the containers.

# Frontend Repo
The Frontend is built using Vite + React. It makes authenticated api calls to the backend to retrieve the data for a logged in user.

The frontend contains the following views:
- Login: Where users may authenticate.
- Signup: Where users may create an account.
- Tracker: Where authenticated users may view their workouts.
- Analytics: Where users may view graphs that track their workout data through time.
- Exercies: Where users may view info about an exercise or browse a gallery of listings.
- Settings: Where users may configure website customization, or provide API keys/perform OAuth2.0 login to integrate with external services for some features.

# Backend Repo
The backend is a server built using **Ktor** and exposes a number of endpoints that the frontend app uses.
Storage will be done by connecting to a **Postgres** container, using the appropriate methods for local testing and production. 

Data will be scoped per user, and will contain the following table information.

Here are some high level api overviews:
- /auth/: Authorization of the user within the system to grant a short lived token to be used with apis. Supports periodic refresh of the token as best practices would dictate.
- /user/: Used for user authentication. Employs the typical auth/deauth/reauth actions you might expect.
- /user/bio: CRUD of biological data about a user that they may track. For now, we'll implement just /user/weight/.
- /workout: CRUD of workout related data. This will include registration and management of new workout types, recording of sets.

Here are some databse definitions:
1. User
- user_id (uuid)
- whatever else is necessary
2. Exercise
- exercise_id (uuid)
- name
- measurement (weight/rep/time)
- interface (dumbbell/barbell/cable/bodyweight/plate/kettlebell)
- muscle_group (arms/back/chest/shoulders/legs/core)
- muscle_subgroup (bicep/tricep/lats/lower_back/quads/calves/upper_core/lower_core/obliques)
- description
- form notes (array of string)
3. Sets
- set_id (uuid)
- user_id
- exercise_id
- date
- reps (nullable)
- weight (nullable)
- time (nullable)
- Notes

The enums that are created for these database tables (e.g. muscle_group/subgroup) should encode the db name in the table and the human readable name in their entries.
