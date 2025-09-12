# Views, Stored Procedures, and Functions

## Views

### 1. Basic Views
```sql
-- Simple view
CREATE VIEW employee_details AS
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    l.city
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id;

-- Using the view
SELECT * FROM employee_details
WHERE city = 'New York';
```

### 2. Materialized Views
```sql
-- Create materialized view
CREATE MATERIALIZED VIEW department_summary AS
SELECT 
    d.department_name,
    COUNT(e.employee_id) as emp_count,
    AVG(e.salary) as avg_salary,
    SUM(e.salary) as total_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name
WITH DATA;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW department_summary;
```

### 3. Indexed Views
```sql
-- Create indexed view (SQL Server)
CREATE VIEW sales_summary
WITH SCHEMABINDING
AS
SELECT 
    product_id,
    SUM(quantity) as total_quantity,
    SUM(quantity * unit_price) as total_amount,
    COUNT_BIG(*) as transaction_count
FROM dbo.order_details
GROUP BY product_id;

CREATE UNIQUE CLUSTERED INDEX idx_sales_summary
ON sales_summary(product_id);
```

## Stored Procedures

### 1. Basic Stored Procedure
```sql
-- Create procedure
CREATE PROCEDURE update_employee_salary
    @employee_id INT,
    @new_salary DECIMAL(10,2),
    @raise_percentage DECIMAL(5,2) OUTPUT
AS
BEGIN
    DECLARE @old_salary DECIMAL(10,2);
    
    -- Get current salary
    SELECT @old_salary = salary
    FROM employees
    WHERE employee_id = @employee_id;
    
    -- Calculate raise percentage
    SET @raise_percentage = (@new_salary - @old_salary) / @old_salary * 100;
    
    -- Update salary
    UPDATE employees
    SET salary = @new_salary
    WHERE employee_id = @employee_id;
    
    RETURN 0;
END;

-- Execute procedure
DECLARE @raise DECIMAL(5,2);
EXEC update_employee_salary 123, 55000, @raise OUTPUT;
PRINT 'Raise percentage: ' + CAST(@raise as VARCHAR(10)) + '%';
```

### 2. Stored Procedure with Error Handling
```sql
CREATE PROCEDURE transfer_funds
    @from_account INT,
    @to_account INT,
    @amount DECIMAL(10,2)
AS
BEGIN
    SET NOCOUNT ON;
    
    BEGIN TRY
        BEGIN TRANSACTION;
            -- Check sufficient balance
            IF EXISTS (
                SELECT 1 FROM accounts 
                WHERE account_id = @from_account 
                AND balance >= @amount
            )
            BEGIN
                -- Deduct from source
                UPDATE accounts
                SET balance = balance - @amount
                WHERE account_id = @from_account;
                
                -- Add to destination
                UPDATE accounts
                SET balance = balance + @amount
                WHERE account_id = @to_account;
                
                COMMIT TRANSACTION;
            END
            ELSE
            BEGIN
                THROW 50001, 'Insufficient funds', 1;
            END
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;
            
        DECLARE @ErrorMessage NVARCHAR(4000);
        SET @ErrorMessage = ERROR_MESSAGE();
        THROW 50000, @ErrorMessage, 1;
    END CATCH
END;
```

## Functions

### 1. Scalar Functions
```sql
-- Create scalar function
CREATE FUNCTION calculate_age
(
    @birth_date DATE
)
RETURNS INT
AS
BEGIN
    DECLARE @age INT;
    SET @age = DATEDIFF(YEAR, @birth_date, GETDATE()) -
        CASE
            WHEN (MONTH(@birth_date) > MONTH(GETDATE())) OR
                 (MONTH(@birth_date) = MONTH(GETDATE()) AND 
                  DAY(@birth_date) > DAY(GETDATE()))
            THEN 1
            ELSE 0
        END;
    RETURN @age;
END;

-- Use scalar function
SELECT 
    first_name,
    birth_date,
    dbo.calculate_age(birth_date) as age
FROM employees;
```

### 2. Table-Valued Functions
```sql
-- Create table-valued function
CREATE FUNCTION get_department_employees
(
    @department_id INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.salary
    FROM employees e
    WHERE e.department_id = @department_id
);

-- Use table-valued function
SELECT * FROM get_department_employees(10);
```

### 3. Aggregate Functions
```sql
-- Create aggregate function
CREATE FUNCTION weighted_average
(
    @value DECIMAL(10,2),
    @weight DECIMAL(10,2)
)
RETURNS DECIMAL(10,2)
AS
BEGIN
    RETURN @value * @weight;
END;

-- Use in query
SELECT 
    department_id,
    SUM(dbo.weighted_average(salary, 1.0)) / SUM(1.0) as weighted_avg_salary
FROM employees
GROUP BY department_id;
```

## Best Practices

### 1. View Best Practices
```sql
-- Use appropriate indexes
CREATE VIEW indexed_sales AS
SELECT 
    product_id,
    SUM(quantity) as total_quantity
FROM orders
GROUP BY product_id;

CREATE INDEX idx_sales_product
ON orders(product_id);

-- Consider materialization for complex queries
CREATE MATERIALIZED VIEW complex_summary AS
SELECT /* complex query here */
WITH DATA;
```

### 2. Stored Procedure Best Practices
```sql
-- Parameter validation
CREATE PROCEDURE insert_employee
    @first_name VARCHAR(50),
    @last_name VARCHAR(50),
    @salary DECIMAL(10,2)
AS
BEGIN
    -- Validate parameters
    IF @salary < 0
    BEGIN
        THROW 50000, 'Salary cannot be negative', 1;
        RETURN;
    END
    
    -- Rest of procedure
END;

-- Use proper error handling
BEGIN TRY
    -- Procedure logic
END TRY
BEGIN CATCH
    -- Error handling
END CATCH;
```

## Interview Questions

1. **When should you use views vs materialized views?**
   - Views for real-time data
   - Materialized views for complex calculations
   - Consider refresh frequency
   - Storage vs computation trade-off

2. **Explain stored procedure vs function**
   - Procedures can modify data
   - Functions must return value
   - Functions can be used in queries
   - Transaction handling differences

3. **Best practices for error handling**
   - Use TRY-CATCH blocks
   - Proper transaction management
   - Meaningful error messages
   - Logging considerations
