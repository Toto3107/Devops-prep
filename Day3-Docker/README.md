# 📘 DevOps Production Prep – Day 3

## Docker in Production: Real Incidents, Failures & Fixes

> **Objective:**
> Understand how Docker behaves in **real production environments**, why containers fail, and how DevOps engineers debug and fix them under pressure.

This day focuses on **real outages**, not basic Docker commands.

---

## 🧠 Why Docker Fails in Production

Docker issues are rarely about syntax.
They happen due to:

* Misconfigured images
* Resource limits
* Missing environment variables
* Bad Dockerfile practices
* CI/CD assumptions

---

## 🛠️ Production Environment Context

* Application: Backend API (Node.js / Python)
* Build via CI
* Image deployed to:

  * Kubernetes / ECS / Docker Host
* Logs via `docker logs` / centralized logging

---

## 🚨 Incident 1: Container Exits Immediately

### ❌ Symptom

```bash
docker ps -a
```

```text
Exited (0) after 2 seconds
```

### 🧠 Root Cause

* CMD/ENTRYPOINT finishes execution
* No long-running foreground process

### 🔍 Diagnose

```bash
docker logs <container_id>
```

### ✅ Fix

Ensure container runs a foreground process:

```dockerfile
CMD ["npm", "start"]
```

📌 **Lesson:**

> A container must run one long-lived foreground process.

---

## 🚨 Incident 2: Docker Image Size Explodes (1GB+)

### ❌ Symptom

```bash
docker images
```

### 🧠 Root Cause

* Full OS base image
* Build tools inside runtime image
* No `.dockerignore`

### 🔍 Diagnose

```bash
docker history <image>
```

### ✅ Fix (Multi-stage Build)

```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist .
CMD ["node", "index.js"]
```

📌 **Lesson:**

> Build tools must never ship to production.

---

## 🚨 Incident 3: Environment Variables Missing

### ❌ Symptom

```text
Database connection failed
```

### 🧠 Root Cause

* ENV not passed during runtime
* `.env` not injected in CI/CD

### 🔍 Diagnose

```bash
docker exec -it <container> env
```

### ✅ Fix

```bash
docker run -e DB_HOST=postgres backend
```

📌 **Lesson:**

> Configuration must be injected, not baked.

---

## 🚨 Incident 4: Works Locally, Fails in Container

### 🧠 Root Cause

* Missing OS libraries
* Wrong base image
* Architecture mismatch

### 🔍 Diagnose

```bash
docker run -it backend sh
```

### ✅ Fix

```dockerfile
FROM python:3.11-slim
RUN apt-get update && apt-get install -y libpq-dev
```

📌 **Lesson:**

> Containers are isolated OS environments.

---

## 🚨 Incident 5: Container Crashes Under Load (OOMKilled)

### ❌ Symptom

```text
OOMKilled
```

### 🧠 Root Cause

* No memory limits
* App consumes uncontrolled memory

### 🔍 Diagnose

```bash
docker inspect <container>
```

### ✅ Fix

```bash
docker run --memory=512m backend
```

Or tune app:

```bash
NODE_OPTIONS="--max-old-space-size=256"
```

📌 **Lesson:**

> Always set memory limits in production.

---

## 🚨 Incident 6: Secrets Leaked in Docker Image

### ❌ Symptom

Secrets visible via:

```bash
docker history <image>
```

### 🧠 Root Cause

* `.env` copied
* No `.dockerignore`

### ✅ Fix

`.dockerignore`

```text
.env
.git
node_modules
```

📌 **Lesson:**

> Secrets must NEVER exist in images.

---

## 🚨 Incident 7: CI Build Passes, Prod Fails

### 🧠 Root Cause

* No health checks
* App crashes silently

### ✅ Fix

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

📌 **Lesson:**

> Health checks prevent bad deployments.

---

## 🚨 Incident 8: Docker Build Very Slow

### 🧠 Root Cause

* Poor layer caching
* COPY done too early

### ✅ Fix

```dockerfile
COPY package.json .
RUN npm install
COPY . .
```

📌 **Lesson:**

> Docker caching is a performance weapon.

---

## 🧠 Production Docker Rules

1. One process per container
2. Multi-stage builds always
3. ENV-based configuration
4. Secrets injected at runtime
5. Set CPU & memory limits
6. Logs to stdout/stderr
7. Small images deploy faster

---

## 🎯 Interview Questions & Answers

**Q: Why containers exit immediately?**
A: Because the foreground process finished execution.

**Q: Why multi-stage builds?**
A: To reduce image size and remove build dependencies.

**Q: Why not store secrets in images?**
A: Images are immutable and widely distributed.

**Q: What causes OOMKilled?**
A: Unbounded memory usage without limits.