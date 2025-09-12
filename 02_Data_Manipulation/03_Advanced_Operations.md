# Advanced Data Manipulation

## Transaction Management

### 1. Basic Transaction
```sql
-- Start transaction
BEGIN TRANSACTION;

-- Perform operations
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Commit if successful
COMMIT;

-- Or rollback if error
ROLLBACK;
```

### 2. Transaction Isolation Levels
```sql
-- Set isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Different levels
READ UNCOMMITTED    -- Dirty reads possible
READ COMMITTED      -- No dirty reads
REPEATABLE READ    -- Consistent reads
SERIALIZABLE       -- Highest isolation
```

## Batch Processing

### 1. Bulk Insert
```sql
-- Insert multiple rows
INSERT INTO employees (first_name, last_name, salary)
VALUES 
    ('John', 'Doe', 50000),
    ('Jane', 'Smith', 60000),
    ('Bob', 'Johnson', 55000);

-- Insert from select
INSERT INTO employee_archive
SELECT * FROM employees
WHERE termination_date IS NOT NULL;
```

### 2. Batch Update
```sql
-- Update multiple rows
UPDATE employees
SET salary = 
    CASE department
        WHEN 'IT' THEN salary * 1.1
        WHEN 'HR' THEN salary * 1.05
        ELSE salary
    END;
```

## Advanced Queries

### 1. Window Functions
```sql
-- Row number
SELECT 
    department,
    first_name,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) as salary_rank
FROM employees;

-- Running totals
SELECT 
    department,
    salary,
    SUM(salary) OVER (
        PARTITION BY department 
        ORDER BY salary
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total
FROM employees;
```

### 2. Pivot Tables
```sql
-- Create pivot table
SELECT *
FROM (
    SELECT department, job_title, salary
    FROM employees
) AS source
PIVOT (
    AVG(salary)
    FOR department IN ('IT', 'HR', 'Finance')
) AS pivot_table;
```

### 3. Dynamic SQL
```sql
-- Basic dynamic SQL
DECLARE @sql NVARCHAR(MAX);
DECLARE @department VARCHAR(50) = 'IT';

SET @sql = 'SELECT * FROM employees WHERE department = @dept';
EXEC sp_executesql @sql, N'@dept VARCHAR(50)', @department;
```

## Error Handling

### 1. Try-Catch Blocks
```sql
BEGIN TRY
    BEGIN TRANSACTION;
        INSERT INTO employees (first_name, last_name)
        VALUES ('John', 'Doe');
        
        -- Intentional error
        UPDATE nonexistent_table SET col = 1;
    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    
    SELECT 
        ERROR_NUMBER() as ErrorNumber,
        ERROR_MESSAGE() as ErrorMessage;
END CATCH;
```

### 2. Custom Error Handling
```sql
-- Raise custom error
CREATE PROCEDURE insert_employee
    @first_name VARCHAR(50),
    @last_name VARCHAR(50)
AS
BEGIN
    IF LEN(@first_name) < 2
    BEGIN
        RAISERROR('First name must be at least 2 characters', 16, 1);
        RETURN;
    END
    
    INSERT INTO employees (first_name, last_name)
    VALUES (@first_name, @last_name);
END;
```

## Practice Problems

### 1. Transaction Management
```sql
-- Implement a bank transfer with error handling
CREATE PROCEDURE transfer_money(
    @from_account INT,
    @to_account INT,
    @amount DECIMAL(10,2)
)
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;
            -- Check sufficient balance
            IF EXISTS (
                SELECT 1 FROM accounts 
                WHERE account_id = @from_account 
                AND balance >= @amount
            )
            BEGIN
                -- Perform transfer
                UPDATE accounts 
                SET balance = balance - @amount 
                WHERE account_id = @from_account;
                
                UPDATE accounts 
                SET balance = balance + @amount 
                WHERE account_id = @to_account;
                
                COMMIT;
            END
            ELSE
            BEGIN
                RAISERROR('Insufficient funds', 16, 1);
                ROLLBACK;
            END
    END TRY
    BEGIN CATCH
        ROLLBACK;
        THROW;
    END CATCH
END;
```

### 2. Window Functions
```sql
-- Calculate running totals and moving averages
SELECT 
    department,
    employee_id,
    salary,
    SUM(salary) OVER (
        PARTITION BY department 
        ORDER BY hire_date
    ) as running_total,
    AVG(salary) OVER (
        PARTITION BY department 
        ORDER BY hire_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3_months
FROM employees;
```

## Interview Questions

1. **Explain ACID properties in transactions**
   - Atomicity: All or nothing
   - Consistency: Valid state maintained
   - Isolation: Transactions independent
   - Durability: Permanent changes

2. **What are the benefits of batch processing?**
   - Improved performance
   - Reduced network traffic
   - Better resource utilization
   - Easier error handling

3. **How do you handle deadlocks?**
   - Use consistent order of operations
   - Keep transactions short
   - Set deadlock priority
   - Implement retry logic
