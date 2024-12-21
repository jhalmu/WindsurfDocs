
# Technology Stack Templates

> **Related Guides:**
> - Project setup: [01_project_setup.md](01_project_setup.md)
> - Development workflow: [03_development_and_maintenance.md](03_development_and_maintenance.md)
> - Deployment platforms: [09_deployment_platforms.md](09_deployment_platforms.md)
> - Database migrations: [14_database_migrations.md](14_database_migrations.md)

## Stack Decision Questions

Before implementation, ask these questions:


1. Frontend Requirements:

      -$2Framework preference? (React, Vue, Svelte, Angular)
    -$2SSR requirements? (Next.js, Nuxt, SvelteKit)
    -$2Static vs Dynamic?
    -$2UI component library preferences?


1. Backend Requirements:

      -$2Language preference? (Node.js, Python, Go, Java, Rust)
    -$2Framework preference? (Express, FastAPI, Gin, Spring, Actix)
    -$2API architecture? (REST, GraphQL, gRPC)
    -$2Authentication requirements?


1. Database Requirements:

      -$2Development database? (SQLite, PostgreSQL, MySQL)
    -$2Production database? (PostgreSQL, MySQL, MongoDB)
    -$2ORM preference?
    -$2Migration strategy?


1. Development Environment:

      -$2Local development setup?
    -$2CI/CD requirements?
    -$2Containerization needs?
    -$2Development team size?

## Template Path 1: Modern JavaScript Full Stack

### Development Environment

#### Frontend (Next.js)

```yaml

# docker-compose.dev.frontend.yml

version: '3.8'
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    volumes:
        -$2./frontend:/app
        -$2/app/node_modules

    ports:
        -$2"3000:3000"

    environment:
        -$2NODE_ENV=development
        -$2NEXT_PUBLIC_API_URL=<<http://localhost:4000>>

```text

#### Backend (Node.js/Express)

```yaml

# docker-compose.dev.backend.yml

version: '3.8'
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    volumes:
        -$2./backend:/app
        -$2/app/node_modules

    ports:
        -$2"4000:4000"

    environment:
        -$2NODE_ENV=development
        -$2DATABASE_URL=postgresql://user:pass@db:5432/devdb

    depends_on:
        -$2db

  db:
    image: postgres:15-alpine
    environment:
        -$2POSTGRES_USER=user
        -$2POSTGRES_PASSWORD=pass
        -$2POSTGRES_DB=devdb

    ports:
        -$2"5432:5432"

    volumes:
        -$2postgres_dev_data:/var/lib/postgresql/data

volumes:
  postgres_dev_data:

```text

### Production Environment

#### Frontend (Next.js)

```yaml

# docker-compose.prod.frontend.yml

version: '3.8'
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    ports:
        -$2"3000:3000"

    environment:
        -$2NODE_ENV=production
        -$2NEXT_PUBLIC_API_URL=<<https://api.production.com>>

```text

#### Backend (Node.js/Express)

```yaml

# docker-compose.prod.backend.yml

version: '3.8'
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    ports:
        -$2"4000:4000"

    environment:
        -$2NODE_ENV=production
        -$2DATABASE_URL=${PROD_DATABASE_URL}

```text

## Template Path 2: Python/Vue Stack

### Development Environment

#### Frontend (Vue.js)

```yaml

# docker-compose.dev.frontend.yml

version: '3.8'
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    volumes:
        -$2./frontend:/app
        -$2/app/node_modules

    ports:
        -$2"8080:8080"

    environment:
        -$2NODE_ENV=development
        -$2VUE_APP_API_URL=<<http://localhost:8000>>

```text

#### Backend (Python/FastAPI)

```yaml

# docker-compose.dev.backend.yml

version: '3.8'
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    volumes:
        -$2./backend:/app

    ports:
        -$2"8000:8000"

    environment:
        -$2ENVIRONMENT=development
        -$2DATABASE_URL=mysql://user:pass@db:3306/devdb

    depends_on:
        -$2db

  db:
    image: mysql:8
    environment:
        -$2MYSQL_ROOT_PASSWORD=rootpass
        -$2MYSQL_DATABASE=devdb
        -$2MYSQL_USER=user
        -$2MYSQL_PASSWORD=pass

    ports:
        -$2"3306:3306"

    volumes:
        -$2mysql_dev_data:/var/lib/mysql

volumes:
  mysql_dev_data:

```text

### Production Environment

#### Frontend (Vue.js)

```yaml

# docker-compose.prod.frontend.yml

version: '3.8'
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    ports:
        -$2"80:80"

    environment:
        -$2NODE_ENV=production
        -$2VUE_APP_API_URL=<<https://api.production.com>>

```text

#### Backend (Python/FastAPI)

```yaml

# docker-compose.prod.backend.yml

version: '3.8'
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    ports:
        -$2"8000:8000"

    environment:
        -$2ENVIRONMENT=production
        -$2DATABASE_URL=${PROD_DATABASE_URL}

```text

## Container Orchestration

> **Related Sections:**
> - For deployment strategies: See [04_deployment.md](04_deployment.md)
> - For monitoring setup: See [10_troubleshooting.md](10_troubleshooting.md)
> - For AI integration: See [00_ai_guidelines.md](00_ai_guidelines.md)

### 1. Kubernetes Templates

```yaml

# deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        -$2name: web

        image: app:latest
        ports:
          -$2containerPort: 80

        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
          requests:
            cpu: "0.5"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

```text

### 2. Service Mesh Configuration

```yaml

# istio-config.yaml

apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: app-routes
spec:
  hosts:
    -$2app.example.com

  gateways:
    -$2app-gateway

  http:
    -$2match:
      -$2uri:

        prefix: /api
    route:
      -$2destination:

        host: api-service
        port:
          number: 80
    -$2route:
      -$2destination:

        host: web-service
        port:
          number: 80

```text

### 3. Helm Charts

```yaml

# values.yaml

replicaCount: 3
image:
  repository: app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
      -$2host: app.example.com

      paths: ["/"]

resources:
  limits:
    cpu: 1
    memory: 512Mi
  requests:
    cpu: 500m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

```text

### 4. Docker Compose for Development

```yaml

# docker-compose.dev.yml

version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    volumes:
        -$2.:/app
        -$2/app/node_modules

    ports:
        -$2"3000:3000"

    environment:
        -$2NODE_ENV=development

    command: npm run dev

  db:
    image: postgres:13-alpine
    volumes:
        -$2postgres_data:/var/lib/postgresql/data

    environment:
        -$2POSTGRES_USER=dev
        -$2POSTGRES_PASSWORD=dev
        -$2POSTGRES_DB=app_dev

  redis:
    image: redis:alpine
    ports:
        -$2"6379:6379"

    volumes:
        -$2redis_data:/data

volumes:
  postgres_data:
  redis_data:

```text

### 5. Multi-Stage Builds

```dockerfile

# Dockerfile

# Build stage

FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Development stage

FROM node:16-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# Production stage

FROM node:16-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["npm", "start"]

```text

## YAML Configuration Testing

> **Related Testing Guides:**
> - General testing setup: [02_testing_setup.md](02_testing_setup.md)
> - AI-driven testing: [00_ai_guidelines.md](00_ai_guidelines.md)
> - Deployment testing: [09_deployment_platforms.md](09_deployment_platforms.md)

### 1. Pre-Deployment Validation

```bash
```bash
#!/bin/bash
```

# validate_yaml.sh

# Required tools

command -v yamllint >/dev/null 2>&1 || { echo "yamllint required"; exit 1; }
command -v yq >/dev/null 2>&1 || { echo "yq required"; exit 1; }

# Validate YAML syntax

validate_yaml() {
    local file=$1
    echo "Validating $file..."
    if ! yamllint -d relaxed "$file"; then
        echo "❌ YAML syntax error in $file"
        return 1
    fi
    echo "✅ $file is valid"
    return 0
}

# Find and validate all YAML files

find . -type f \( -name "*.yml" -o -name "*.yaml" \) -exec bash -c 'validate_yaml "$0"' {} \;

```text

### 2. Schema Validation

```yaml

# .yamllint.yml

extends: default

rules:
  line-length: disable
  document-start: disable
  truthy:
    check-keys: false

```text

### 3. Environment Variable Testing

```bash

# test_env_substitution.sh

```bash
#!/bin/bash
```

test_env_substitution() {
    local file=$1
    echo "Testing env vars in: $file"

    # Create test environment file
    cat > .env.test << EOF
    DATABASE_URL=postgresql://test:test@localhost:5432/testdb
    API_KEY=test_key
    NODE_ENV=test
EOF

    # Test substitution
    if ! docker-compose --env-file .env.test -f "$file" config > /dev/null 2>&1; then
        echo "❌ Environment variable substitution failed in: $file"
        return 1
    fi

    echo "✅ Environment variable substitution successful in: $file"
    return 0
}

# Test all docker-compose files

find . -type f -name "docker-compose*.yml" -exec bash -c 'test_env_substitution "$0"' {} \;

```text

### 4. CI/CD Integration

```yaml

# .github/workflows/yaml-validation.yml

name: YAML Validation

on:
  push:
    paths:
        -$2'**.yml'
        -$2'**.yaml'

  pull_request:
    paths:
        -$2'**.yml'
        -$2'**.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
        -$2uses: actions/checkout@v2
        -$2name: Install tools

        run: |
          pip install yamllint
          sudo snap install yq
        -$2name: Validate YAML

        run: ./validate_yaml.sh

```text

## Database Migration Strategies

### Development to Production

#### PostgreSQL to PostgreSQL

```bash
```bash
#!/bin/bash
```

# migrate_postgres.sh

# Dump development database

pg_dump -h localhost -U devuser -d devdb > dev_dump.sql

# Modify production connection settings

sed -i 's/devuser/produser/g' dev_dump.sql
sed -i 's/devdb/proddb/g' dev_dump.sql

# Import to production

psql -h production-host -U produser -d proddb < dev_dump.sql

```text

#### SQLite to PostgreSQL

```python

# migrate_sqlite_to_postgres.py

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

def migrate_data():
    sqlite_engine = create_engine('sqlite:///dev.db')
    postgres_engine = create_engine('postgresql://user:pass@host/proddb')

    Session = sessionmaker(bind=sqlite_engine)
    sqlite_session = Session()

    ProdSession = sessionmaker(bind=postgres_engine)
    prod_session = ProdSession()

    # Migration logic here

```text

#### MySQL to PostgreSQL

```bash
```bash
#!/bin/bash
```

# migrate_mysql_to_postgres.sh

# Export from MySQL

mysqldump -h localhost -u devuser -p devdb > mysql_dump.sql

# Convert to PostgreSQL format

pgloader mysql://devuser:pass@localhost/devdb \
  postgresql://produser:pass@production-host/proddb

```text

## Environment-Specific Configurations

### Development

```yaml

# .env.development

NODE_ENV=development
API_URL=<<http://localhost:8000>>
DATABASE_URL=postgresql://user:pass@localhost:5432/devdb
REDIS_URL=redis://localhost:6379

```text

### Production

```yaml

# .env.production

NODE_ENV=production
API_URL=<<https://api.production.com>>
DATABASE_URL=${PROD_DATABASE_URL}
REDIS_URL=${PROD_REDIS_URL}

```text

## Implementation Guidelines


1. **Start with Development Environment**

    -$2Set up local databases first
    -$2Configure development tools
    -$2Set up hot-reloading
    -$2Enable debugging tools


1. **Create Production Configuration**

    -$2Remove development dependencies
    -$2Configure production databases
    -$2Set up SSL/TLS
    -$2Configure caching


1. **Set up CI/CD Pipeline**

    -$2Configure testing environments
    -$2Set up staging environment
    -$2Configure production deployment
    -$2Set up monitoring


1. **Database Migration Process**

    -$2Create migration scripts
    -$2Test with sample data
    -$2Plan production migration
    -$2Set up backup strategy

## Security Considerations


1. **Development**

    -$2Use dummy data
    -$2Local environment variables
    -$2Disable sensitive features
    -$2Use development certificates


1. **Production**

    -$2Secure environment variables
    -$2Production SSL certificates
    -$2Database encryption
    -$2Access control

## Monitoring Setup


1. **Development**

    -$2Local logging
    -$2Development metrics
    -$2Performance profiling
    -$2Error tracking


1. **Production**

    -$2Production logging
    -$2Metrics collection
    -$2Alert system
    -$2Performance monitoring
