# SQL Functions and Procedures

## Built-in Functions

### 1. String Functions
```sql
-- Basic string manipulation
SELECT 
    UPPER(first_name) as upper_name,            -- Convert to uppercase
    LOWER(last_name) as lower_name,             -- Convert to lowercase
    LENGTH(first_name) as name_length,          -- String length
    TRIM(first_name) as trimmed_name,           -- Remove leading/trailing spaces
    SUBSTRING(first_name, 1, 3) as name_start,  -- Extract substring
    REPLACE(email, '@', '#') as masked_email,   -- Replace characters
    CONCAT(first_name, ' ', last_name) as full_name  -- Concatenate strings
FROM employees;

-- Advanced string functions
SELECT 
    LEFT(first_name, 3) as name_start,          -- First 3 characters
    RIGHT(last_name, 3) as name_end,            -- Last 3 characters
    REVERSE(first_name) as reversed_name,        -- Reverse string
    POSITION('@' IN email) as at_position       -- Find position of character
FROM employees;
```

### 2. Numeric Functions
```sql
-- Mathematical operations
SELECT 
    ABS(balance) as absolute_value,             -- Absolute value
    ROUND(salary, 2) as rounded_salary,         -- Round to 2 decimals
    CEIL(salary) as ceiling_value,              -- Round up
    FLOOR(salary) as floor_value,               -- Round down
    POWER(salary, 2) as squared_salary,         -- Power
    SQRT(salary) as square_root,                -- Square root
    MOD(salary, 1000) as remainder              -- Modulo
FROM employees;

-- Statistical functions
SELECT 
    AVG(salary) as average_salary,
    SUM(salary) as total_salary,
    MIN(salary) as minimum_salary,
    MAX(salary) as maximum_salary,
    COUNT(*) as employee_count
FROM employees;
```

### 3. Date Functions
```sql
-- Date manipulation
SELECT 
    CURRENT_DATE as today,                      -- Current date
    CURRENT_TIMESTAMP as now,                   -- Current date and time
    DATE_ADD(hire_date, INTERVAL 1 YEAR) as anniversary,  -- Add interval
    DATE_SUB(hire_date, INTERVAL 6 MONTH) as six_months_ago,  -- Subtract interval
    DATEDIFF(CURRENT_DATE, hire_date) as days_employed,   -- Date difference
    YEAR(hire_date) as hire_year,              -- Extract year
    MONTH(hire_date) as hire_month,            -- Extract month
    DAY(hire_date) as hire_day                 -- Extract day
FROM employees;

-- Date formatting
SELECT 
    DATE_FORMAT(hire_date, '%Y-%m-%d') as formatted_date,
    DATE_FORMAT(hire_date, '%M %D, %Y') as pretty_date
FROM employees;
```

## Stored Procedures

### 1. Basic Stored Procedure
```sql
-- Create procedure
CREATE PROCEDURE get_employee_info(IN emp_id INT)
BEGIN
    SELECT 
        first_name,
        last_name,
        salary,
        department
    FROM employees
    WHERE employee_id = emp_id;
END;

-- Call procedure
CALL get_employee_info(123);
```

### 2. Procedure with Multiple Parameters
```sql
CREATE PROCEDURE update_employee_salary(
    IN emp_id INT,
    IN new_salary DECIMAL(10,2),
    OUT status VARCHAR(50)
)
BEGIN
    DECLARE current_salary DECIMAL(10,2);
    
    -- Get current salary
    SELECT salary INTO current_salary
    FROM employees
    WHERE employee_id = emp_id;
    
    -- Update if found
    IF current_salary IS NOT NULL THEN
        UPDATE employees 
        SET salary = new_salary 
        WHERE employee_id = emp_id;
        SET status = 'Success';
    ELSE
        SET status = 'Employee not found';
    END IF;
END;
```

### 3. Function vs Procedure
```sql
-- Function (returns a value)
CREATE FUNCTION calculate_bonus(
    salary DECIMAL(10,2)
) 
RETURNS DECIMAL(10,2)
BEGIN
    DECLARE bonus DECIMAL(10,2);
    SET bonus = salary * 0.1;
    RETURN bonus;
END;

-- Using the function
SELECT 
    first_name,
    salary,
    calculate_bonus(salary) as bonus
FROM employees;
```

## Practice Problems

### 1. String Manipulation
```sql
-- Create email from name
SELECT 
    first_name,
    last_name,
    LOWER(CONCAT(
        SUBSTRING(first_name, 1, 1),
        last_name,
        '@company.com'
    )) as email
FROM employees;
```

### 2. Date Calculations
```sql
-- Calculate service years
SELECT 
    first_name,
    hire_date,
    FLOOR(DATEDIFF(CURRENT_DATE, hire_date) / 365) as years_of_service
FROM employees;
```

## Interview Questions

1. **What's the difference between a function and a stored procedure?**
   - Functions must return a value, procedures may not
   - Functions can be used in SELECT statements
   - Procedures can have output parameters
   - Procedures can perform transactions

2. **How do you handle NULL values in functions?**
   - Use IFNULL or COALESCE
   - Check for NULL in conditions
   - Handle NULL in aggregate functions
   - Understand NULL propagation

3. **Explain the importance of DETERMINISTIC vs NON-DETERMINISTIC functions**
   - DETERMINISTIC returns same output for same input
   - NON-DETERMINISTIC may vary (e.g., CURRENT_DATE)
   - Affects optimization and replication
   - Important for index creation
