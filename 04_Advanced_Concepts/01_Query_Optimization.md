# SQL Query Optimization and Performance Tuning

## Understanding Query Execution

### 1. Query Execution Plan
```sql
-- View execution plan
EXPLAIN ANALYZE
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE salary > 50000;

-- Common execution steps:
-- 1. Table scan
-- 2. Index scan
-- 3. Join operations
-- 4. Filter operations
-- 5. Sort operations
```

### 2. Query Cost Analysis
```sql
-- Analyzing query cost components
EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS)
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 50000;
```

## Optimization Techniques

### 1. Index Optimization
```sql
-- Creating effective indexes
CREATE INDEX idx_employee_dept_salary 
ON employees(department_id, salary);

-- Covering index
CREATE INDEX idx_employee_coverage
ON employees(department_id, salary, first_name, last_name);

-- Partial index
CREATE INDEX idx_high_salary_employees
ON employees(employee_id, salary)
WHERE salary > 50000;
```

### 2. JOIN Optimization
```sql
-- Optimize join order
SELECT /*+ ORDERED */ 
    e.first_name,
    d.department_name,
    l.city
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id;

-- Use proper join types
SELECT e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 50000;
```

### 3. WHERE Clause Optimization
```sql
-- Avoid functions on indexed columns
-- Bad:
SELECT * FROM employees 
WHERE YEAR(hire_date) = 2025;

-- Good:
SELECT * FROM employees 
WHERE hire_date >= '2025-01-01' 
AND hire_date < '2026-01-01';

-- Use proper operators
-- Bad:
SELECT * FROM employees 
WHERE salary + 5000 > 50000;

-- Good:
SELECT * FROM employees 
WHERE salary > 45000;
```

## Performance Monitoring

### 1. Query Statistics
```sql
-- Check query execution time
SET STATISTICS TIME ON;
GO

-- Monitor IO statistics
SET STATISTICS IO ON;
GO

-- Run query
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

### 2. Cache Management
```sql
-- Clear cache (SQL Server)
DBCC DROPCLEANBUFFERS;
DBCC FREEPROCCACHE;

-- Analyze cache hits
SELECT 
    plan_handle,
    execution_count,
    total_worker_time/execution_count as avg_cpu_time
FROM sys.dm_exec_query_stats;
```

## Common Performance Issues and Solutions

### 1. N+1 Query Problem
```sql
-- Bad: Separate query for each department
SELECT department_name
FROM departments;
-- Then for each department:
SELECT COUNT(*) FROM employees WHERE department_id = ?;

-- Good: Single query
SELECT 
    d.department_name,
    COUNT(e.employee_id) as employee_count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;
```

### 2. Improper Indexing
```sql
-- Check missing indexes
SELECT 
    dm_mid.database_id,
    dm_migs.avg_user_impact,
    dm_migs.last_user_seek,
    dm_mid.statement as table_name,
    dm_mid.equality_columns,
    dm_mid.inequality_columns
FROM sys.dm_db_missing_index_details dm_mid
CROSS APPLY sys.dm_db_missing_index_groups_query_stats dm_migs;

-- Create suggested indexes
CREATE INDEX idx_suggested 
ON employees(department_id)
INCLUDE (first_name, last_name, salary);
```

## Best Practices

### 1. Query Writing
```sql
-- Use specific column names
SELECT first_name, last_name FROM employees;  -- Good
SELECT * FROM employees;  -- Bad

-- Avoid SELECT DISTINCT when possible
SELECT DISTINCT department_id FROM employees;  -- Can be expensive
SELECT department_id FROM employees GROUP BY department_id;  -- Better
```

### 2. Index Design
```sql
-- Consider selectivity
CREATE INDEX idx_employee_gender ON employees(gender);  -- Bad (low selectivity)
CREATE INDEX idx_employee_email ON employees(email);    -- Good (high selectivity)

-- Composite index order matters
CREATE INDEX idx_emp_dept_date ON employees(department_id, hire_date);
```

## Practice Problems

### Problem 1: Optimize Complex Query
```sql
-- Original query
SELECT 
    d.department_name,
    COUNT(e.employee_id) as emp_count,
    AVG(e.salary) as avg_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
WHERE e.hire_date >= '2024-01-01'
GROUP BY d.department_name
HAVING AVG(e.salary) > 50000;

-- Optimization steps:
1. Add index for hire_date and salary
2. Consider materialized view
3. Analyze join order
```

### Problem 2: Fix Performance Issues
```sql
-- Identify and fix issues in query
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    (SELECT COUNT(*) 
     FROM employees 
     WHERE department_id = e.department_id) as dept_count
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE YEAR(e.hire_date) = 2025;
```

## Interview Questions

1. **How do you identify slow queries?**
   - Use query execution plans
   - Monitor query statistics
   - Check wait statistics
   - Use profiling tools

2. **Explain index design best practices**
   - Consider column selectivity
   - Choose correct column order
   - Balance between reads and writes
   - Monitor index usage

3. **How do you handle large table optimization?**
   - Partitioning
   - Archiving strategies
   - Proper indexing
   - Query optimization
