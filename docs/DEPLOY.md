# Deployment Guide

This project has two applications and one database:

```text
backend/   Express API and PostgreSQL database code
frontend/  React/Vite web application
PostgreSQL database
```

## Ports

| Service | Address |
| --- | --- |
| Frontend | http://localhost:5173 |
| Backend API | http://localhost:3000 |
| Swagger docs | http://localhost:3000/api-docs |
| PostgreSQL | port 5432 (Docker network only) |

## Recommended: run locally with Docker

### Requirements

- Git
- Docker Desktop

### Start the application

From the repository root:

```powershell
Copy-Item backend\.env.example backend\.env
Copy-Item frontend\.env.example frontend\.env
docker compose --profile dev up --build
```

Open http://localhost:5173 in your browser.

Docker Compose starts:

1. PostgreSQL
2. The backend API
3. The frontend

The backend applies database migrations before it starts. The database tables
are created automatically; you do not need to create them manually.

Stop the application:

```powershell
docker compose --profile dev down
```

To also delete the local database data:

```powershell
docker compose --profile dev down -v
```

## Run locally without Docker

This option requires PostgreSQL to be installed and running.

### 1. Create the database

Create a database named `employee_app`, or choose another name:

```sql
CREATE DATABASE employee_app;
```

### 2. Configure the backend

Copy `backend\.env.example` to `backend\.env` and use local PostgreSQL values:

```env
NODE_ENV=development
PORT=3000
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=employee_app
DB_USER=postgres
DB_PASSWORD=postgres123
CORS_ORIGIN=*
JWT_SECRET=change-this-in-development
SECRET_WORD=change-this-in-development
```

### 3. Start the backend

```powershell
cd backend
npm ci
npm run migrate:dev
npm run seed:dev
npm run dev
```

`migrate:dev` creates the application tables. `seed:dev` adds sample data.
Seeding is optional.

### 4. Start the frontend

In a second terminal:

```powershell
cd frontend
npm ci
Copy-Item .env.example .env
npm run dev
```

The frontend `.env` must contain:

```env
VITE_API_URL=http://localhost:3000
```

The browser loads the frontend from port `5173` and sends API requests to the
backend on port `3000`. The backend connects to PostgreSQL on port `5432`.

## Database commands

Run these from `backend/`:

```powershell
npm run migrate:dev    # Apply pending migrations
npm run seed:dev       # Add sample development data
npm run reset-db:dev   # Delete and recreate all development tables
```

`reset-db:dev` is destructive. Never run it against production data.

## Default local admin

When the backend starts, it creates this admin account if one does not exist:

```text
Email:    admin@example.com
Password: adminpassword
```

Change this password after logging in.

## Deploy to Render

[render.yaml](../render.yaml) defines three Render resources:

- A backend API service
- A frontend web service
- A managed PostgreSQL database

To deploy:

1. Push the repository to GitHub.
2. In Render, choose **New Blueprint**.
3. Select the repository and apply `render.yaml`.
4. Wait for the database and services to finish deploying.
5. Set the backend `CORS_ORIGIN` to the exact frontend URL.
6. Set the frontend `VITE_API_URL` to the exact backend URL, then redeploy it.

Render provides the backend with `DATABASE_URL`. The backend runs migrations
automatically during deployment. Production seed data is not required.

## Useful backend endpoints

```text
GET    /health
GET    /api-docs
POST   /register
POST   /login
GET    /profile
GET    /users
GET    /users/:id
POST   /users
PUT    /users/:id
DELETE /users/:id
GET    /projects
POST   /projects
GET    /notifications
```

Most endpoints require the JWT token returned by `/login`.

## Troubleshooting

- **API cannot connect to PostgreSQL:** use `DB_HOST=db` with Docker and
  `DB_HOST=127.0.0.1` without Docker.
- **Frontend cannot reach the API:** check `frontend/.env` and confirm
  `VITE_API_URL=http://localhost:3000`.
- **CORS error:** set `CORS_ORIGIN` in `backend/.env` and restart the backend.
- **Migration error:** verify the database exists and the credentials in
  `backend/.env` are correct.
