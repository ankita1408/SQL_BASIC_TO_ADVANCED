# Basic SQL Queries and Operations

## SELECT Statement Fundamentals

### Basic SELECT
```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary FROM employees;

-- Select with column alias
SELECT 
    first_name AS fname,
    last_name AS lname,
    salary * 12 AS annual_salary
FROM employees;
```

### WHERE Clause
```sql
-- Basic conditions
SELECT * FROM employees WHERE salary > 50000;

-- Multiple conditions
SELECT * FROM employees 
WHERE salary > 50000 AND department = 'IT';

-- IN operator
SELECT * FROM employees 
WHERE department IN ('IT', 'HR', 'Finance');

-- BETWEEN operator
SELECT * FROM employees 
WHERE salary BETWEEN 40000 AND 60000;

-- LIKE operator
SELECT * FROM employees 
WHERE last_name LIKE 'S%';
```

### ORDER BY
```sql
-- Single column ascending
SELECT * FROM employees ORDER BY salary;

-- Single column descending
SELECT * FROM employees ORDER BY salary DESC;

-- Multiple columns
SELECT * FROM employees 
ORDER BY department ASC, salary DESC;
```

### DISTINCT
```sql
-- Single column
SELECT DISTINCT department FROM employees;

-- Multiple columns
SELECT DISTINCT department, job_title FROM employees;
```

## Advanced Query Elements

### GROUP BY
```sql
-- Basic grouping
SELECT department, COUNT(*) as emp_count
FROM employees
GROUP BY department;

-- Multiple columns
SELECT department, job_title, AVG(salary) as avg_salary
FROM employees
GROUP BY department, job_title;
```

### HAVING
```sql
-- Filter groups
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;

-- Complex conditions
SELECT 
    department, 
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 5 AND AVG(salary) > 50000;
```

## Common Functions

### String Functions
```sql
-- Concatenation
SELECT 
    CONCAT(first_name, ' ', last_name) as full_name,
    UPPER(first_name) as upper_name,
    LOWER(last_name) as lower_name,
    LENGTH(first_name) as name_length
FROM employees;
```

### Date Functions
```sql
-- Current date
SELECT CURRENT_DATE;

-- Date arithmetic
SELECT 
    hire_date,
    DATE_ADD(hire_date, INTERVAL 1 YEAR) as anniversary,
    DATEDIFF(CURRENT_DATE, hire_date) as days_employed
FROM employees;
```

### Numeric Functions
```sql
-- Round numbers
SELECT 
    salary,
    ROUND(salary, 2) as rounded_salary,
    CEIL(salary) as ceiling_salary,
    FLOOR(salary) as floor_salary
FROM employees;
```

## Practice Problems

### Problem 1: Basic Queries
Write queries to:
1. Find all employees in IT department with salary > 60000
2. List departments with more than 3 employees
3. Calculate average salary by department

### Problem 2: Data Analysis
```sql
-- Department statistics
SELECT 
    department,
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;
```

## Interview Questions

1. **What is the difference between WHERE and HAVING?**
   - WHERE filters individual rows
   - HAVING filters groups
   - WHERE is processed before GROUP BY
   - HAVING is processed after GROUP BY

2. **Explain the ORDER BY clause**
   - Sorts query results
   - Can sort by multiple columns
   - ASC is default order
   - Processed last in query execution

3. **How does NULL affect queries?**
   - Cannot use = or != with NULL
   - Must use IS NULL or IS NOT NULL
   - NULL values are excluded from many aggregate functions
   - Special handling in GROUP BY and ORDER BY
