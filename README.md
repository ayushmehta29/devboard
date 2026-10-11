# DevBoard --- Full-Stack Task Management Application

DevBoard is a full-stack project management dashboard built to
demonstrate how a React frontend, Go API, and PostgreSQL database work
together in a containerized environment.

Unlike a frontend-only mockup, this version retrieves and persists
project and task data through a backend API and database.

## Architecture

``` text
Browser
   |
   v
React + Vite frontend (:8080 on host)
   |  /api requests
   v
Go backend API (:8081 on host, :8080 in container)
   |
   v
PostgreSQL (:5432)
```

The frontend forwards requests beginning with `/api` to the backend. The
Go API reads and writes data in PostgreSQL. SQL initialization scripts
load example schema and data when the database volume is initialized for
the first time.

**Project scope:** This version focuses on the frontend/backend/database
integration. It intentionally has no login system or AI functionality.

## Features

-   React frontend built with Vite
-   Go backend API
-   PostgreSQL database with initialization scripts and example data
-   API proxying from the frontend to the backend
-   Dockerfiles for building and running the frontend and backend
-   Docker Compose orchestration for the complete stack
-   Environment-based configuration using `.env`
-   Optional Makefile shortcuts for common development tasks
-   GitHub Actions CI/CD workflows configured for security scanning
    (SonarQube/SAST and OWASP ZAP/DAST, depending on workflow
    configuration)
-   Docker Scout vulnerability scanning experiments on alternative
    frontend images

## Technology Stack

  Component                            Technology
  ------------------------------------ -------------------------------------------
  Frontend                             React, Vite, Node.js
  Backend                              Go
  Database                             PostgreSQL 16 Alpine
  Containerization                     Docker, Docker Compose
  Frontend static serving              `serve` or Nginx, depending on Dockerfile
  Image security analysis              Docker Scout
  CI/CD security tooling               GitHub Actions, SonarQube, OWASP ZAP
  Optional local AI tooling practice   Docker Model Runner

## Prerequisites

You only need Docker Desktop with Docker Compose to run the full stack.
Node.js, Go, and PostgreSQL do not need to be installed on the host for
the containerized setup.

Optional tools: - Git - Docker Scout CLI, for image security scans -
`make`, if you want to use the Makefile shortcuts

## Quick Start --- Docker Compose

Run these commands from the repository root (the directory containing
`docker-compose.yml`).

### 1. Create your environment file

PowerShell:

``` powershell
Copy-Item .env.example .env
```

Linux/macOS:

``` bash
cp .env.example .env
```

Review `.env` and use appropriate local credentials. Do not commit the
real `.env` file or production secrets.

### 2. Build and start the stack

``` powershell
docker compose up --build
```

Wait for the services to start, then open:

-   Frontend: <http://localhost:8080>
-   Backend health check: <http://localhost:8081/health>
-   PostgreSQL: `localhost:5432`

The frontend is the main application entry point. Requests under `/api`
are forwarded to the backend.

### 3. Stop the stack

``` powershell
docker compose down
```

To remove the Compose-managed database volume as well, only if you
intentionally want to delete its persisted data:

``` powershell
docker compose down -v
```

**Warning:** `docker compose down -v` deletes the Compose-managed
database volume and its stored data.

## Manual Docker Setup

The manual workflow shows what Compose automates: creating a network,
building images, configuring services, and starting containers.

> Run the following from the repository root. These commands assume the
> paths and ports described here match the checked-out project.

### 1. Create a network

``` powershell
docker network create devboard-net
```

### 2. Build the frontend and backend

``` powershell
docker build -t devboard-frontend ./frontend
docker build -t devboard-backend ./backend
```

### 3. Start PostgreSQL

PowerShell:

``` powershell
docker run -d --name postgres --network devboard-net -e POSTGRES_USER=devboard -e POSTGRES_PASSWORD=devboard -e POSTGRES_DB=devboard -v "${PWD}/init/postgres:/docker-entrypoint-initdb.d:ro" -p 5432:5432 postgres:16-alpine
```

The official PostgreSQL image runs the SQL initialization scripts when
initializing a fresh database directory. Those scripts do not
automatically rerun against an already initialized database directory.

### 4. Start the Go backend

``` powershell
docker run -d --name backend --network devboard-net -e PORT=8080 -e POSTGRES_URL="postgres://devboard:devboard@postgres:5432/devboard?sslmode=disable" -p 8081:8080 devboard-backend
```

### 5. Start the frontend

``` powershell
docker run -d --name frontend --network devboard-net -p 8080:4173 devboard-frontend
```

### 6. Test the application

Open <http://localhost:8080>.

PowerShell checks:

``` powershell
curl.exe http://localhost:8081/health
curl.exe "http://localhost:8080/api/tasks?project_id=1"
```

The second request checks the path through the frontend proxy to the
backend and database.

### 7. Clean up the manual setup

``` powershell
docker rm -f frontend backend postgres
docker network rm devboard-net
```

If you used Docker Compose instead, use `docker compose down` rather
than these manual cleanup commands.

## API Reference

The browser accesses the API through `/api`; the backend handles the
corresponding paths without that prefix.

  Method    Path                        Description
  --------- --------------------------- -----------------------------------
  `GET`     `/projects`                 List projects
  `POST`    `/projects`                 Create a project
  `GET`     `/tasks?project_id=N`       List tasks for a project
  `POST`    `/tasks`                    Create a task
  `PATCH`   `/tasks/:id`                Update a task, such as its status
  `GET`     `/search?q=&project_id=N`   Search tasks by title
  `GET`     `/health`                   Backend health check

Example:

``` powershell
curl.exe "http://localhost:8080/api/tasks?project_id=1"
```

For write requests, consult the backend handlers for the expected JSON
request body and validation rules.

## Environment Configuration

`.env.example` is the version-controlled template. Create `.env` locally
and configure the values expected by `docker-compose.yml`.

-   Keep credentials and tokens out of Git.
-   Do not use example database credentials for a public production
    deployment.
-   If a setting changes in `.env.example`, update this documentation
    where appropriate.

## Docker Images and Security Scanning

The frontend Docker work explored three image strategies:

  ------------------------------------------------------------------------------------------
  Dockerfile                         Build stage      Runtime stage    Port inside container
  ---------------------------------- ---------------- ---------------- ---------------------
  `frontend/Dockerfile.multistage`   Node.js 24       Node.js Alpine +                  3000
                                                      `serve`          

  `frontend/Dockerfile.alpine`       Node.js 24       Node.js Alpine +                  3000
                                     Alpine           `serve`          

  `frontend/Dockerfile.dhi`          Node.js 24       Docker Hardened                   8080
                                                      Nginx            
  ------------------------------------------------------------------------------------------

Build examples from the frontend directory:

``` powershell
docker build -f Dockerfile.multistage -t devboard-fe:multistage .
docker build -f Dockerfile.alpine -t devboard-fe:alpine .
docker build --pull --no-cache -f Dockerfile.dhi -t devboard-fe:dhi .
```

Docker Scout commands:

``` powershell
docker scout quickview local://devboard-fe:alpine
docker scout cves devboard-fe:alpine
docker scout policy local://devboard-fe:alpine

docker scout quickview local://devboard-fe:dhi
docker scout cves devboard-fe:dhi
docker scout policy local://devboard-fe:dhi
```

### Previously observed scan results

These are historical results from the scans performed during
development, not a guarantee of the current vulnerability status.

  Image                    Critical   High   Medium   Low
  ---------------------- ---------- ------ -------- -----
  `devboard-fe:dhi`               3     16       15    39
  `devboard-fe:alpine`            0     10       12     1

The DHI scan reported vulnerabilities in packages including OpenSSL,
glibc and PCRE2. The Alpine image had fewer reported findings in that
scan, but still had High and Medium findings.

A successful build or the use of a hardened image does not guarantee
that an image is vulnerability-free. Check current scan output, use a
verified patched base image where available, rebuild, and scan again.
Also review non-root execution and supply-chain attestation
requirements.

## CI/CD and DevSecOps

The repository description indicates that GitHub Actions workflows are
configured for security scanning with:

-   **SonarQube:** static analysis (SAST)
-   **OWASP ZAP:** dynamic application security testing (DAST)

Review the workflow files under `.github/workflows/` to confirm trigger
conditions, scan targets, and whether the most recent runs succeeded. A
configured workflow should not be described as passing until its run
results have been checked.

### SonarQube on an EC2 instance

For a self-hosted SonarQube test instance, ensure Docker is installed on
the EC2 host. A basic container command is:

``` bash
docker run -d --name sonarqube -p 127.0.0.1:9000:9000 sonarqube:community
```

This binds the web interface to the EC2 host's loopback interface. To
access it remotely, prefer an SSH tunnel or a properly secured reverse
proxy/private network. Avoid opening port `9000` to the entire internet.

If you intentionally configure remote access, restrict the EC2
security-group inbound rule to trusted source IPs and follow the current
SonarQube deployment guidance. Change the initial administrator password
at first login.

### GitHub Actions secrets

In GitHub, open **Settings → Secrets and variables → Actions** and add
only the values required by your workflow.

Typical names may include:

-   `SONAR_TOKEN` --- a token created in SonarQube/SonarCloud
-   `SONAR_HOST_URL` --- the SonarQube server URL, if the workflow
    expects it
-   `DOCKERHUB_USERNAME` --- Docker Hub username (often configured as a
    variable)
-   `DOCKERHUB_TOKEN` --- Docker Hub access token (store as a secret)

The exact names must match the workflow YAML. Never commit token values,
passwords, or private keys into source files.

## Docker Model Runner (Optional Learning)

Docker Model Runner was also explored as part of local AI tooling
practice. It is not part of the DevBoard application architecture
described above.

Check its availability with:

``` powershell
docker model status
```

If it is not running, enable Docker Model Runner in Docker Desktop
settings and use the commands supported by your installed Docker
version.

## Project Structure

``` text
.
├── docker-compose.yml
├── Makefile
├── .env.example
├── frontend/
│   ├── Dockerfile
│   ├── Dockerfile.multistage
│   ├── Dockerfile.alpine
│   ├── Dockerfile.dhi
│   ├── package.json
│   └── src/
├── backend/
│   ├── Dockerfile
│   └── main.go
└── init/
    └── postgres/
```

The layout above reflects the documented project structure; exact files
may vary. Check the repository before relying on every optional file or
Makefile target.

## Troubleshooting

### Port already in use

Check running containers:

``` powershell
docker ps
docker ps -a
```

Stop and remove a conflicting container, or change the host port in the
Compose configuration.

### Check logs

``` powershell
docker compose logs
docker compose logs frontend
docker compose logs backend
docker compose logs postgres
```

### Database initialization did not run again

PostgreSQL initialization scripts normally run only when the database
data directory is first initialized. If you need a fresh database, back
up any important data first, then intentionally remove the Compose
volume or use the project's reset procedure.

### Docker Scout reports vulnerabilities

Read the package names and fixed versions in the scan output. Update to
an appropriate patched image or dependency, rebuild, and scan again. Do
not assume a newer tag fixes every finding without checking the new
scan.

## Learning Outcomes

This project provided practice with:

-   Connecting a React frontend, Go API, and PostgreSQL database
-   Docker images, containers, networks, port mappings, and volumes
-   Manual container orchestration versus Docker Compose
-   Multi-stage Docker builds
-   Comparing Node/Alpine and Nginx-based runtime images
-   Docker Scout vulnerability and policy scanning
-   GitHub Actions security scanning concepts, including SAST and DAST
-   Environment configuration and safer secret handling
-   Exploring Docker Model Runner separately from the application

## License

Add the license and attribution information that applies to this
repository. Do not assume a license unless one has been selected for the
project.
