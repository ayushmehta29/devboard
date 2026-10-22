# DevBoard
### A Full-Stack Project & Task Management Dashboard

<p align="center">
  <strong>React · Go · PostgreSQL · Docker</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Frontend-React-61DAFB?logo=react&logoColor=black" alt="React" />
  <img src="https://img.shields.io/badge/Backend-Go-00ADD8?logo=go&logoColor=white" alt="Go" />
  <img src="https://img.shields.io/badge/Database-PostgreSQL-4169E1?logo=postgresql&logoColor=white" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/Containers-Docker-2496ED?logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/Build-Vite-646CFF?logo=vite&logoColor=white" alt="Vite" />
</p>

**DevBoard** is a full-stack project and task management dashboard built with React, Go, and PostgreSQL. It combines a modern frontend with a real backend API and persistent database, running together in a Docker-based environment.

The project demonstrates how to structure, containerize, run, and maintain a multi-service web application, with additional exploration of optimized Docker builds and container security.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [System Architecture](#-system-architecture)
- [Technology Stack](#-technology-stack)
- [Application Workflow](#-application-workflow)
- [API Reference](#-api-reference)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Docker Deployment](#-docker-deployment)
- [Container Optimization and Security](#-container-optimization-and-security)
- [Testing and Verification](#-testing-and-verification)
- [Learning Outcomes](#-learning-outcomes)
- [Future Improvements](#-future-improvements)

## 🎯 Overview

DevBoard demonstrates a practical full-stack architecture in which the frontend, backend, and database operate as separate services.

Unlike a static dashboard that displays hardcoded information, DevBoard's advanced version retrieves project and task data from a Go API backed by PostgreSQL.

### Project objectives

- Build an interactive project and task management interface.
- Develop a backend REST API using Go.
- Store and retrieve application data using PostgreSQL.
- Connect frontend and backend services through HTTP requests.
- Containerize the application using Docker Compose.
- Simplify development workflows using a Makefile.
- Explore optimized container images and secure build practices.

## ✨ Features

### Project Dashboard
- Display project information through a React-based interface.
- Organize example tasks by project.
- Retrieve application data from the backend API.
- Separate presentation logic from data access.

### Task Management
- Fetch tasks associated with a specific project.
- Retrieve individual task details.
- Explore task and project relationships through API endpoints.

### Search
- Search application data through a dedicated backend endpoint.

### Backend API
- Go-based HTTP API.
- Health-check endpoint for service verification.
- PostgreSQL-backed data access.
- Separate frontend and backend responsibilities.

### Docker-Based Development
- Multi-container application using Docker Compose.
- Dedicated frontend, backend, and database services.
- Repeatable environment setup.
- Convenient lifecycle commands through a Makefile.

> The repository version documented here uses seeded example data. Authentication and AI functionality are not part of this version.

## 🏗️ System Architecture

```mermaid
flowchart TB
    U["User / Browser"]
    F["React + Vite Frontend"]
    A["Go REST API"]
    D[("PostgreSQL 16")]
    C["Docker Compose"]

    U <--> F
    F <-->|"HTTP /api"| A
    A <-->|"SQL Queries"| D

    C -. "Manages services" .-> F
    C -. "Manages services" .-> A
    C -. "Manages services" .-> D

    classDef frontend fill:#DBEAFE,stroke:#2563EB,color:#1E3A8A
    classDef backend fill:#D1FAE5,stroke:#059669,color:#064E3B
    classDef database fill:#EDE9FE,stroke:#7C3AED,color:#4C1D95
    classDef infra fill:#F3F4F6,stroke:#6B7280,color:#111827

    class U,F frontend
    class A backend
    class D database
    class C infra
```

### Architecture components

| Component | Responsibility |
|---|---|
| React + Vite | Renders the dashboard and handles user interactions |
| Go API | Processes requests and provides application data |
| PostgreSQL 16 | Stores project and task data |
| Docker Compose | Coordinates the application services |
| Makefile | Simplifies common development and maintenance commands |

### Service communication

1. The user opens the DevBoard frontend in a browser.
2. The frontend sends HTTP requests to the Go API through the `/api` route.
3. The backend processes requests and queries PostgreSQL when required.
4. PostgreSQL returns the relevant records.
5. The API returns a response to the frontend.
6. The frontend renders the retrieved data.

## 🧰 Technology Stack

| Technology | Purpose |
|---|---|
| React | Component-based user interface |
| Vite | Frontend development and build tooling |
| Go | Backend API development |
| PostgreSQL 16 | Relational database |
| Docker | Application containerization |
| Docker Compose | Multi-service orchestration |
| Make | Development workflow automation |
| Docker Scout | Container image analysis and vulnerability scanning |
| Docker Hardened Images | Exploration of hardened container base images |
| Git & GitHub | Version control and collaboration |

## 🔌 API Reference

The backend exposes the following endpoints.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Check API health |
| GET | `/projects` | Retrieve projects |
| GET | `/tasks?project_id=N` | Retrieve tasks for a project |
| GET | `/tasks/:id` | Retrieve a specific task |
| GET | `/search` | Search application data |

Replace `N` and `:id` with actual identifiers.

### Example requests

Retrieve projects:

```http
GET /projects
```

Retrieve tasks for project 1:

```http
GET /tasks?project_id=1
```

Retrieve an individual task:

```http
GET /tasks/1
```

Check backend health:

```http
GET /health
```

The exact response schema and search query parameters should be verified against the API implementation.

## 📁 Project Structure

```text
devboard/
├── frontend/
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── vite.config.*
│   ├── Dockerfile.multistage
│   ├── Dockerfile.alpine
│   └── Dockerfile.dhi
│
├── backend/
│   ├── main.go
│   ├── go.mod
│   └── ...
│
├── init/
│   └── postgres/
│
├── docker-compose.yml
├── Makefile
├── .env.example
└── README.md
```

*This is a representative structure based on the project's discussed setup. Retain only paths and files that exist in your actual repository.*

## 🚀 Getting Started

### Prerequisites

- Docker Desktop or Docker Engine with Docker Compose.
- Git.
- A terminal such as PowerShell, Windows Terminal, or a Linux shell.

**Docker is the primary prerequisite:** the standard workflow runs the frontend, backend, and PostgreSQL as containers without requiring a separate local Go or PostgreSQL installation.

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd devboard
```

Replace the placeholder with your actual GitHub repository URL.

### 2. Configure environment variables

Create your local environment file using the provided template.

PowerShell:

```powershell
Copy-Item .env.example .env
```

Linux or macOS:

```bash
cp .env.example .env
```

The discussed configuration uses the following values:

```dotenv
POSTGRES_USER=devboard
POSTGRES_PASSWORD=devboard
POSTGRES_DB=devboard
POSTGRES_URL=postgres://devboard:devboard@postgres:5432/devboard?sslmode=disable
PORT=8080
```

These are development examples, not production credentials. Use the actual variables expected by your Compose configuration and backend.

### 3. Start the application

Build the images and start all services:

```bash
docker compose up --build
```

To run the services in the background:

```bash
docker compose up --build -d
```

### 4. Access the application

With the discussed port mappings:

| Service | Local address |
|---|---|
| Frontend | `http://localhost:8080` |
| Go API | `http://localhost:8081` |
| PostgreSQL | `localhost:5432` if the database port is published |

The frontend uses its configured `/api` proxy to communicate with the backend. Verify the actual Compose port mappings before relying on these addresses.

## 🐳 Docker Deployment

Docker Compose coordinates the services and their configuration.

### Start services

```bash
docker compose up --build
```

### Run in the background

```bash
docker compose up --build -d
```

### View running containers

```bash
docker compose ps
```

### Follow service logs

```bash
docker compose logs -f
```

### Stop services

```bash
docker compose down
```

### Reset the development database

If you intentionally want to remove the database's persisted Compose volume and start fresh, identify the correct volume first. The following command can permanently delete local database data:

```bash
docker compose down -v
```

Use this only when you are comfortable losing the persisted development database contents.

## ⚙️ Development Workflow

DevBoard includes a Makefile to simplify recurring tasks.

| Command | Purpose |
|---|---|
| `make setup` | Prepare the development environment |
| `make up` | Start application services |
| `make down` | Stop application services |
| `make logs` | Inspect application logs |
| `make reset` | Reset the development environment |
| `make smoke` | Run the configured smoke checks |

Run these commands from the project root. The exact behavior depends on the targets defined in your Makefile.

## 🔒 Container Optimization and Security

The project also explores frontend containerization approaches beyond a basic Docker build.

### Multi-stage builds

A multi-stage Dockerfile separates the build environment from the runtime environment.

Typical workflow:

1. Use Node.js to install dependencies and build the Vite frontend.
2. Generate the production assets in `dist/`.
3. Copy only the required output into the runtime image.
4. Avoid shipping unnecessary development dependencies in the final image.

### Alpine-based runtime

Alpine-based images can reduce image size by using a lightweight Linux distribution. Compatibility and security still depend on the selected image, dependencies, and configuration.

### Docker Hardened Images

The project includes an exploration of a Docker Hardened Image-based frontend build. The runtime configuration and exposed port depend on the selected image and its entrypoint.

### Docker Scout

Docker Scout can help analyze images for known vulnerabilities and review image composition.

Example commands:

```bash
docker scout quickview devboard-fe:full
docker scout cves devboard-fe:full
```

Use the image tag that you have actually built. Scan results should be reviewed and addressed rather than assumed to be clean.

## 🧪 Testing and Verification

Use this checklist to verify the application after starting the services.

- [ ] Frontend loads successfully.
- [ ] Backend health endpoint responds.
- [ ] Project data is retrieved through the API.
- [ ] Tasks can be filtered by project ID.
- [ ] Individual task endpoint responds correctly.
- [ ] Search endpoint behaves as expected.
- [ ] Backend connects successfully to PostgreSQL.
- [ ] Data remains available after an ordinary container restart.
- [ ] Docker Compose starts the required services.
- [ ] Frontend production build completes successfully.
- [ ] Docker image scans have been reviewed.

Only mark items as complete after testing them.

## 🧠 Learning Outcomes

DevBoard provides practical experience in:

- Full-stack application architecture.
- React frontend development.
- REST API development using Go.
- Relational database integration using PostgreSQL.
- Inter-service communication through HTTP.
- Containerization with Docker.
- Multi-service orchestration with Docker Compose.
- Environment configuration and service networking.
- Multi-stage Docker builds.
- Container image optimization and vulnerability analysis.
- Development workflow automation with Make.
- Troubleshooting application and container issues.

## 🔮 Future Enhancements

- [ ] Add user authentication and authorization.
- [ ] Implement project and task creation, editing, and deletion.
- [ ] Add automated unit and integration tests.
- [ ] Introduce a CI/CD pipeline with GitHub Actions.
- [ ] Add database migration management.
- [ ] Improve structured logging and monitoring.
- [ ] Add production deployment configuration.
- [ ] Automate security checks for container images.
- [ ] Add API documentation and request validation.

## 👨‍💻 Author

**Ayush Mehta**

B.Tech — Computer Science and Engineering

GitHub: [@ayushmehta29](https://github.com/ayushmehta29)

---

<p align="center">
  <strong>DevBoard</strong><br />
  <em>Building full-stack applications with Go, React, PostgreSQL, and Docker.</em>
</p>
