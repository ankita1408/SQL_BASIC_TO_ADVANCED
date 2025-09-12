# SQL Problem-Solving Patterns and Optimization

## Common SQL Problem Patterns

### 1. Consecutive Numbers Pattern
```sql
-- Pattern: Finding consecutive occurrences
-- Common in: Streak calculations, Sequential analysis

-- Example 1: Find consecutive login days
WITH ConsecutiveDays AS (
    SELECT 
        user_id,
        login_date,
        DATE_SUB(
            login_date,
            INTERVAL ROW_NUMBER() OVER (
                PARTITION BY user_id 
                ORDER BY login_date
            ) DAY
        ) as group_date
    FROM Logins
)
SELECT 
    user_id,
    COUNT(*) as consecutive_days
FROM ConsecutiveDays
GROUP BY user_id, group_date;

-- Example 2: Find seats occupied for 3 consecutive days
WITH ConsecutiveSeats AS (
    SELECT 
        seat_number,
        date,
        seat_number - ROW_NUMBER() OVER (ORDER BY date) as grp
    FROM Cinema
    WHERE occupied = 1
)
SELECT DISTINCT seat_number
FROM ConsecutiveSeats
GROUP BY seat_number, grp
HAVING COUNT(*) >= 3;
```

### 2. Running Totals and Moving Averages
```sql
-- Pattern: Cumulative calculations
-- Common in: Financial analysis, Performance tracking

-- Example 1: Running total of sales
SELECT 
    date,
    amount,
    SUM(amount) OVER (
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total
FROM Sales;

-- Example 2: 7-day moving average
SELECT 
    date,
    amount,
    AVG(amount) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7day
FROM Sales;
```

### 3. Gaps and Islands Pattern
```sql
-- Pattern: Finding continuous ranges or gaps
-- Common in: Schedule analysis, Inventory tracking

-- Example 1: Find gaps in sequence
WITH NumberSequence AS (
    SELECT 
        number,
        LEAD(number) OVER (ORDER BY number) as next_number
    FROM Numbers
)
SELECT 
    number + 1 as gap_start,
    next_number - 1 as gap_end
FROM NumberSequence
WHERE next_number - number > 1;

-- Example 2: Group continuous ranges
WITH RankedDates AS (
    SELECT 
        date,
        DATE_SUB(
            date,
            INTERVAL ROW_NUMBER() OVER (ORDER BY date) DAY
        ) as group_date
    FROM Dates
)
SELECT 
    MIN(date) as range_start,
    MAX(date) as range_end
FROM RankedDates
GROUP BY group_date;
```

### 4. Top-N per Group Pattern
```sql
-- Pattern: Finding top/bottom records per category
-- Common in: Sales analysis, Performance ranking

-- Example 1: Top 3 products per category
WITH RankedProducts AS (
    SELECT 
        category_id,
        product_id,
        sales_amount,
        DENSE_RANK() OVER (
            PARTITION BY category_id 
            ORDER BY sales_amount DESC
        ) as sales_rank
    FROM ProductSales
)
SELECT *
FROM RankedProducts
WHERE sales_rank <= 3;

-- Example 2: Bottom 5 performers by department
WITH EmployeeRanks AS (
    SELECT 
        department_id,
        employee_id,
        performance_score,
        ROW_NUMBER() OVER (
            PARTITION BY department_id 
            ORDER BY performance_score
        ) as performance_rank
    FROM Employees
)
SELECT *
FROM EmployeeRanks
WHERE performance_rank <= 5;
```

### 5. Aggregation and Pivoting Pattern
```sql
-- Pattern: Transforming rows to columns
-- Common in: Reporting, Cross-tabulation

-- Example 1: Pivot monthly sales
SELECT 
    product_id,
    MAX(CASE WHEN month = 1 THEN sales END) as Jan_sales,
    MAX(CASE WHEN month = 2 THEN sales END) as Feb_sales,
    MAX(CASE WHEN month = 3 THEN sales END) as Mar_sales
FROM MonthlySales
GROUP BY product_id;

-- Example 2: Dynamic pivot
SET @sql = NULL;
SELECT GROUP_CONCAT(
    DISTINCT CONCAT(
        'MAX(CASE WHEN month = ',
        month,
        ' THEN sales END) as month_',
        month
    )
) INTO @sql
FROM MonthlySales;

SET @sql = CONCAT('
    SELECT product_id, ', @sql, '
    FROM MonthlySales
    GROUP BY product_id
');

PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

## Optimization Techniques

### 1. Index Optimization
```sql
-- Pattern: Proper index usage
-- Impact: Query performance

-- Example 1: Composite index for range and equality
CREATE INDEX idx_emp_dept_salary 
ON employees(department_id, salary);

-- Query using index
SELECT *
FROM employees
WHERE department_id = 10
AND salary BETWEEN 50000 AND 75000;

-- Example 2: Covering index
CREATE INDEX idx_emp_coverage
ON employees(department_id, salary, first_name, last_name);

-- Query fully covered by index
SELECT first_name, last_name, salary
FROM employees
WHERE department_id = 10;
```

### 2. JOIN Optimization
```sql
-- Pattern: Efficient join strategies
-- Impact: Query performance, Resource usage

-- Example 1: Pre-filtering before join
SELECT 
    c.customer_name,
    o.order_date
FROM (
    SELECT customer_id, order_date
    FROM orders
    WHERE order_date >= '2025-01-01'
) o
JOIN customers c ON o.customer_id = c.customer_id;

-- Example 2: Using EXISTS instead of IN
-- Better for large datasets
SELECT *
FROM orders o
WHERE EXISTS (
    SELECT 1
    FROM customers c
    WHERE c.customer_id = o.customer_id
    AND c.status = 'active'
);
```

### 3. Subquery Optimization
```sql
-- Pattern: Efficient subquery usage
-- Impact: Query performance

-- Example 1: Correlated subquery to JOIN
-- Instead of:
SELECT 
    department_name,
    (SELECT COUNT(*) 
     FROM employees e 
     WHERE e.department_id = d.department_id) as emp_count
FROM departments d;

-- Better:
SELECT 
    d.department_name,
    COUNT(e.employee_id) as emp_count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;

-- Example 2: Using CTEs for readability and performance
WITH DeptStats AS (
    SELECT 
        department_id,
        COUNT(*) as emp_count,
        AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT 
    d.department_name,
    ds.emp_count,
    ds.avg_salary
FROM departments d
JOIN DeptStats ds ON d.department_id = ds.department_id;
```

### 4. Aggregation Optimization
```sql
-- Pattern: Efficient grouping and aggregation
-- Impact: Memory usage, Performance

-- Example 1: Pre-aggregating before join
WITH DeptSummary AS (
    SELECT 
        department_id,
        COUNT(*) as emp_count,
        SUM(salary) as total_salary
    FROM employees
    GROUP BY department_id
)
SELECT 
    d.department_name,
    ds.emp_count,
    ds.total_salary
FROM departments d
JOIN DeptSummary ds ON d.department_id = ds.department_id;

-- Example 2: Using window functions instead of self-joins
SELECT 
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) as dept_avg
FROM employees;
```

## Interview Tips for Optimization

1. **Always Consider Indexes**
   - Primary key columns
   - Foreign key columns
   - Frequently filtered columns
   - Join columns

2. **JOIN Strategies**
   - Choose appropriate join types
   - Consider table sizes
   - Pre-filter when possible
   - Use proper join conditions

3. **Subquery Best Practices**
   - Avoid correlated subqueries
   - Use CTEs for readability
   - Consider JOIN alternatives
   - Proper placement in query

4. **Performance Monitoring**
   - Use EXPLAIN ANALYZE
   - Check execution plans
   - Monitor resource usage
   - Test with realistic data volumes
