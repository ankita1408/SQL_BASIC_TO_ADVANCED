# Data Manipulation in SQL

## DML Operations

### INSERT Operations
```sql
-- Single row insert
INSERT INTO employees (emp_id, first_name, last_name, salary)
VALUES (1, 'John', 'Doe', 50000);

-- Multiple row insert
INSERT INTO employees (emp_id, first_name, last_name, salary)
VALUES 
    (2, 'Jane', 'Smith', 60000),
    (3, 'Bob', 'Johnson', 55000);
```

### UPDATE Operations
```sql
-- Update single record
UPDATE employees 
SET salary = 65000 
WHERE emp_id = 1;

-- Update multiple records
UPDATE employees 
SET salary = salary * 1.1 
WHERE department = 'IT';
```

### DELETE Operations
```sql
-- Delete specific records
DELETE FROM employees WHERE emp_id = 1;

-- Delete all records from a table
DELETE FROM employees;
```

## SQL Functions

### Aggregate Functions
```sql
-- COUNT
SELECT COUNT(*) FROM employees;

-- AVG
SELECT AVG(salary) FROM employees;

-- SUM
SELECT SUM(salary) FROM employees;

-- MIN and MAX
SELECT MIN(salary), MAX(salary) FROM employees;
```

### String Functions
```sql
-- CONCAT
SELECT CONCAT(first_name, ' ', last_name) as full_name FROM employees;

-- UPPER and LOWER
SELECT UPPER(first_name), LOWER(last_name) FROM employees;

-- SUBSTRING
SELECT SUBSTRING(first_name, 1, 3) FROM employees;
```

### Date Functions
```sql
-- Current date
SELECT CURRENT_DATE;

-- Date arithmetic
SELECT DATE_ADD(hire_date, INTERVAL 1 YEAR) FROM employees;

-- Date difference
SELECT DATEDIFF(CURRENT_DATE, hire_date) FROM employees;
```

## Practice Problems

1. Calculate department-wise average salary
2. Find employees hired in the last 6 months
3. Update salaries with different percentage raises based on performance

## LeetCode Problems
1. [#176 Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)
2. [#177 Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/)

## Interview Questions
1. How to handle NULL values in aggregate functions?
2. Difference between DELETE and TRUNCATE
3. How to perform bulk updates efficiently?
