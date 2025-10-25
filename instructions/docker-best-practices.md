# Docker Best Practices

Docker best practices including Dockerfile optimization, multi-stage builds, security hardening, and image size reduction. These guidelines help AI assistants generate efficient, secure Docker configurations.

## Core Guidelines

### 1. Use Multi-Stage Builds

Separate build dependencies from runtime dependencies using multi-stage builds. This dramatically reduces final image size.

**Example**:
```dockerfile
# Good (multi-stage build)
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]

# Avoid (single stage with all dependencies)
FROM node:18
WORKDIR /app
COPY . .
RUN npm install  # Includes dev dependencies
RUN npm run build
CMD ["node", "dist/index.js"]
```

### 2. Use Specific Base Image Tags

Always specify exact version tags for base images. Never use `latest` tag in production. Use Alpine variants for smaller images.

**Example**:
```dockerfile
# Good (specific version)
FROM python:3.11-slim
FROM node:18.17-alpine
FROM nginx:1.25-alpine

# Avoid (latest or no tag)
FROM python
FROM python:latest
FROM node
```

### 3. Minimize Layers and Combine RUN Commands

Combine related RUN commands with `&&` to reduce layers. Clean up package manager caches in the same RUN command.

**Example**:
```dockerfile
# Good (combined commands, cleaned cache)
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        python3 \
        python3-pip \
        curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Python example
FROM python:3.11-slim
RUN pip install --no-cache-dir \
        fastapi==0.104.0 \
        uvicorn==0.24.0 \
        pydantic==2.5.0

# Avoid (multiple RUN commands, cache left behind)
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y curl
# /var/lib/apt/lists/ still consuming space
```

### 4. Copy Only What's Needed

Use `.dockerignore` to exclude unnecessary files. Copy dependency files before source code to leverage layer caching.

**Example**:
```dockerfile
# Good (optimal layer caching)
FROM node:18-alpine
WORKDIR /app

# Copy dependency files first (cached if unchanged)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code last (changes frequently)
COPY src/ ./src/
COPY tsconfig.json ./

CMD ["npm", "start"]

# .dockerignore file:
# node_modules
# .git
# .env
# *.log
# dist
# coverage

# Avoid (copy everything, no caching benefit)
FROM node:18-alpine
WORKDIR /app
COPY . .  # Copies everything, breaks cache on any file change
RUN npm install
```

### 5. Run as Non-Root User

Create and use a non-root user for running applications. This limits damage if container is compromised.

**Example**:
```dockerfile
# Good (non-root user)
FROM python:3.11-slim

# Create app user
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]

# Also Good (use node user in Node images)
FROM node:18-alpine
WORKDIR /app
COPY --chown=node:node . .
RUN npm ci --only=production
USER node
CMD ["node", "server.js"]

# Avoid (running as root)
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]  # Runs as root!
```

### 6. Use COPY Instead of ADD

Use `COPY` for copying local files. Use `ADD` only for tar extraction or remote URLs (rare cases). COPY is more explicit and predictable.

**Example**:
```dockerfile
# Good (COPY for local files)
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
COPY src/ ./src/
COPY config/ ./config/

# Avoid (ADD for simple copying)
FROM node:18-alpine
WORKDIR /app
ADD package*.json ./  # Unnecessary, use COPY
ADD src/ ./src/
```

### 7. Set Meaningful Labels and Metadata

Add labels for image metadata including version, maintainer, and description. Use standard label keys.

**Example**:
```dockerfile
# Good (labeled image)
FROM python:3.11-slim

LABEL org.opencontainers.image.title="My Application" \
      org.opencontainers.image.description="RESTful API service" \
      org.opencontainers.image.version="1.0.0" \
      org.opencontainers.image.authors="team@example.com" \
      org.opencontainers.image.source="https://github.com/org/repo"

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
USER nobody

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]

# Avoid (no metadata)
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

## Quick Reference

- [ ] Multi-stage builds used to separate build from runtime
- [ ] Base images use specific version tags (not latest)
- [ ] Alpine or slim variants used to minimize size
- [ ] Related RUN commands combined with && to reduce layers
- [ ] Package manager caches cleaned in same RUN command
- [ ] .dockerignore file excludes unnecessary files
- [ ] Dependency files copied before source for caching
- [ ] Application runs as non-root user
- [ ] COPY used instead of ADD for local files
- [ ] Labels added for image metadata
- [ ] HEALTHCHECK defined for container health monitoring
- [ ] Single process per container (not using supervisord)
