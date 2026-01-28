1. Project Overview

This project demonstrates the containerization and orchestration of a full-stack web application using Docker and Docker Compose.

Application Stack

Frontend: React (Vite, TypeScript)

Backend: Django (REST API)

Web Server: Nginx

Container Platform: Docker & Docker Compose

OS: CentOS VM 10

#############################################################################################################

2.Setup Guide

2.1 Prerequisites--
Ensure the following are installed on the system:

1.Docker

2.Docker Compose (v2+)

3.Git

Verify:

docker --version
docker compose version

2.2 Clone the Repository

git clone https://github.com/Nexgensis/devops-assessment.git
cd devops-assessment/

2.3  Environment Configuration

Create a .env file in the project root:
DJANGO_DEBUG=False
ALLOWED_HOSTS=*
VITE_API_URL=http://backend:8000

This file is used by Docker Compose to inject runtime configuration into containers.

2.4 Backend Setup (Docker)
Backend Dockerfile highlights:

Uses python:3.10-slim
Installs dependencies from requirements.txt
Runs Django on 0.0.0.0:8000
Uses environment variables for configuration

Backend startup command:
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

2.5 Frontend Setup (Docker)
Frontend Dockerfile highlights:
Multi-stage build
 Build stage: Node.js
 Runtime stage: Nginx
React app is built using Vite
Static files served by Nginx on port 80
Security hardening applied

2.6 Orchestration with Docker Compose

Docker Compose is used to:

Start frontend and backend together
Attach both services to a shared Docker network
Inject environment variables
Simplify startup with a single command
check docker-compose.yaml -- /opt/devops/devops-assisment



2.7 Build and Run the Application
 From the project root:
 docker compose up -d --build

Verify:
docker ps

Access application:
http://<VM-IP>:5173

#############################################################################################################

Issue 1: Frontend Build Failure Due to Node.js Version Mismatch

Error Encountered
ERROR: failed to build: failed to solve:
process "/bin/sh -c npm run build" did not complete successfully: exit code: 1


Root Cause

Frontend project required a newer Node.js version
Dockerfile was using node:18
Vite build tools were incompatible with Node 18

Resolution
Updated Node.js version in frontend Dockerfile
FROM node:22-alpine AS build

Rebuilt the frontend image
____________________________________________________________________________________________________________

Issue 2: Frontend Container Exiting Immediately After Startup

Symptoms
Container started but disappeared from docker ps
Frontend not accessible in browser

Error Logs
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)


Root Cause--
Nginx was running with hardened security
Required directories were owned by root
Nginx could not create cache/temp files

Resolution
Created appuser and appgroup
Changed ownership of required Nginx directories

RUN mkdir -p /var/cache/nginx /var/run /var/log/nginx \
    && chown -R appuser:appgroup /var/cache/nginx /var/run /var/log/nginx
_____________________________________________________________________________________________________________

Issue 3: Incorrect Nginx Configuration Context (user Directive Misuse)

Error Encountered
nginx: [emerg] "user" directive is not allowed here in /etc/nginx/conf.d/default.conf


Root Cause
 user appuser; was added to default.conf
 user directive is only valid in main nginx.conf

Resolution
 Removed user directive from default.conf
 Allowed Nginx to use its default privilege model:
    Root master process
    Non-root worker processes

_____________________________________________________________________________________________________________

Issue 4: Frontend Unable to Connect to Backend (DisallowedHost)

Error Encountered

django.core.exceptions.DisallowedHost:
Invalid HTTP_HOST header: 'xx.xx.xx.xx:8000'


Root Cause--
Django ALLOWED_HOSTS was hardcoded as empty list:
ALLOWED_HOSTS = []
Environment variable was set in Docker Compose but not read by Django

Resolution
Updated Django settings.py to read from environment variables:

import os
ALLOWED_HOSTS = os.getenv("ALLOWED_HOSTS", "*").split(",")


Rebuilt backend image using Docker Compose

_____________________________________________________________________________________________________________
