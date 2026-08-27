# Nikaasi Deployment and Operations Runbook

## Document Control

- **Document Version:** 1.0.0
- **Status:** Approved Operations Standard
- **Classification:** DevOps and Infrastructure Specification

---

## 1. System Architecture and Hosting Model

Nikaasi is built as a cloud-native, containerized web application designed for high availability and low-latency response across India:

```
+-------------------------------------------------------------+
|               Cloudflare CDN / Edge Routing                 |
+------------------------------+------------------------------+
                               | (HTTPS / TLS 1.3)
                               v
+-------------------------------------------------------------+
|                 Next.js App Server (Node.js)                |
|      - SSR / App Router for Dynamic Citizen Journeys        |
|      - Edge Middleware for Geolocation and Localization     |
+------------------------------+------------------------------+
                               |
               +---------------+---------------+
               |                               |
               v                               v
+-------------------------------+ +---------------------------+
| In-Memory Cache / State Store | | Immutable Audit Log Sink  |
+-------------------------------+ +---------------------------+
```

---

## 2. Local and Production Build Workflows

### 2.1 Production Build Execution

```bash
# Set production environment
export NODE_ENV=production

# Install dependencies with frozen lockfile
npm ci

# Build optimized production bundle
npm run build

# Start production server
npm run start -- -p 3000
```

### 2.2 Containerization (Dockerfile Reference)

```dockerfile
# Multi-stage production Dockerfile
FROM node:20-alpine AS base

# Step 1: Install dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Step 2: Build the application
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

# Step 3: Production runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000

CMD ["node", "server.js"]
```

---

## 3. Continuous Integration and Deployment (CI/CD)

The automated GitHub Actions workflow validates code quality, security vulnerabilities, and deployment health on every push to `main` and pull request:

```yaml
name: CI/CD Production Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  quality-gate:
    name: Code Quality & Linting
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js Environment
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Typecheck
        run: npm run typecheck --if-present

      - name: Run Linter
        run: npm run lint --if-present

      - name: Run Unit & Algorithm Tests
        run: npm test --if-present

  security-scan:
    name: Dependency Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm audit --audit-level=high
```

---

## 4. Observability, Telemetry, and Health Checks

### 4.1 Liveness and Readiness Probes

| Endpoint | Purpose | Expected Status | Response Payload |
| :--- | :--- | :--- | :--- |
| `GET /api/health/live` | Container liveness check | `200 OK` | `{"status": "ALIVE"}` |
| `GET /api/health/ready` | Readiness verification (sandbox loaded) | `200 OK` | `{"status": "READY", "sandbox": "ONLINE"}` |

### 4.2 Key Performance Indicators (KPIs) and SLOs

- **Pre-Flight Validation Latency:** p95 < 350ms, p99 < 800ms.
- **System Availability:** 99.9% uptime target.
- **Client Error Rate (4xx):** < 1.0% of total transactions.
- **Server Error Rate (5xx):** < 0.05% of total transactions.
