# Database Migration Strategies

## Migration Paths

### 1. SQLite to Production Databases

#### SQLite to PostgreSQL
```python
# migrate_sqlite_to_postgres.py
from sqlalchemy import create_engine
import sqlite3
import psycopg2
import pandas as pd

def migrate_tables():
    # Source SQLite connection
    sqlite_conn = sqlite3.connect('dev.db')
    
    # Target PostgreSQL connection
    pg_conn = psycopg2.connect(
        dbname="proddb",
        user="produser",
        password="prodpass",
        host="prodhost"
    )
    
    # Get all tables
    tables = pd.read_sql_query(
        "SELECT name FROM sqlite_master WHERE type='table'",
        sqlite_conn
    )
    
    for table in tables['name']:
        # Read data from SQLite
        df = pd.read_sql_query(f"SELECT * FROM {table}", sqlite_conn)
        
        # Write to PostgreSQL
        df.to_sql(
            table,
            create_engine('postgresql://produser:prodpass@prodhost/proddb'),
            if_exists='replace',
            index=False
        )
```

#### SQLite to MySQL
```python
# migrate_sqlite_to_mysql.py
from sqlalchemy import create_engine
import sqlite3
import mysql.connector
import pandas as pd

def migrate_tables():
    # Source SQLite connection
    sqlite_conn = sqlite3.connect('dev.db')
    
    # Target MySQL connection
    mysql_conn = mysql.connector.connect(
        host="prodhost",
        user="produser",
        password="prodpass",
        database="proddb"
    )
    
    # Get all tables
    tables = pd.read_sql_query(
        "SELECT name FROM sqlite_master WHERE type='table'",
        sqlite_conn
    )
    
    for table in tables['name']:
        df = pd.read_sql_query(f"SELECT * FROM {table}", sqlite_conn)
        df.to_sql(
            table,
            create_engine('mysql://produser:prodpass@prodhost/proddb'),
            if_exists='replace',
            index=False
        )
```

### 2. PostgreSQL Migrations

#### PostgreSQL Version Upgrade
```bash
#!/bin/bash
# postgres_upgrade.sh

OLD_VERSION="14"
NEW_VERSION="15"

# Stop PostgreSQL services
systemctl stop postgresql-${OLD_VERSION}
systemctl stop postgresql-${NEW_VERSION}

# Initialize new database cluster
/usr/pgsql-${NEW_VERSION}/bin/postgresql-${NEW_VERSION}-setup initdb

# Upgrade database
/usr/pgsql-${NEW_VERSION}/bin/pg_upgrade \
  --old-datadir="/var/lib/pgsql/${OLD_VERSION}/data" \
  --new-datadir="/var/lib/pgsql/${NEW_VERSION}/data" \
  --old-bindir="/usr/pgsql-${OLD_VERSION}/bin" \
  --new-bindir="/usr/pgsql-${NEW_VERSION}/bin" \
  --check
```

#### Cross-Server Migration
```bash
#!/bin/bash
# postgres_cross_server.sh

# Variables
SRC_HOST="source-host"
SRC_DB="sourcedb"
SRC_USER="sourceuser"
DEST_HOST="dest-host"
DEST_DB="destdb"
DEST_USER="destuser"

# Create schema-only backup
pg_dump -h ${SRC_HOST} -U ${SRC_USER} -d ${SRC_DB} --schema-only > schema.sql

# Create data-only backup
pg_dump -h ${SRC_HOST} -U ${SRC_USER} -d ${SRC_DB} --data-only > data.sql

# Apply schema to new database
psql -h ${DEST_HOST} -U ${DEST_USER} -d ${DEST_DB} < schema.sql

# Apply data to new database
psql -h ${DEST_HOST} -U ${DEST_USER} -d ${DEST_DB} < data.sql
```

### 3. MySQL Migrations

#### MySQL to PostgreSQL with Schema Conversion
```python
# mysql_to_postgres.py
import pymysql
import psycopg2
from sqlalchemy import create_engine
import pandas as pd

def convert_schema():
    # MySQL connection
    mysql_conn = pymysql.connect(
        host='mysql-host',
        user='mysql-user',
        password='mysql-pass',
        db='mysql-db'
    )
    
    # PostgreSQL connection
    pg_conn = psycopg2.connect(
        dbname="pg-db",
        user="pg-user",
        password="pg-pass",
        host="pg-host"
    )
    
    # Get MySQL schema
    mysql_cursor = mysql_conn.cursor()
    mysql_cursor.execute("SHOW TABLES")
    tables = mysql_cursor.fetchall()
    
    for table in tables:
        table_name = table[0]
        # Get create table statement
        mysql_cursor.execute(f"SHOW CREATE TABLE {table_name}")
        create_stmt = mysql_cursor.fetchone()[1]
        
        # Convert MySQL to PostgreSQL syntax
        pg_create_stmt = convert_to_postgres_syntax(create_stmt)
        
        # Create table in PostgreSQL
        pg_cursor = pg_conn.cursor()
        pg_cursor.execute(pg_create_stmt)
        pg_conn.commit()
```

### 4. MongoDB to SQL Migrations

#### MongoDB to PostgreSQL
```python
# mongo_to_postgres.py
from pymongo import MongoClient
import psycopg2
import json
from sqlalchemy import create_engine
import pandas as pd

def flatten_document(doc, parent_key='', sep='_'):
    items = []
    for k, v in doc.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_document(v, new_key, sep=sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

def migrate_collection(collection_name):
    # MongoDB connection
    mongo_client = MongoClient('mongodb://mongo-host:27017/')
    mongo_db = mongo_client['source-db']
    
    # PostgreSQL connection
    pg_engine = create_engine('postgresql://user:pass@pg-host/target-db')
    
    # Get MongoDB collection
    collection = mongo_db[collection_name]
    
    # Flatten and convert documents
    flat_docs = [flatten_document(doc) for doc in collection.find()]
    
    # Convert to DataFrame and save to PostgreSQL
    df = pd.DataFrame(flat_docs)
    df.to_sql(collection_name, pg_engine, if_exists='replace', index=False)
```

## Migration Verification Tools

### 1. Data Integrity Checker
```python
# verify_migration.py
def verify_data_integrity(source_conn, target_conn, table_name):
    # Check row counts
    src_count = pd.read_sql(f"SELECT COUNT(*) FROM {table_name}", source_conn)
    tgt_count = pd.read_sql(f"SELECT COUNT(*) FROM {table_name}", target_conn)
    
    if src_count.iloc[0,0] != tgt_count.iloc[0,0]:
        raise ValueError(f"Row count mismatch in {table_name}")
    
    # Check data checksums
    src_checksum = pd.read_sql(f"SELECT MD5(CAST((SELECT * FROM {table_name} FOR UPDATE) AS text))", source_conn)
    tgt_checksum = pd.read_sql(f"SELECT MD5(CAST((SELECT * FROM {table_name} FOR UPDATE) AS text))", target_conn)
    
    if src_checksum.iloc[0,0] != tgt_checksum.iloc[0,0]:
        raise ValueError(f"Data checksum mismatch in {table_name}")
```

### 2. Performance Comparison
```python
# compare_performance.py
def compare_query_performance(source_conn, target_conn, test_queries):
    results = []
    
    for query in test_queries:
        # Test source database
        start_time = time.time()
        pd.read_sql(query, source_conn)
        source_time = time.time() - start_time
        
        # Test target database
        start_time = time.time()
        pd.read_sql(query, target_conn)
        target_time = time.time() - start_time
        
        results.append({
            'query': query,
            'source_time': source_time,
            'target_time': target_time,
            'difference': abs(source_time - target_time)
        })
    
    return pd.DataFrame(results)
```

## Rollback Strategies

### 1. Point-in-Time Recovery
```bash
#!/bin/bash
# rollback_to_point.sh

TIMESTAMP="2024-01-01 00:00:00"

# PostgreSQL
pg_restore --target-timestamp="${TIMESTAMP}" -d dbname backup.dump

# MySQL
mysqlbinlog --stop-datetime="${TIMESTAMP}" binlog.* | mysql -u user -p dbname
```

### 2. Full Database Rollback
```bash
#!/bin/bash
# full_rollback.sh

# Create backup of current state
pg_dump dbname > pre_rollback_backup.sql

# Restore previous version
pg_restore -d dbname previous_backup.dump

# Verify restoration
psql -d dbname -c "SELECT COUNT(*) FROM critical_table;"
```
