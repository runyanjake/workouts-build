# Workouts-Build
The build and development tools for my workout tracking application.

## Related Repositories
[Frontend UI Application](https://github.com/runyanjake/workouts-frontend)
[Backend Web Server](https://github.com/runyanjake/workouts-backend)

## Configuration

### Child Repositories
The two child repositories should be cloned into this folder. They are ignored in the `.gitignore` file.
This repository will build them and deploy them either locally for development or in a docker environment for production.

### Claude 
The `.claude` directory contains instructions for claude to work on both of the child repositories.

## Deployment 

### Local, By Parts
#### 1. Services Start
Start Postgres and Redis using the dockerfile in the main repos.
```bash
docker compose down
docker volume rm workouts-build_postgres_data workouts-build_postgres_data
docker compose up -d
```
#### 2. Backend Start
Start the backend which will configure Postgres tables.
```bash
go run main.go
```
If you need to use test data, load it using the utility under `scripts/`:
```bash
go run script/seed-test-data/
```
#### 3. Frontend Start
Start the frontend dev server through Node:
```bash
npm run dev
```

### Local (Windows), All-In-One **(WARNING: Clears Data)**
```bash
docker compose -f docker-compose.dev.yml down ; docker volume rm workouts_postgres_data workouts_redis_data ; docker system prune ; docker compose -f docker-compose.dev.yml up -d
```

### Production (Unix), All-In-One

- Rebuild
```bash
docker compose down && docker system prune && docker compose -f docker-compose.prod.yml up -d --build
```

- Full Clean **(WARNING: Clears Data)**
```bash
docker compose -f docker-compose.prod.yml down && sudo rm -r /pwspool/software/workouts && docker system prune && docker compose -f docker-compose.prod.yml up -d --build
```
