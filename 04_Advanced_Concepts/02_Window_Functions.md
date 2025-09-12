# Window Functions and Advanced Analytics

## Basic Window Functions

### 1. ROW_NUMBER, RANK, DENSE_RANK
```sql
-- Basic ranking
SELECT 
    first_name,
    salary,
    department_id,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as row_num,
    RANK() OVER (ORDER BY salary DESC) as rank_num,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank_num
FROM employees;

-- Partitioned ranking
SELECT 
    first_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
    ) as dept_salary_rank
FROM employees;
```

### 2. LAG, LEAD
```sql
-- Previous and next values
SELECT 
    first_name,
    hire_date,
    salary,
    LAG(salary) OVER (ORDER BY hire_date) as prev_salary,
    LEAD(salary) OVER (ORDER BY hire_date) as next_salary
FROM employees;

-- With partitioning
SELECT 
    department_id,
    first_name,
    salary,
    LAG(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary
    ) as prev_dept_salary
FROM employees;
```

### 3. FIRST_VALUE, LAST_VALUE
```sql
-- First and last values in window
SELECT 
    department_id,
    first_name,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
    ) as highest_dept_salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
        RANGE BETWEEN UNBOUNDED PRECEDING 
        AND UNBOUNDED FOLLOWING
    ) as lowest_dept_salary
FROM employees;
```

## Advanced Analytics

### 1. Running Totals and Moving Averages
```sql
-- Running total
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING 
        AND CURRENT ROW
    ) as running_total
FROM orders;

-- Moving average
SELECT 
    order_date,
    amount,
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING 
        AND CURRENT ROW
    ) as moving_avg_3_days
FROM orders;
```

### 2. Percentiles and Distribution
```sql
-- Percentile calculation
SELECT 
    department_id,
    salary,
    PERCENTILE_CONT(0.5) WITHIN GROUP (
        ORDER BY salary
    ) OVER (PARTITION BY department_id) as median_salary,
    NTILE(4) OVER (
        PARTITION BY department_id 
        ORDER BY salary
    ) as salary_quartile
FROM employees;

-- Cumulative distribution
SELECT 
    department_id,
    salary,
    CUME_DIST() OVER (
        PARTITION BY department_id 
        ORDER BY salary
    ) as cumulative_dist,
    PERCENT_RANK() OVER (
        PARTITION BY department_id 
        ORDER BY salary
    ) as percent_rank
FROM employees;
```

### 3. Custom Window Frames
```sql
-- Sliding window calculations
SELECT 
    order_date,
    amount,
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 3 PRECEDING 
        AND 1 FOLLOWING
    ) as centered_avg,
    SUM(amount) OVER (
        ORDER BY order_date
        RANGE BETWEEN INTERVAL '3' DAY PRECEDING 
        AND CURRENT ROW
    ) as rolling_3_day_sum
FROM orders;
```

## Practical Applications

### 1. Time Series Analysis
```sql
-- Month-over-month growth
SELECT 
    date_trunc('month', order_date) as month,
    sum(amount) as monthly_sales,
    LAG(sum(amount)) OVER (ORDER BY date_trunc('month', order_date)) as prev_month_sales,
    (sum(amount) - LAG(sum(amount)) OVER (ORDER BY date_trunc('month', order_date))) /
        NULLIF(LAG(sum(amount)) OVER (ORDER BY date_trunc('month', order_date)), 0) * 100 
        as growth_percentage
FROM orders
GROUP BY date_trunc('month', order_date);
```

### 2. Employee Analytics
```sql
-- Salary bands and rankings
SELECT 
    e.first_name,
    e.salary,
    d.department_name,
    NTILE(5) OVER (ORDER BY salary) as salary_band,
    PERCENT_RANK() OVER (ORDER BY salary) as salary_percentile,
    AVG(salary) OVER (
        PARTITION BY department_id
    ) as dept_avg_salary,
    (salary - AVG(salary) OVER (
        PARTITION BY department_id)
    ) / AVG(salary) OVER (
        PARTITION BY department_id
    ) * 100 as diff_from_dept_avg
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

## Practice Problems

### Problem 1: Sales Analysis
```sql
-- Calculate sales metrics
WITH monthly_sales AS (
    SELECT 
        date_trunc('month', order_date) as month,
        sum(amount) as total_sales
    FROM orders
    GROUP BY date_trunc('month', order_date)
)
SELECT 
    month,
    total_sales,
    LAG(total_sales) OVER (ORDER BY month) as prev_month_sales,
    ROUND(
        (total_sales - LAG(total_sales) OVER (ORDER BY month)) /
        NULLIF(LAG(total_sales) OVER (ORDER BY month), 0) * 100,
        2
    ) as growth_rate,
    AVG(total_sales) OVER (
        ORDER BY month
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3_months
FROM monthly_sales;
```

### Problem 2: Employee Ranking
```sql
-- Complex employee rankings
SELECT 
    e.first_name,
    e.salary,
    d.department_name,
    ROW_NUMBER() OVER (
        PARTITION BY e.department_id 
        ORDER BY e.salary DESC
    ) as dept_salary_rank,
    PERCENT_RANK() OVER (
        PARTITION BY e.department_id 
        ORDER BY e.salary
    ) as dept_salary_percentile,
    FIRST_VALUE(e.salary) OVER (
        PARTITION BY e.department_id 
        ORDER BY e.salary DESC
    ) as dept_highest_salary,
    e.salary - LAG(e.salary) OVER (
        PARTITION BY e.department_id 
        ORDER BY e.salary
    ) as salary_gap_from_prev
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

## Interview Questions

1. **Explain the difference between ROW_NUMBER, RANK, and DENSE_RANK**
   - ROW_NUMBER: Unique sequential numbers
   - RANK: Same rank for ties, gaps in sequence
   - DENSE_RANK: Same rank for ties, no gaps

2. **How do window frames work?**
   - ROWS vs RANGE
   - Frame boundaries
   - Default frames
   - Performance implications

3. **What are practical uses of window functions?**
   - Time series analysis
   - Running totals
   - Ranking and distribution
   - Gap analysis
