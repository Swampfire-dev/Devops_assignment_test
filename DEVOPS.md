# DevOps Assessment – Full Stack Application

## Overview

This repository contains a **full-stack web application** containerized and orchestrated using **Docker** and **Docker Compose**.
The project was built as part of a DevOps assessment to demonstrate **containerization, service orchestration, environment-based configuration, and real-world troubleshooting**.

The focus of this project is not only to run the application, but also to document **design decisions, issues faced, and how they were resolved**—similar to real DevOps work in production environments.

---

## Application Architecture

### High-Level Architecture

```
Browser
   |
   |  HTTP (Port 80 / 5173)
   v
Nginx (Frontend Container)
   |
   |  Docker internal network
   v
Django Backend (Port 8000)
```

### Components

* **Frontend**

  * React (Vite, TypeScript)
  * Built as static assets
  * Served via Nginx

* **Backend**

  * Django REST API
  * Exposes API endpoints on port `8000`
  * Configured using environment variables

* **Web Server**

  * Nginx
  * Serves frontend static files
  * (Optional enhancement) Can proxy API requests to backend

* **Container Platform**

  * Docker
  * Docker Compose for orchestration

* **Operating System**

  * CentOS 10 VM

---

## Repository Structure

```
devops-assessment/
├── backend/
│   ├── Dockerfile
│   ├── manage.py
│   ├── requirements.txt
│   └── config/
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── src/
├── docker-compose.yml
├── .env
└── README.md
```

---

## Setup Guide

### Prerequisites

Make sure the following tools are installed on your system:

* Docker
* Docker Compose (v2 or later)
* Git

Verify installation:

```bash
docker --version
docker compose version
git --version
```

---

### Clone the Repository

```bash
git clone https://github.com/Nexgensis/devops-assessment.git
cd devops-assessment
```

---

### Environment Configuration

Create a `.env` file in the project root:

```env
DJANGO_DEBUG=False
ALLOWED_HOSTS=*
VITE_API_URL=http://backend:8000
```

**Why environment variables?**

* Keeps configuration separate from code
* Allows the same image to run in different environments
* Follows 12-factor app principles

---

## Backend (Django) – Design & Setup

### Docker Design Decisions

* Base image: `python:3.10-slim`
* Dependencies installed via `requirements.txt`
* Application listens on `0.0.0.0:8000` (required for containers)
* Configuration read from environment variables

### Backend Startup Command

```dockerfile
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

> Note: Django development server is sufficient for this assessment.
> In production, this would be replaced with Gunicorn or another WSGI server.

---

## Frontend (React + Nginx) – Design & Setup

### Docker Design Decisions

* **Multi-stage Docker build**

  * Build stage: Node.js (for compiling React app)
  * Runtime stage: Nginx (lightweight, secure)
* Static assets copied into Nginx image
* Application exposed on port `80`
* Container runs with hardened security (non-root)

### Why Multi-Stage Builds?

* Smaller final image
* No Node.js in runtime container
* Faster startup
* Reduced attack surface

---

## Orchestration with Docker Compose

Docker Compose is used to:

* Start frontend and backend together
* Attach both services to a shared Docker network
* Inject environment variables at runtime
* Simplify startup with a single command

### Start the Application

```bash
docker compose up -d --build
```

Verify running containers:

```bash
docker ps
```

Access the application:

```text
http://<VM-IP>
```

---

## Troubleshooting & Issues Faced

This section documents **real issues encountered during implementation** and how they were resolved.

---

### Issue 1: Frontend Build Failure (Node.js Version Mismatch)

**Error**

```text
ERROR: failed to build: process "/bin/sh -c npm run build" did not complete successfully
```

**Root Cause**

* Frontend required a newer Node.js version
* Dockerfile was using `node:18`
* Vite build tools were incompatible

**Resolution**

* Updated Dockerfile to use:

  ```dockerfile
  FROM node:22-alpine AS build
  ```
* Rebuilt frontend image

---

### Issue 2: Frontend Container Exiting Immediately

**Symptoms**

* Container started but exited immediately
* Frontend not visible in browser

**Error Logs**

```text
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
```

**Root Cause**

* Nginx running with non-root user
* Required directories owned by root
* Nginx unable to create temp/cache files

**Resolution**

* Created `appuser` and `appgroup`
* Updated directory ownership:

  ```dockerfile
  RUN mkdir -p /var/cache/nginx /var/run /var/log/nginx \
      && chown -R appuser:appgroup /var/cache/nginx /var/run /var/log/nginx
  ```

---

### Issue 3: Incorrect Nginx Configuration Context

**Error**

```text
"user" directive is not allowed here
```

**Root Cause**

* `user appuser;` was added to `default.conf`
* `user` directive is only allowed in main `nginx.conf`

**Resolution**

* Removed `user` directive from `default.conf`
* Allowed Nginx to use default privilege model

---

### Issue 4: Frontend Unable to Connect to Backend (DisallowedHost)

**Error**

```text
django.core.exceptions.DisallowedHost
```

**Root Cause**

* `ALLOWED_HOSTS` was hardcoded as empty list
* Environment variable existed but Django did not read it

**Resolution**

* Updated `settings.py`:

  ```python
  import os
  ALLOWED_HOSTS = os.getenv("ALLOWED_HOSTS", "*").split(",")
  ```
* Rebuilt backend container

---

## Key Design Learnings

* Docker build context controls what goes into images
* Environment variables must be explicitly read by applications
* Non-root containers require filesystem preparation
* Nginx configuration context matters
* Docker service names resolve only inside Docker networks
* Logs are critical for debugging container issues

---

## Notes & Future Improvements

* Replace Django dev server with Gunicorn
* Add HTTPS support
* Implement CI/CD using GitHub Actions
* Deploy to cloud (AWS / Azure)
* Use reverse proxy for API to avoid CORS

---

## Final Status

* ✅ Frontend and backend containers running
* ✅ Docker Compose orchestration working
* ✅ Issues documented and resolved
* ✅ Ready for review and submission

