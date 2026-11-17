# CampusHub – Dockerized 9-Service Application

A containerized event management application built using Docker, Docker Compose, Nginx, PostgreSQL, Redis, RabbitMQ, and MinIO.

## 1. Project Overview

CampusHub is a multi-service application that allows users to browse events, register for events, and upload event images.

The application is containerized using Docker Compose with separate services for the frontend, backend API, background worker, database, caching, message queue, object storage, email testing, and reverse proxy.

## 2. Architecture

The application consists of nine services:

| Service | Technology | Purpose |
|---|---|---|
| Gateway | Nginx Unprivileged | Reverse proxy and single public entry point |
| Frontend | React, Vite, Nginx | Serves the frontend |
| API | Node.js | Handles application requests |
| Worker | Node.js | Processes background email jobs |
| Database | PostgreSQL 16 | Persistent application data |
| Cache | Redis 7 | API response caching |
| Message Broker | RabbitMQ | Handles asynchronous jobs |
| Object Storage | MinIO | Stores uploaded event images |
| Email Testing | MailHog | Captures outgoing emails |

### Request Flow

- `/` → Frontend
- `/api/` → Backend API
- `/storage/` → MinIO object storage

The gateway acts as the only publicly exposed application entry point.

## 3. Prerequisites

- Docker Desktop
- Docker Compose V2
- Git

## 4. Environment Setup

Clone the repository:

```bash
git clone <repository-url>
cd MSE-2-mock-SSED
```

Create the environment file:

```bash
cp .env.example .env
```

Update the passwords and credentials in `.env` before starting the application.

**Important:** Never commit `.env` or expose actual credentials in the repository.

## 5. Build and Run

Build the Docker images:

```bash
docker compose build
```

Start all services:

```bash
docker compose up -d --wait
```

Check the running containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

Stop the application:

```bash
docker compose down
```

Named volumes preserve PostgreSQL and MinIO data across container restarts. Use `docker compose down -v` only when you intentionally want to delete persistent data.

## 6. Access the Application

| Component | Address |
|---|---|
| CampusHub Application | http://localhost:8080 |
| API Health Check | http://localhost:8080/api/health |
| Events API | http://localhost:8080/api/events |
| MailHog UI | http://localhost:8025 |
| RabbitMQ Management | http://localhost:15672 |

MailHog and RabbitMQ management interfaces are bound to localhost only.

## 7. Docker Implementation Details

### Frontend

- Uses a multi-stage Docker build.
- Node.js is used only during the build stage.
- Production static files are served through Nginx Unprivileged.
- Runs as a non-root user.
- Includes a container health check.

### API and Worker

- Share the same Docker image to avoid duplicate builds.
- Install production dependencies only.
- Run as a non-root Node.js user.
- API includes a health check.
- Worker overrides the default API command and runs `node src/worker.js`.
- Worker disables the inherited API health check.

### Docker Compose

- Uses nine independent services.
- Implements separate frontend, backend, and data networks.
- Uses named volumes for PostgreSQL and MinIO persistence.
- Uses health checks and service dependency conditions.
- Configures restart policies and CPU/memory resource limits.
- Publishes only the gateway on the public host interface.

### Gateway

Nginx routes requests to the appropriate internal service.

It also supports image uploads through the `/storage/` route and configures a 5 MB request body limit.

## 8. Important Design Decisions

1. **Multi-stage frontend build:** Separates build dependencies from the production runtime, reducing the final image size.

2. **Shared API and worker image:** Both services use the same image, reducing duplication and ensuring consistent application dependencies.

3. **Network isolation:** Separate Docker networks restrict unnecessary communication between services and improve application organization.

4. **Named volumes:** Preserve database and object storage data even when containers are recreated.

5. **Non-root containers:** Reduce security risks by avoiding unnecessary root privileges.

6. **Health checks:** Help Docker Compose determine when dependent services are ready.

7. **Environment-based configuration:** Keeps sensitive credentials outside the committed Compose configuration.

## 9. Verification

The project was tested using the provided `grade.sh` script.

Verification includes:

- All nine containers running.
- Application health endpoint accessible through the gateway.
- Frontend and seeded events accessible.
- Redis cache HIT verification.
- Event registration returning HTTP 202.
- RabbitMQ queue creation.
- Background worker email delivery to MailHog.
- Image upload and retrieval through MinIO.
- PostgreSQL data persistence after container recreation.
- Non-root execution for API, worker, frontend, and gateway.
- Only one publicly exposed host port.
- API image size below 250 MB.
- No hardcoded secrets in the Compose configuration.

**Result: All 18 automated checks passed.**

## 10. Repository Structure

```text
MSE-2-mock-SSED/
├── app/
│   ├── api/
│   │   ├── Dockerfile
│   │   └── .dockerignore
│   ├── frontend/
│   │   ├── Dockerfile
│   │   └── .dockerignore
│   └── nginx.conf
├── .env.example
├── .gitignore
├── docker-compose.yml
├── grade.sh
└── README.md
```

---

**Project:** CampusHub – Dockerized 9-Service Application