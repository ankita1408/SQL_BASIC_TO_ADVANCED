# SQL Joins Fundamentals

## Understanding Joins

### Types of Joins
1. INNER JOIN
2. LEFT (OUTER) JOIN
3. RIGHT (OUTER) JOIN
4. FULL (OUTER) JOIN
5. CROSS JOIN
6. SELF JOIN

## Detailed Examples

### 1. INNER JOIN
```sql
-- Basic syntax
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Multiple conditions
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
INNER JOIN departments d 
    ON e.department_id = d.department_id 
    AND e.location_id = d.location_id;

-- Multiple tables
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    l.city
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id;
```

### 2. LEFT (OUTER) JOIN
```sql
-- Basic syntax
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- Finding unmatched records
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

### 3. RIGHT (OUTER) JOIN
```sql
-- Basic syntax
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- Finding departments with no employees
SELECT d.department_name, e.first_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE e.employee_id IS NULL;
```

### 4. FULL (OUTER) JOIN
```sql
-- Basic syntax
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- Finding all unmatched records
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id
WHERE e.department_id IS NULL OR d.department_id IS NULL;
```

### 5. CROSS JOIN
```sql
-- Basic syntax
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
CROSS JOIN departments d;

-- Practical example: Generate all possible combinations
SELECT p.product_name, c.color
FROM products p
CROSS JOIN colors c;
```

### 6. SELF JOIN
```sql
-- Finding employees and their managers
SELECT 
    e1.first_name as employee_name,
    e2.first_name as manager_name
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.employee_id;

-- Finding peers (same department)
SELECT 
    e1.first_name as employee1,
    e2.first_name as employee2
FROM employees e1
INNER JOIN employees e2 
    ON e1.department_id = e2.department_id
    AND e1.employee_id < e2.employee_id;
```

## Common Join Patterns

### 1. Bridge Tables (Many-to-Many)
```sql
-- Students and Courses example
SELECT 
    s.student_name,
    c.course_name
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id;
```

### 2. Hierarchical Data
```sql
-- Organization hierarchy
WITH RECURSIVE org_hierarchy AS (
    -- Base case: top-level employees
    SELECT 
        employee_id,
        first_name,
        manager_id,
        1 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers
    SELECT 
        e.employee_id,
        e.first_name,
        e.manager_id,
        h.level + 1
    FROM employees e
    INNER JOIN org_hierarchy h ON e.manager_id = h.employee_id
)
SELECT * FROM org_hierarchy;
```

## Practice Problems

### Problem 1: Employee Report
```sql
-- Create a comprehensive employee report
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    l.city,
    c.country_name,
    m.first_name as manager_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
LEFT JOIN locations l ON d.location_id = l.location_id
LEFT JOIN countries c ON l.country_id = c.country_id
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

### Problem 2: Sales Analysis
```sql
-- Calculate sales by product and category
SELECT 
    c.category_name,
    p.product_name,
    SUM(o.quantity) as total_quantity,
    SUM(o.quantity * p.price) as total_revenue
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
LEFT JOIN order_items o ON p.product_id = o.product_id
GROUP BY c.category_name, p.product_name;
```

## Interview Questions

1. **What is the difference between INNER JOIN and LEFT JOIN?**
   - INNER JOIN returns only matching records
   - LEFT JOIN returns all records from left table
   - NULL values appear for unmatched right table records
   - Common use cases for each

2. **When would you use a CROSS JOIN?**
   - Generate combinations
   - Create test data
   - Calculate cartesian products
   - Performance considerations

3. **How do you optimize join performance?**
   - Use proper indexes
   - Join order matters
   - Avoid unnecessary columns
   - Consider table sizes
   - Use appropriate join types

4. **Explain the concept of a SELF JOIN**
   - Join table to itself
   - Used for hierarchical data
   - Manager-employee relationships
   - Peer relationships
