# SQL Fundamentals

## Database Basics

### What is a Database?
A database is an organized collection of structured information or data, typically stored electronically in a computer system.

### RDBMS Concepts
- Tables (Relations)
- Records (Tuples)
- Fields (Attributes)
- Keys (Primary and Foreign)
- Relationships

## SQL Commands

### 1. DDL (Data Definition Language)
```sql
-- Create a table
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    salary DECIMAL(10,2)
);

-- Alter table structure
ALTER TABLE employees ADD COLUMN department VARCHAR(50);

-- Drop table
DROP TABLE employees;
```

### 2. Basic Queries
```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary FROM employees;

-- Where clause
SELECT * FROM employees WHERE salary > 50000;

-- Order by
SELECT * FROM employees ORDER BY salary DESC;

-- Distinct values
SELECT DISTINCT department FROM employees;
```

## Practice Problems

### Problem 1: Basic Queries
```sql
-- Find all employees in IT department
SELECT * FROM employees WHERE department = 'IT';

-- Find employees with salary greater than average
SELECT * FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Problem 2: NULL Values
```sql
-- Find employees with no department
SELECT * FROM employees WHERE department IS NULL;

-- Count employees by department
SELECT department, COUNT(*) as emp_count 
FROM employees 
GROUP BY department;
```

## Interview Questions

1. What is the difference between DELETE and TRUNCATE?
2. Explain the difference between PRIMARY KEY and UNIQUE constraint
3. What is the purpose of the DISTINCT keyword?

## Exercises
1. Create a table for storing customer information
2. Write queries to insert and retrieve data
3. Practice filtering and sorting data
