# Technology Stack Templates

> **Related Guides:**
> - Project setup: [01_project_setup.md](01_project_setup.md)
> - Development workflow: [03_development_and_maintenance.md](03_development_and_maintenance.md)
> - Deployment platforms: [09_deployment_platforms.md](09_deployment_platforms.md)
> - Database migrations: [14_database_migrations.md](14_database_migrations.md)


## Stack Decision Questions

Before implementation, ask these questions:


1. Frontend Requirements:

   - Framework preference? (React, Vue, Svelte, Angular)
   - SSR requirements? (Next.js, Nuxt, SvelteKit)
   - Static vs Dynamic?
   - UI component library preferences?


2. Backend Requirements:

   - Language preference? (Node.js, Python, Go, Java, Rust)
   - Framework preference? (Express, FastAPI, Gin, Spring, Actix)
   - API architecture? (REST, GraphQL, gRPC)
   - Authentication requirements?


3. Database Requirements:

   - Development database? (SQLite, PostgreSQL, MySQL)
   - Production database? (PostgreSQL, MySQL, MongoDB)
   - ORM preference?
   - Migration strategy?


4. Development Environment:

   - Local development setup?
   - CI/CD requirements?
   - Containerization needs?
   - Development team size?


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
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - NEXT_PUBLIC_API_URL=http://localhost:4000
```

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
      - ./backend:/app
      - /app/node_modules
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:pass@db:5432/devdb
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=devdb
    ports:
      - "5432:5432"
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data

volumes:
  postgres_dev_data:
```

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
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=https://api.production.com
```

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
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${PROD_DATABASE_URL}
```

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
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=development
      - VUE_APP_API_URL=http://localhost:8000
```

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
      - ./backend:/app
    ports:
      - "8000:8000"
    environment:
      - ENVIRONMENT=development
      - DATABASE_URL=mysql://user:pass@db:3306/devdb
    depends_on:
      - db

  db:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=devdb
      - MYSQL_USER=user
      - MYSQL_PASSWORD=pass
    ports:
      - "3306:3306"
    volumes:
      - mysql_dev_data:/var/lib/mysql

volumes:
  mysql_dev_data:
```

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
      - "80:80"
    environment:
      - NODE_ENV=production
      - VUE_APP_API_URL=https://api.production.com
```

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
      - "8000:8000"
    environment:
      - ENVIRONMENT=production
      - DATABASE_URL=${PROD_DATABASE_URL}
```

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
      - name: web
        image: app:latest
        ports:
        - containerPort: 80
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
```

### 2. Service Mesh Configuration

```yaml
# istio-config.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: app-routes
spec:
  hosts:
  - app.example.com
  gateways:
  - app-gateway
  http:
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: api-service
        port:
          number: 80
  - route:
    - destination:
        host: web-service
        port:
          number: 80
```

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
    - host: app.example.com
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
```

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
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    command: npm run dev

  db:
    image: postgres:13-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=dev
      - POSTGRES_PASSWORD=dev
      - POSTGRES_DB=app_dev

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

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
```

## YAML Configuration Testing

> **Related Testing Guides:**
> - General testing setup: [02_testing_setup.md](02_testing_setup.md)
> - AI-driven testing: [00_ai_guidelines.md](00_ai_guidelines.md)
> - Deployment testing: [09_deployment_platforms.md](09_deployment_platforms.md)


### 1. Pre-Deployment Validation

```bash
#!/bin/bash
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
```

### 2. Schema Validation

```yaml
# .yamllint.yml
extends: default

rules:
  line-length: disable
  document-start: disable
  truthy:
    check-keys: false
```

### 3. Environment Variable Testing

```bash
# test_env_substitution.sh
#!/bin/bash

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
```

### 4. CI/CD Integration

```yaml
# .github/workflows/yaml-validation.yml
name: YAML Validation

on:
  push:
    paths:
      - '**.yml'
      - '**.yaml'
  pull_request:
    paths:
      - '**.yml'
      - '**.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install tools
        run: |
          pip install yamllint
          sudo snap install yq
      - name: Validate YAML
        run: ./validate_yaml.sh
```

## Database Migration Strategies

### Development to Production

#### PostgreSQL to PostgreSQL

```bash
#!/bin/bash
# migrate_postgres.sh

# Dump development database
pg_dump -h localhost -U devuser -d devdb > dev_dump.sql

# Modify production connection settings
sed -i 's/devuser/produser/g' dev_dump.sql
sed -i 's/devdb/proddb/g' dev_dump.sql

# Import to production
psql -h production-host -U produser -d proddb < dev_dump.sql
```

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
```

#### MySQL to PostgreSQL

```bash
#!/bin/bash
# migrate_mysql_to_postgres.sh

# Export from MySQL
mysqldump -h localhost -u devuser -p devdb > mysql_dump.sql

# Convert to PostgreSQL format
pgloader mysql://devuser:pass@localhost/devdb \
  postgresql://produser:pass@production-host/proddb
```

## Environment-Specific Configurations

### Development

```yaml
# .env.development
NODE_ENV=development
API_URL=http://localhost:8000
DATABASE_URL=postgresql://user:pass@localhost:5432/devdb
REDIS_URL=redis://localhost:6379
```

### Production

```yaml
# .env.production
NODE_ENV=production
API_URL=https://api.production.com
DATABASE_URL=${PROD_DATABASE_URL}
REDIS_URL=${PROD_REDIS_URL}
```

## Implementation Guidelines

1. **Start with Development Environment**
   - Set up local databases first
   - Configure development tools
   - Set up hot-reloading
   - Enable debugging tools

2. **Create Production Configuration**
   - Remove development dependencies
   - Configure production databases
   - Set up SSL/TLS
   - Configure caching

3. **Set up CI/CD Pipeline**
   - Configure testing environments
   - Set up staging environment
   - Configure production deployment
   - Set up monitoring

4. **Database Migration Process**
   - Create migration scripts
   - Test with sample data
   - Plan production migration
   - Set up backup strategy

## Security Considerations

1. **Development**
   - Use dummy data
   - Local environment variables
   - Disable sensitive features
   - Use development certificates

2. **Production**
   - Secure environment variables
   - Production SSL certificates
   - Database encryption
   - Access control

## Monitoring Setup

1. **Development**
   - Local logging
   - Development metrics
   - Performance profiling
   - Error tracking

2. **Production**
   - Production logging
   - Metrics collection
   - Alert system
   - Performance monitoring
