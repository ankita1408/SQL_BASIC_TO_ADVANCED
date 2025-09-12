# Advanced SQL Concepts

## Window Functions

### ROW_NUMBER
```sql
SELECT 
    first_name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as salary_rank
FROM employees;
```

### RANK and DENSE_RANK
```sql
SELECT 
    first_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) as salary_rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank
FROM employees;
```

### LAG and LEAD
```sql
SELECT 
    first_name,
    salary,
    LAG(salary) OVER (ORDER BY salary) as prev_salary,
    LEAD(salary) OVER (ORDER BY salary) as next_salary
FROM employees;
```

## Views and Indexes

### Creating Views
```sql
-- Simple View
CREATE VIEW employee_details AS
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- Materialized View (in supported databases)
CREATE MATERIALIZED VIEW dept_summary AS
SELECT 
    department_name,
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
GROUP BY department_name;
```

### Indexes
```sql
-- Create index
CREATE INDEX idx_employee_salary ON employees(salary);

-- Create composite index
CREATE INDEX idx_emp_dept ON employees(dept_id, hire_date);
```

## Common Table Expressions (CTE)
```sql
WITH avg_salary_by_dept AS (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT e.first_name, e.salary, a.avg_salary
FROM employees e
JOIN avg_salary_by_dept a ON e.dept_id = a.dept_id;
```

## Practice Problems

### Complex Queries
1. Calculate running totals
2. Find median salary
3. Implement row version history

### Performance Optimization
1. Index usage analysis
2. Query execution plans
3. Performance tuning techniques

## Interview Questions
1. When to use window functions vs GROUP BY
2. Advantages and disadvantages of indexes
3. Best practices for view creation
