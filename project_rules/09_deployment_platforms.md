# Deployment Platforms Guide

## Docker Deployment

### Container Setup
1. **Base Configuration**
   ```dockerfile
   FROM node:latest
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build
   ```

2. **Multi-Stage Build**
   ```dockerfile
   # Build stage
   FROM node:latest AS builder
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build

   # Production stage
   FROM node:alpine
   COPY --from=builder /app/dist /app
   CMD ["npm", "start"]
   ```

### Database Integration
1. **Containerized Database**
   ```yaml
   version: '3.8'
   services:
     db:
       image: postgres:latest
       environment:
         POSTGRES_PASSWORD: ${DB_PASSWORD}
         POSTGRES_DB: ${DB_NAME}
       volumes:
         - db-data:/var/lib/postgresql/data
   ```

2. **External Database**
   ```yaml
   version: '3.8'
   services:
     app:
       environment:
         DATABASE_URL: ${EXTERNAL_DB_URL}
   ```

## Cloud Platform Deployments

### AWS Deployment

1. **ECS Fargate**
   ```yaml
   AWSTemplateFormatVersion: '2010-09-09'
   Resources:
     ECSCluster:
       Type: AWS::ECS::Cluster
     TaskDefinition:
       Type: AWS::ECS::TaskDefinition
       Properties:
         RequiresCompatibilities:
           - FARGATE
   ```

2. **RDS Setup**
   ```yaml
   Resources:
     Database:
       Type: AWS::RDS::DBInstance
       Properties:
         Engine: postgres
         EngineVersion: 13.7
   ```

### Google Cloud Platform

1. **Cloud Run**
   ```yaml
   steps:
   - name: 'gcr.io/cloud-builders/docker'
     args: ['build', '-t', 'gcr.io/$PROJECT_ID/app', '.']
   - name: 'gcr.io/cloud-builders/docker'
     args: ['push', 'gcr.io/$PROJECT_ID/app']
   ```

2. **Cloud SQL**
   ```yaml
   resources:
   - name: database
     type: gcp-types/sqladmin-v1beta4:instances
     properties:
       databaseVersion: POSTGRES_13
   ```

### Azure Deployment

1. **Container Apps**
   ```yaml
   resources:
     containerApps:
       type: Microsoft.App/containerApps
       properties:
         template:
           containers:
             - image: ${REGISTRY}/app:latest
   ```

2. **Azure Database**
   ```yaml
   resources:
     database:
       type: Microsoft.DBforPostgreSQL/flexibleServers
       properties:
         version: '13'
   ```

## CI/CD Integration

### GitHub Actions
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build and Deploy
        env:
          PLATFORM: ${{ secrets.DEPLOY_PLATFORM }}
        run: |
          case $PLATFORM in
            "aws") ./deploy/aws.sh ;;
            "gcp") ./deploy/gcp.sh ;;
            "azure") ./deploy/azure.sh ;;
          esac
```

### GitLab CI
```yaml
stages:
  - build
  - test
  - deploy

deploy:
  stage: deploy
  script:
    - case $DEPLOY_PLATFORM in
        "aws") ./deploy/aws.sh ;;
        "gcp") ./deploy/gcp.sh ;;
        "azure") ./deploy/azure.sh ;;
      esac
```

## Monitoring Setup

### Container Monitoring
1. **Docker Stats**
   ```bash
   docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
   ```

2. **Prometheus + Grafana**
   ```yaml
   services:
     prometheus:
       image: prom/prometheus
       volumes:
         - ./prometheus.yml:/etc/prometheus/prometheus.yml
     
     grafana:
       image: grafana/grafana
       depends_on:
         - prometheus
   ```

### Cloud Monitoring

1. **AWS CloudWatch**
   ```yaml
   Resources:
     Monitoring:
       Type: AWS::CloudWatch::Dashboard
       Properties:
         DashboardName: AppMetrics
   ```

2. **Google Cloud Monitoring**
   ```yaml
   monitoring:
     metrics:
       - name: custom.googleapis.com/app/requests
         type: custom.googleapis.com/app
   ```

3. **Azure Monitor**
   ```yaml
   resources:
     monitoring:
       type: Microsoft.Monitor/components
       properties:
         Application_Type: web
   ```

## Backup Strategies

### Container Backups
1. **Volume Backups**
   ```bash
   docker run --rm \
     --volumes-from app \
     -v $(pwd):/backup \
     alpine tar cvf /backup/backup.tar /app/data
   ```

2. **Database Backups**
   ```yaml
   services:
     backup:
       image: postgres:latest
       volumes:
         - db-data:/source
         - ./backups:/backup
       command: |
         pg_dump -U postgres -d mydb > /backup/dump.sql
   ```

### Cloud Backups

1. **AWS Backup**
   ```yaml
   Resources:
     BackupVault:
       Type: AWS::Backup::BackupVault
     BackupPlan:
       Type: AWS::Backup::BackupPlan
   ```

2. **GCP Backup**
   ```yaml
   resources:
     backup:
       type: gcp-types/sqladmin-v1beta4:backupRuns
       properties:
         instance: ${DATABASE_INSTANCE}
   ```

3. **Azure Backup**
   ```yaml
   resources:
     backup:
       type: Microsoft.RecoveryServices/vaults
       properties:
         sku:
           name: RS0
           tier: Standard
   ```
