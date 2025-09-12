# Database Administration and Management

## Database Installation and Configuration

### 1. Installation Best Practices
```sql
-- Directory structure
/data           -- Database files
/logs           -- Transaction logs
/backup         -- Backup files
/archive        -- Archive logs

-- Configuration settings
SET GLOBAL max_connections = 200;
SET GLOBAL innodb_buffer_pool_size = 4G;
SET GLOBAL query_cache_size = 64M;
```

### 2. Instance Configuration
```sql
-- Memory configuration
innodb_buffer_pool_size = 70% of total RAM
innodb_log_file_size = 256M
key_buffer_size = 256M

-- Connection settings
max_connections = 500
thread_cache_size = 100
```

## Backup and Recovery

### 1. Backup Types
```sql
-- Full backup
mysqldump -u root -p --all-databases > full_backup.sql

-- Differential backup
mysqldump -u root -p --all-databases 
    --where="modified_date >= '2025-01-01'" > diff_backup.sql

-- Transaction log backup
BACKUP LOG DatabaseName 
TO DISK = 'C:\Backup\LogBackup.trn';
```

### 2. Recovery Procedures
```sql
-- Point-in-time recovery
RESTORE DATABASE YourDatabase
FROM DISK = 'C:\Backup\FullBackup.bak'
WITH NORECOVERY;

RESTORE LOG YourDatabase
FROM DISK = 'C:\Backup\LogBackup.trn'
WITH RECOVERY,
STOPAT = '2025-09-12 14:00:00';
```

## Security Management

### 1. User Management
```sql
-- Create user
CREATE USER 'username'@'localhost' 
IDENTIFIED BY 'password';

-- Grant privileges
GRANT SELECT, INSERT, UPDATE 
ON database_name.* 
TO 'username'@'localhost';

-- Role-based access
CREATE ROLE 'app_read';
GRANT SELECT ON app_db.* TO 'app_read';
GRANT 'app_read' TO 'username'@'localhost';
```

### 2. Security Audit
```sql
-- Enable audit logging
SET GLOBAL audit_log_file = 'audit.log';
SET GLOBAL audit_log_policy = 'ALL';

-- Monitor login attempts
SELECT * FROM mysql.general_log
WHERE command_type = 'Connect'
ORDER BY event_time DESC;
```

## Performance Monitoring

### 1. System Statistics
```sql
-- Monitor system metrics
SELECT * FROM sys.host_summary;

-- Query performance
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY sum_timer_wait DESC;
```

### 2. Resource Usage
```sql
-- Memory usage
SELECT * FROM sys.memory_by_host_by_current_bytes;

-- Disk I/O
SELECT * FROM sys.io_global_by_file_by_bytes;
```

## Maintenance Tasks

### 1. Database Maintenance
```sql
-- Rebuild indexes
ALTER INDEX ALL ON TableName REBUILD;

-- Update statistics
UPDATE STATISTICS TableName;

-- Check database integrity
DBCC CHECKDB (DatabaseName);
```

### 2. Space Management
```sql
-- Monitor space usage
SELECT 
    name,
    size/128.0 as size_in_mb,
    size/128.0 - CAST(FILEPROPERTY(name, 'SpaceUsed') AS INT)/128.0 AS free_space_mb
FROM sys.database_files;

-- Shrink database
DBCC SHRINKDATABASE (DatabaseName);
```

## High Availability and Replication

### 1. Replication Setup
```sql
-- Configure master
GRANT REPLICATION SLAVE ON *.* 
TO 'replica_user'@'replica_host';

-- Configure replica
CHANGE MASTER TO
    MASTER_HOST='master_host',
    MASTER_USER='replica_user',
    MASTER_PASSWORD='password';

START SLAVE;
```

### 2. Monitoring Replication
```sql
-- Check replica status
SHOW SLAVE STATUS\G

-- Monitor lag
SELECT 
    replica_host,
    seconds_behind_master
FROM performance_schema.replication_connection_status;
```

## Practice Scenarios

### Scenario 1: Database Migration
```sql
-- Steps for migration:
1. Backup source database
BACKUP DATABASE SourceDB 
TO DISK = 'C:\Backup\SourceDB.bak'
WITH CHECKSUM;

2. Verify backup
RESTORE VERIFYONLY 
FROM DISK = 'C:\Backup\SourceDB.bak';

3. Restore to target
RESTORE DATABASE TargetDB 
FROM DISK = 'C:\Backup\SourceDB.bak'
WITH MOVE 'SourceDB' TO 'D:\Data\TargetDB.mdf',
     MOVE 'SourceDB_log' TO 'D:\Log\TargetDB_log.ldf';
```

### Scenario 2: Performance Troubleshooting
```sql
-- Identify slow queries
SELECT 
    query_id,
    db_name,
    exec_count,
    total_latency,
    avg_latency,
    rows_sent_avg
FROM sys.statements_with_runtimes_in_95th_percentile;

-- Check missing indexes
SELECT * FROM sys.dm_db_missing_index_details;
```

## Common DBA Tasks

### 1. Daily Checks
```sql
-- Check backup status
SELECT 
    database_name,
    backup_start_date,
    backup_finish_date,
    backup_type
FROM msdb.dbo.backupset
WHERE backup_finish_date >= DATEADD(day, -1, GETDATE());

-- Monitor error logs
sp_readerrorlog;
```

### 2. Weekly Tasks
```sql
-- Index maintenance
SELECT 
    object_name(id) as table_name,
    name as index_name,
    avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats
    (DB_ID(), NULL, NULL, NULL, NULL);

-- Statistics update
sp_updatestats;
```

## Interview Questions

1. **Explain backup strategies**
   - Full vs differential vs incremental
   - Recovery point objective (RPO)
   - Recovery time objective (RTO)
   - Backup verification procedures

2. **How do you handle performance issues?**
   - Monitoring tools
   - Query optimization
   - Resource management
   - Hardware considerations

3. **Describe high availability solutions**
   - Replication
   - Clustering
   - Always On
   - Failover procedures

4. **Security best practices**
   - Authentication methods
   - Authorization schemes
   - Encryption options
   - Audit procedures
