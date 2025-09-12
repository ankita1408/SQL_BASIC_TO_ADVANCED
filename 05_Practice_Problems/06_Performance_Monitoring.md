# SQL Performance Monitoring and Troubleshooting

## Performance Analysis Tools

### 1. EXPLAIN ANALYZE
```sql
-- Basic EXPLAIN
EXPLAIN
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE salary > 50000;

-- Detailed EXPLAIN ANALYZE
EXPLAIN ANALYZE
SELECT 
    d.department_name,
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_name;

/* Example Output Analysis:
-> Hash Join (cost=2.70 rows=10)
   -> Table scan on d (cost=0.85 rows=10)
   -> Hash
      -> Table scan on e (cost=1.70 rows=100)
*/
```

### 2. Query Profiling
```sql
-- Enable profiling
SET profiling = 1;

-- Run your query
SELECT 
    d.department_name,
    COUNT(*) as emp_count
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_name;

-- View profile
SHOW PROFILES;

-- Detailed profile
SHOW PROFILE FOR QUERY 1;

-- CPU and memory profile
SHOW PROFILE CPU, MEMORY FOR QUERY 1;
```

### 3. System Statistics
```sql
-- Table statistics
ANALYZE TABLE employees, departments;

-- Index statistics
SHOW INDEX FROM employees;

-- Buffer pool statistics
SHOW STATUS LIKE 'Innodb_buffer_pool%';

-- Query cache statistics
SHOW STATUS LIKE 'Qcache%';
```

## Common Performance Issues and Solutions

### 1. Slow Queries
```sql
-- Problem: Full table scan
SELECT *
FROM employees
WHERE salary > 50000;

-- Solution: Add index
CREATE INDEX idx_salary ON employees(salary);

-- Problem: Inefficient JOIN
SELECT *
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_date >= '2025-01-01';

-- Solution: Pre-filter before JOIN
SELECT *
FROM (
    SELECT *
    FROM orders
    WHERE order_date >= '2025-01-01'
) o
JOIN order_items oi ON o.order_id = oi.order_id;
```

### 2. Memory Issues
```sql
-- Monitor memory usage
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_pages%';

-- Optimize memory settings
SET GLOBAL innodb_buffer_pool_size = 4294967296; -- 4GB

-- Check temporary tables usage
SHOW STATUS LIKE 'Created_tmp%';

-- Reduce memory usage in queries
SELECT 
    department_id,
    COUNT(*) -- Instead of SELECT *
FROM employees
GROUP BY department_id;
```

### 3. Deadlocks
```sql
-- Monitor deadlocks
SHOW ENGINE INNODB STATUS;

-- Prevent deadlocks
BEGIN;
    -- Access tables in consistent order
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Use row-level locking
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
```

## Real-World Optimization Scenarios

### 1. E-commerce Order Processing
```sql
-- Original query (slow)
SELECT 
    c.customer_name,
    p.product_name,
    COUNT(*) as purchase_count,
    SUM(oi.quantity * oi.unit_price) as total_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
GROUP BY c.customer_name, p.product_name;

-- Optimized query
WITH CustomerOrders AS (
    SELECT 
        customer_id,
        COUNT(*) as order_count,
        SUM(total_amount) as total_spent
    FROM orders
    WHERE order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
    GROUP BY customer_id
),
ProductSales AS (
    SELECT 
        oi.product_id,
        COUNT(*) as sales_count,
        SUM(oi.quantity * oi.unit_price) as product_revenue
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.order_id
    WHERE o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
    GROUP BY oi.product_id
)
SELECT 
    c.customer_name,
    p.product_name,
    co.order_count,
    ps.product_revenue
FROM CustomerOrders co
JOIN customers c ON co.customer_id = c.customer_id
JOIN ProductSales ps ON true
JOIN products p ON ps.product_id = p.product_id;
```

### 2. Real-time Analytics Dashboard
```sql
-- Original query (heavy on resources)
SELECT 
    DATE_FORMAT(event_time, '%Y-%m-%d %H:00:00') as hour,
    event_type,
    COUNT(*) as event_count,
    COUNT(DISTINCT user_id) as unique_users,
    AVG(duration) as avg_duration
FROM user_events
WHERE event_time >= DATE_SUB(NOW(), INTERVAL 24 HOUR)
GROUP BY 
    DATE_FORMAT(event_time, '%Y-%m-%d %H:00:00'),
    event_type;

-- Optimized approach
-- 1. Create summary table
CREATE TABLE hourly_event_stats (
    hour_start DATETIME,
    event_type VARCHAR(50),
    event_count INT,
    unique_users INT,
    avg_duration DECIMAL(10,2),
    PRIMARY KEY (hour_start, event_type)
);

-- 2. Create procedure for regular updates
DELIMITER //
CREATE PROCEDURE update_hourly_stats()
BEGIN
    INSERT INTO hourly_event_stats
    SELECT 
        DATE_FORMAT(event_time, '%Y-%m-%d %H:00:00'),
        event_type,
        COUNT(*),
        COUNT(DISTINCT user_id),
        AVG(duration)
    FROM user_events
    WHERE event_time >= DATE_SUB(NOW(), INTERVAL 1 HOUR)
    GROUP BY 1, 2
    ON DUPLICATE KEY UPDATE
        event_count = VALUES(event_count),
        unique_users = VALUES(unique_users),
        avg_duration = VALUES(avg_duration);
END //
DELIMITER ;

-- 3. Schedule procedure
EVENT add_hourly_stats
    ON SCHEDULE EVERY 1 HOUR
    DO CALL update_hourly_stats();

-- 4. Query summary table (fast)
SELECT *
FROM hourly_event_stats
WHERE hour_start >= DATE_SUB(NOW(), INTERVAL 24 HOUR);
```

### 3. Large Data Migration
```sql
-- Instead of single large transaction
INSERT INTO new_table
SELECT * FROM old_table;

-- Use batched approach
DELIMITER //
CREATE PROCEDURE migrate_data_in_batches(
    batch_size INT,
    start_id INT,
    end_id INT
)
BEGIN
    DECLARE current_id INT;
    SET current_id = start_id;
    
    WHILE current_id <= end_id DO
        INSERT INTO new_table
        SELECT *
        FROM old_table
        WHERE id BETWEEN current_id AND current_id + batch_size - 1;
        
        SET current_id = current_id + batch_size;
        
        -- Add delay to reduce server load
        DO SLEEP(0.1);
    END WHILE;
END //
DELIMITER ;

-- Execute migration in batches
CALL migrate_data_in_batches(1000, 1, 1000000);
```

## Performance Monitoring Best Practices

1. **Regular Monitoring**
   - Check slow query log
   - Monitor server resources
   - Track query execution times
   - Review index usage

2. **Query Optimization**
   - Use EXPLAIN ANALYZE
   - Review execution plans
   - Optimize JOIN orders
   - Use appropriate indexes

3. **System Configuration**
   - Tune buffer pool size
   - Optimize temporary tables
   - Configure query cache
   - Set appropriate timeouts

4. **Maintenance Tasks**
   - Regular ANALYZE TABLE
   - Update statistics
   - Monitor fragmentation
   - Archive old data
