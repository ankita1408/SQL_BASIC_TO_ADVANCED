# SQL Data Types and Operators

## Data Types

### 1. Numeric Types
```sql
-- Integer Types
TINYINT    -- -128 to 127
SMALLINT   -- -32,768 to 32,767
INT        -- -2^31 to 2^31-1
BIGINT     -- -2^63 to 2^63-1

-- Decimal Types
DECIMAL(p,s)  -- p: precision, s: scale
NUMERIC(p,s)  -- same as DECIMAL
FLOAT(p)      -- floating point
REAL          -- floating point
MONEY         -- monetary data
```

### 2. String Types
```sql
-- Fixed Length
CHAR(n)      -- fixed-length string

-- Variable Length
VARCHAR(n)    -- variable-length string
TEXT          -- variable unlimited length
NVARCHAR(n)   -- Unicode variable-length
NTEXT         -- Unicode TEXT
```

### 3. Date and Time Types
```sql
DATE          -- YYYY-MM-DD
TIME          -- HH:MI:SS
DATETIME      -- DATE + TIME
TIMESTAMP     -- UTC DATETIME
YEAR          -- YYYY
```

### 4. Binary Types
```sql
BINARY(n)     -- fixed-length binary
VARBINARY(n)  -- variable-length binary
BLOB          -- binary large object
```

### 5. Boolean Type
```sql
BOOLEAN       -- TRUE/FALSE
BIT           -- 0/1
```

## Operators

### 1. Arithmetic Operators
```sql
-- Basic arithmetic
SELECT 
    salary + 1000 as raised_salary,    -- Addition
    salary - 1000 as reduced_salary,   -- Subtraction
    salary * 1.1 as bonus_salary,      -- Multiplication
    salary / 12 as monthly_salary,     -- Division
    salary % 1000 as remainder         -- Modulo
FROM employees;
```

### 2. Comparison Operators
```sql
-- Basic comparisons
WHERE salary = 50000     -- Equal to
WHERE salary > 50000     -- Greater than
WHERE salary < 50000     -- Less than
WHERE salary >= 50000    -- Greater than or equal
WHERE salary <= 50000    -- Less than or equal
WHERE salary <> 50000    -- Not equal to
WHERE salary != 50000    -- Not equal to (alternative)
```

### 3. Logical Operators
```sql
-- AND
WHERE salary > 50000 AND department = 'IT'

-- OR
WHERE department = 'IT' OR department = 'HR'

-- NOT
WHERE NOT department = 'IT'

-- IN
WHERE department IN ('IT', 'HR', 'Finance')

-- BETWEEN
WHERE salary BETWEEN 40000 AND 60000

-- EXISTS
WHERE EXISTS (SELECT 1 FROM departments WHERE departments.id = employees.dept_id)
```

### 4. String Operators
```sql
-- LIKE patterns
WHERE name LIKE 'John%'      -- Starts with John
WHERE name LIKE '%son'       -- Ends with son
WHERE name LIKE '%oh%'       -- Contains oh
WHERE name LIKE '_ohn'       -- Any single character followed by ohn

-- Concatenation
SELECT first_name || ' ' || last_name as full_name  -- Standard SQL
SELECT CONCAT(first_name, ' ', last_name)           -- MySQL
```

## Type Conversion

### 1. Implicit Conversion
```sql
-- Automatic type conversion
WHERE salary > '50000'    -- String to number
WHERE hire_date > '2025-01-01'  -- String to date
```

### 2. Explicit Conversion
```sql
-- CAST
SELECT CAST(salary AS VARCHAR) as salary_string
SELECT CAST('2025-01-01' AS DATE) as date_value

-- CONVERT (SQL Server)
SELECT CONVERT(VARCHAR, salary) as salary_string
SELECT CONVERT(DATE, '2025-01-01') as date_value
```

## Practice Problems

1. **Data Type Selection**
```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    description TEXT,
    created_at TIMESTAMP,
    is_available BOOLEAN
);
```

2. **Type Conversion**
```sql
-- Convert string to date
SELECT 
    order_id,
    CAST(order_date AS DATE) as order_date
FROM orders;

-- Format numbers
SELECT 
    product_name,
    CAST(price AS DECIMAL(10,2)) as formatted_price
FROM products;
```

## Interview Questions

1. **When would you use CHAR vs VARCHAR?**
   - CHAR for fixed-length strings
   - VARCHAR for variable-length strings
   - CHAR can be faster for fixed-length data
   - VARCHAR saves space for variable-length data

2. **Explain the difference between FLOAT and DECIMAL**
   - DECIMAL is exact, FLOAT is approximate
   - DECIMAL better for monetary calculations
   - FLOAT better for scientific calculations
   - DECIMAL requires more storage

3. **How do you handle NULL values in comparisons?**
   - Use IS NULL/IS NOT NULL
   - NULL = NULL is false
   - NULL in calculations results in NULL
   - COALESCE/IFNULL for default values
