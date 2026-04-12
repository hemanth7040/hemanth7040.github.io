---
title: "Docker for DevOps: The Complete Guide"
date: 2026-04-12
draft: false
tags: ["docker", "devops", "containers", "infrastructure", "ci-cd"]
categories: ["docker"]
description: "A comprehensive guide to Docker — what it is, when to use it, where it fits in your pipeline, and all the mandatory commands every DevOps engineer must know."
cover:
  image: "docker-banner.png"
  alt: "Docker Container Architecture"
---

## What is Docker?

Docker is an open-source **containerization platform** that packages applications and their dependencies into lightweight, portable units called **containers**. Unlike virtual machines, containers share the host OS kernel, making them faster to start, smaller in size, and more resource-efficient.

A Docker container bundles:
- Application code
- Runtime environment
- System libraries and dependencies
- Configuration files

This means: *"Build once, run anywhere."*

---

## When to Use Docker?

| Scenario | Use Docker? | Reason |
|---|---|---|
| Microservices architecture | ✅ Yes | Isolate each service independently |
| CI/CD pipelines | ✅ Yes | Consistent build environments |
| Local development setup | ✅ Yes | Eliminate "works on my machine" issues |
| Multi-environment deployments | ✅ Yes | Same image across dev/staging/prod |
| Monolithic legacy apps | ⚠️ Sometimes | Evaluate complexity vs. benefit |
| Stateful databases (standalone) | ⚠️ Carefully | Use volumes and persistent storage |
| Simple static websites | ❌ Optional | May be overkill without orchestration |

**Best times to adopt Docker:**
- When onboarding new developers (instant environment setup)
- When scaling horizontally with Kubernetes or Docker Swarm
- When enforcing environment parity across teams
- During a migration from monolith to microservices

---

## Where Does Docker Fit?

```
Developer Laptop
      │
      ▼
 Docker Build  ──────────────────────────────────────┐
      │                                              │
      ▼                                              ▼
Docker Registry              Docker Compose (local multi-container)
  (Docker Hub /                      │
   AWS ECR /                         ▼
   GCR / ACR)              Kubernetes / Docker Swarm
      │                     (Production Orchestration)
      ▼
 CI/CD Pipeline
 (GitHub Actions / Jenkins / GitLab CI)
      │
      ▼
  Staging → Production
```

Docker integrates at every layer:
- **Dev**: Local containers replace system installs
- **CI**: Reproducible build agents
- **CD**: Image promotion across environments
- **Prod**: Orchestration with Kubernetes or Swarm

---

## Mandatory Docker Commands for DevOps

### 🔧 Image Management

```bash
# Build an image from Dockerfile
docker build -t my-app:1.0 .

# Build with no cache (clean build)
docker build --no-cache -t my-app:1.0 .

# Build for a specific platform (multi-arch)
docker buildx build --platform linux/amd64,linux/arm64 -t my-app:1.0 .

# List all images
docker images

# Pull an image from registry
docker pull nginx:latest

# Push image to registry
docker tag my-app:1.0 myrepo/my-app:1.0
docker push myrepo/my-app:1.0

# Remove an image
docker rmi my-app:1.0

# Remove all unused images (prune)
docker image prune -a
```

---

### 🚀 Container Lifecycle

```bash
# Run a container (detached mode)
docker run -d --name my-container -p 8080:80 my-app:1.0

# Run with environment variables
docker run -d -e ENV=production -e DB_HOST=localhost my-app:1.0

# Run with volume mount
docker run -d -v /host/path:/container/path my-app:1.0

# Run interactively (useful for debugging)
docker run -it --rm ubuntu:22.04 bash

# Stop a container
docker stop my-container

# Start a stopped container
docker start my-container

# Restart a container
docker restart my-container

# Remove a container
docker rm my-container

# Force remove a running container
docker rm -f my-container
```

---

### 🔍 Inspect & Debug

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs my-container

# Follow logs in real time
docker logs -f my-container

# Show last N lines of logs
docker logs --tail 100 my-container

# Execute command inside running container
docker exec -it my-container bash

# Inspect container metadata
docker inspect my-container

# View container resource usage
docker stats

# View container processes
docker top my-container
```

---

### 🗂️ Volumes & Networking

```bash
# Create a named volume
docker volume create my-data

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect my-data

# Remove unused volumes
docker volume prune

# Create a network
docker network create my-network

# List networks
docker network ls

# Connect container to network
docker network connect my-network my-container

# Inspect network
docker network inspect my-network
```

---

### 🐙 Docker Compose (Multi-Container)

```bash
# Start all services (detached)
docker compose up -d

# Start and force rebuild images
docker compose up -d --build

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v

# View logs for all services
docker compose logs -f

# View logs for specific service
docker compose logs -f web

# Scale a service
docker compose up -d --scale worker=3

# Execute command in a service container
docker compose exec web bash

# List running compose services
docker compose ps

# Pull latest images for compose services
docker compose pull
```

---

### 🧹 Cleanup & Maintenance

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images, containers, networks, volumes
docker system prune -a --volumes

# Check disk usage by Docker
docker system df

# Remove dangling images (untagged)
docker image prune

# Remove specific network
docker network rm my-network
```

---

### 🔐 Registry & Authentication

```bash
# Login to Docker Hub
docker login

# Login to private registry (e.g., AWS ECR)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Logout
docker logout

# Tag image for private registry
docker tag my-app:1.0 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0

# Push to private registry
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0
```

---

### ⚙️ Dockerfile Best Practices

```dockerfile
# Use specific version tags, never 'latest' in production
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy dependency files first (layer caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Document exposed port
EXPOSE 3000

# Use ENTRYPOINT for fixed commands, CMD for defaults
ENTRYPOINT ["node"]
CMD ["server.js"]
```

---

### 📊 Docker in CI/CD (GitHub Actions Example)

```yaml
name: Docker Build & Push

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: myrepo/my-app:${{ github.sha }}
```

---

## Quick Reference Cheatsheet

```bash
# THE MOST USED COMMANDS AT A GLANCE

docker build -t app:tag .              # Build image
docker run -d -p 8080:80 app:tag       # Run container
docker ps                              # List running containers
docker logs -f container_name          # Tail logs
docker exec -it container_name bash    # Shell access
docker stop container_name            # Stop container
docker rm container_name              # Remove container
docker compose up -d --build          # Start compose stack
docker system prune -a                # Full cleanup
```

---

## Summary

Docker is the backbone of modern DevOps workflows. Mastering these commands gives you full control over the container lifecycle — from local development to production deployments. Pair Docker with **Kubernetes** for orchestration, **Docker Compose** for local multi-service setups, and **CI/CD pipelines** for automated delivery.

> 💡 **Pro Tip:** Always use `.dockerignore` to exclude unnecessary files (like `node_modules`, `.git`, `.env`) from your build context to keep images small and builds fast.

---

*Last updated: April 2025 | Tags: docker, devops, containers, ci-cd*