---
name: devops-docker
description: Docker containerization, multi-stage builds, Docker Compose orchestration, and container security best practices
tags: [docker, containers, devops, compose]
author: Antigravity Team
version: 1.0.0
---

# Docker Containerization Skill

Comprehensive guide for Docker containerization.

## Dockerfile Best Practices

### Multi-stage Build (Node.js)
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:20-alpine AS production
WORKDIR /app

# Security: run as non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .

EXPOSE 3000
CMD ["node", "server.js"]
```

### Multi-stage Build (Python)
```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
RUN pip install --no-cache-dir poetry
COPY pyproject.toml poetry.lock ./
RUN poetry export -f requirements.txt > requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

RUN useradd -m appuser
USER appuser

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Docker Compose

### Full Stack Example
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - VITE_API_URL=http://localhost:4000
    depends_on:
      - api
    networks:
      - app-network

  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "4000:4000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    networks:
      - app-network
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

## Docker Commands

```bash
# Build
docker build -t myapp:latest .
docker build --target production -t myapp:prod .

# Run
docker run -d -p 3000:3000 --name myapp --env-file .env myapp:latest

# Compose
docker compose up -d
docker compose down -v  # Remove volumes too
docker compose logs -f api  # Follow logs
docker compose exec api sh  # Shell into container

# Cleanup
docker system prune -af
docker volume prune -f

# Inspect
docker stats
docker logs --tail 100 -f myapp
docker inspect myapp | jq '.[0].NetworkSettings'
```

## Security

```dockerfile
# Use specific versions, not latest
FROM node:20.11.0-alpine3.19

# Scan for vulnerabilities
# docker scout cves myapp:latest

# Read-only filesystem
docker run --read-only --tmpfs /tmp myapp

# Resource limits
docker run --memory="512m" --cpus="1.0" myapp

# No new privileges
docker run --security-opt=no-new-privileges myapp
```

## .dockerignore
```
node_modules
npm-debug.log
.git
.gitignore
.env*
*.md
tests/
coverage/
.dockerignore
Dockerfile*
docker-compose*
```

## Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```
