# SQL Commands Deep Dive

## Types of SQL Commands

### 1. DDL (Data Definition Language)

Commands that define and modify database structure:

#### CREATE
```sql
-- Create database
CREATE DATABASE company;

-- Create table
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50) NOT NULL,
    location VARCHAR(100),
    budget DECIMAL(12,2)
);

-- Create index
CREATE INDEX idx_dept_name ON departments(dept_name);
```

#### ALTER
```sql
-- Add column
ALTER TABLE departments ADD COLUMN manager_id INT;

-- Modify column
ALTER TABLE departments MODIFY COLUMN dept_name VARCHAR(100);

-- Add constraint
ALTER TABLE departments 
ADD CONSTRAINT fk_manager 
FOREIGN KEY (manager_id) REFERENCES employees(emp_id);
```

#### DROP
```sql
-- Drop table
DROP TABLE departments;

-- Drop column
ALTER TABLE departments DROP COLUMN manager_id;

-- Drop database
DROP DATABASE company;
```

#### TRUNCATE
```sql
-- Remove all records from table
TRUNCATE TABLE departments;
```

### 2. DML (Data Manipulation Language)

Commands that manipulate data within the database:

#### SELECT
```sql
-- Basic select
SELECT * FROM departments;

-- Conditional select
SELECT dept_name, budget 
FROM departments 
WHERE budget > 100000;

-- Aggregation
SELECT dept_name, SUM(budget) as total_budget
FROM departments
GROUP BY dept_name;
```

#### INSERT
```sql
-- Single row insert
INSERT INTO departments (dept_id, dept_name, location)
VALUES (1, 'IT', 'New York');

-- Multiple row insert
INSERT INTO departments (dept_id, dept_name, location)
VALUES 
    (2, 'HR', 'Chicago'),
    (3, 'Finance', 'Boston');
```

#### UPDATE
```sql
-- Update single record
UPDATE departments 
SET budget = 150000 
WHERE dept_id = 1;

-- Update multiple records
UPDATE departments 
SET budget = budget * 1.1 
WHERE location = 'New York';
```

#### DELETE
```sql
-- Delete specific records
DELETE FROM departments WHERE dept_id = 1;

-- Delete all records
DELETE FROM departments;
```

### 3. DCL (Data Control Language)

Commands that control access to data:

#### GRANT
```sql
-- Grant select permission
GRANT SELECT ON departments TO user1;

-- Grant multiple permissions
GRANT SELECT, INSERT, UPDATE ON departments TO user1;
```

#### REVOKE
```sql
-- Revoke select permission
REVOKE SELECT ON departments FROM user1;

-- Revoke all permissions
REVOKE ALL PRIVILEGES ON departments FROM user1;
```

### 4. TCL (Transaction Control Language)

Commands that manage transactions:

```sql
-- Begin transaction
BEGIN TRANSACTION;

-- Make changes
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Commit if successful
COMMIT;

-- Or rollback if error
ROLLBACK;
```

## Practice Problems

1. Create a complete database schema for an e-commerce system
2. Write queries to handle common data operations
3. Implement transaction management for order processing

## Common Interview Questions

1. **What is the difference between TRUNCATE and DELETE?**
   - TRUNCATE is DDL, DELETE is DML
   - TRUNCATE cannot be rolled back
   - TRUNCATE is faster than DELETE
   - TRUNCATE resets auto-increment values

2. **Explain the importance of COMMIT and ROLLBACK.**
   - Ensures data integrity
   - Allows for error recovery
   - Maintains database consistency
   - Supports concurrent operations

3. **What is the difference between DDL and DML?**
   - DDL defines structure, DML manipulates data
   - DDL changes are immediate, DML can be rolled back
   - DDL affects entire tables/databases, DML affects rows
