# SQL Subqueries

## Types of Subqueries

### 1. Single-Row Subqueries
```sql
-- Basic single-row subquery
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);

-- Nested single-row subquery
SELECT first_name, last_name, salary
FROM employees
WHERE department_id = (
    SELECT department_id
    FROM departments
    WHERE department_name = (
        SELECT 'IT' FROM dual
    )
);
```

### 2. Multiple-Row Subqueries
```sql
-- Using IN operator
SELECT first_name, last_name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location_id = 1700
);

-- Using ANY operator
SELECT first_name, last_name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department_id = 20
);

-- Using ALL operator
SELECT first_name, last_name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department_id = 30
);
```

### 3. Correlated Subqueries
```sql
-- Basic correlated subquery
SELECT first_name, last_name, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e1.department_id = e2.department_id
);

-- Finding highest paid in each department
SELECT first_name, last_name, salary, department_id
FROM employees e1
WHERE salary = (
    SELECT MAX(salary)
    FROM employees e2
    WHERE e1.department_id = e2.department_id
);
```

## Advanced Subquery Techniques

### 1. Subqueries in SELECT Clause
```sql
SELECT 
    department_id,
    department_name,
    (SELECT COUNT(*) 
     FROM employees e 
     WHERE e.department_id = d.department_id) as emp_count
FROM departments d;
```

### 2. Subqueries in FROM Clause
```sql
SELECT 
    dept_summary.department_name,
    dept_summary.avg_salary
FROM (
    SELECT 
        d.department_name,
        AVG(e.salary) as avg_salary
    FROM departments d
    JOIN employees e ON d.department_id = e.department_id
    GROUP BY d.department_name
) dept_summary
WHERE dept_summary.avg_salary > 50000;
```

### 3. Subqueries in HAVING Clause
```sql
SELECT 
    department_id,
    AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
);
```

### 4. EXISTS and NOT EXISTS
```sql
-- Find departments with employees
SELECT department_name
FROM departments d
WHERE EXISTS (
    SELECT 1 
    FROM employees e
    WHERE e.department_id = d.department_id
);

-- Find departments with no employees
SELECT department_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1 
    FROM employees e
    WHERE e.department_id = d.department_id
);
```

## Common Use Cases

### 1. Data Analysis
```sql
-- Identify above-average salaries by department
SELECT 
    e.first_name,
    e.last_name,
    e.salary,
    d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

### 2. Data Filtering
```sql
-- Find employees who haven't made any sales
SELECT first_name, last_name
FROM employees
WHERE employee_id NOT IN (
    SELECT DISTINCT sales_rep_id
    FROM orders
    WHERE sales_rep_id IS NOT NULL
);
```

### 3. Calculations
```sql
-- Calculate percentage of total salary
SELECT 
    first_name,
    last_name,
    salary,
    ROUND(
        (salary / (SELECT SUM(salary) FROM employees)) * 100,
        2
    ) as salary_percentage
FROM employees;
```

## Practice Problems

### Problem 1: Employee Rankings
```sql
-- Rank employees by salary within their department
SELECT 
    first_name,
    last_name,
    salary,
    department_id,
    (SELECT COUNT(DISTINCT salary)
     FROM employees e2
     WHERE e2.department_id = e1.department_id
     AND e2.salary > e1.salary) + 1 as salary_rank
FROM employees e1
ORDER BY department_id, salary_rank;
```

### Problem 2: Complex Filtering
```sql
-- Find employees who earn more than their manager
SELECT 
    e.first_name,
    e.last_name,
    e.salary as employee_salary,
    (SELECT salary 
     FROM employees m 
     WHERE m.employee_id = e.manager_id) as manager_salary
FROM employees e
WHERE e.salary > (
    SELECT salary
    FROM employees m
    WHERE m.employee_id = e.manager_id
);
```

## Interview Questions

1. **What is the difference between correlated and non-correlated subqueries?**
   - Correlated references outer query
   - Non-correlated runs independently
   - Performance implications
   - Use cases for each

2. **When should you use EXISTS vs IN?**
   - EXISTS for large result sets
   - IN for small lists
   - Performance considerations
   - NULL handling differences

3. **How do you optimize subqueries?**
   - Consider joins instead
   - Minimize correlated subqueries
   - Use proper indexes
   - Avoid unnecessary subqueries
