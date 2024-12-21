
# Monitoring and Troubleshooting Guide

## Monitoring Solutions

### 1. Self-Hosted Solutions

#### Prometheus + Grafana

```yaml

# docker-compose.monitoring.yml

version: '3.8'
services:
  prometheus:
    image: prom/prometheus
    volumes:
        -$2./prometheus.yml:/etc/prometheus/prometheus.yml

    ports:
        -$2"9090:9090"

  grafana:
    image: grafana/grafana
    depends_on:
        -$2prometheus

    ports:
        -$2"3000:3000"

```text

#### ELK Stack

```yaml

# docker-compose.elk.yml

version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.3

  logstash:
    image: docker.elastic.co/logstash/logstash:7.9.3

  kibana:
    image: docker.elastic.co/kibana/kibana:7.9.3

```text

### 2. Cloud Solutions

#### AWS CloudWatch

```yaml
Resources:
  MonitoringDashboard:
    Type: AWS::CloudWatch::Dashboard
    Properties:
      DashboardName: AppMetrics

```text

#### Google Cloud Monitoring

```yaml
monitoring:
  metrics:
      -$2name: custom.googleapis.com/app/requests

      type: custom.googleapis.com/app

```text

## Database Troubleshooting

### Connection Issues


1. **Common Problems**

    -$2Connection timeouts
    -$2Authentication failures
    -$2SSL/TLS issues
    -$2Port conflicts


1. **Diagnostic Steps**

```bash

# Check database status

systemctl status postgresql

# Check connection

pg_isready -h localhost -p 5432

# Check logs

tail -f /var/log/postgresql/postgresql-13-main.log

```text

### Performance Issues


1. **Slow Queries**

```sql
-- Find slow queries
SELECT pid, now() - pg_stat_activity.query_start AS duration,
        query, state
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;

```text

1. **Index Problems**

```sql
-- Missing indexes
SELECT schemaname, tablename,
        round(heap_blks_hit::numeric/(heap_blks_hit + heap_blks_read), 3) AS hit_ratio
FROM pg_statio_user_tables
WHERE heap_blks_hit + heap_blks_read > 0
ORDER BY hit_ratio ASC;

```text

## Cloud Platform Issues

### AWS Troubleshooting


1. **ECS Issues**

```bash

# Check service status

aws ecs describe-services \
    --cluster your-cluster \
    --services your-service

# View logs

aws logs get-log-events \
    --log-group-name /ecs/your-service \
    --log-stream-name your-stream

```text

1. **RDS Problems**

```bash

# Check instance status

aws rds describe-db-instances \
    --db-instance-identifier your-instance

# View logs

aws rds download-db-log-file-portion \
    --db-instance-identifier your-instance \
    --log-file-name error/postgresql.log

```text

### GCP Troubleshooting


1. **Cloud Run Issues**

```bash

# View service status

gcloud run services describe your-service \
    --platform managed \
    --region your-region

# Check logs

gcloud logging read "resource.type=cloud_run_revision" \
    --project your-project

```text

1. **Cloud SQL Problems**

```bash

# Check instance status

gcloud sql instances describe your-instance

# View logs

gcloud sql instances list-logs your-instance

```text

### Azure Troubleshooting


1. **Container Apps**

```bash

# Check app status

az containerapp show \
    --name your-app \
    --resource-group your-group

# View logs

az containerapp logs show \
    --name your-app \
    --resource-group your-group

```text

1. **Azure Database**

```bash

# Check server status

az postgres server show \
    --name your-server \
    --resource-group your-group

# View logs

az postgres server-logs list \
    --name your-server \
    --resource-group your-group

```text

## Network Issues

### DNS Problems

```bash

# Check DNS resolution

dig your-domain.com

# Trace DNS path

traceroute your-domain.com

# Check DNS propagation

dig +trace your-domain.com

```text

### SSL/TLS Issues

```bash

# Test SSL certificate

openssl s_client -connect your-domain.com:443

# Check certificate expiry

echo | openssl s_client -servername your-domain.com \
    -connect your-domain.com:443 2>/dev/null | \
    openssl x509 -noout -dates

```text

## Application Issues

### Memory Problems

```bash

# Check memory usage

free -h

# Monitor process memory

ps aux --sort=-%mem | head

# Check swap usage

vmstat 1

```text

### CPU Issues

```bash

# Check CPU usage

top -b -n 1

# Monitor load average

uptime

# Check process CPU usage

pidstat 1

```text

## Recovery Procedures

### Database Recovery


1. **Backup Restoration**

```bash

# Restore from backup

pg_restore -d dbname backup.dump

# Point-in-time recovery

pg_basebackup -D /var/lib/postgresql/data

```text

1. **Data Repair**

```sql
-- Check table consistency
VACUUM ANALYZE tablename;

-- Repair indexes
REINDEX TABLE tablename;

```text

### Application Recovery


1. **Service Restart**

```bash

# Graceful restart

systemctl restart your-service

# Force restart

systemctl force-restart your-service

```text

1. **Cache Clear**

```bash

# Clear Redis cache

redis-cli FLUSHALL

# Clear application cache

rm -rf /tmp/cache/*

```text

## System Monitoring

```bash

# Real-time monitoring

htop

# IO monitoring

iotop

# Network monitoring

iftop

```text

### Log Analysis

```bash

# Search logs

grep -r "error" /var/log/

# Follow logs

tail -f /var/log/syslog

# Analyze Apache logs

apache2ctl status

```text

## Prevention Measures

### Automated Checks


1. **Health Checks**

```bash

# Database health

pg_isready -h localhost

# Web server health

curl -I <<http://localhost>>

```text

1. **Performance Monitoring**

```bash

# System stats

sar -u 1 3

# Network stats

netstat -tulpn

```text

### Backup Verification

```bash

# Verify backup integrity

pg_verifybackup /path/to/backup

# Test restore

pg_restore --list backup.dump
