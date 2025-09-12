# SQL Learning Roadmap and Quick Reference

## Learning Path Overview

### Foundation Phase (Weeks 1-3)
1. **SQL Fundamentals**
   ```sql
   -- First SQL Query
   SELECT * FROM employees;
   
   -- Basic Filtering
   SELECT name, salary 
   FROM employees 
   WHERE department = 'IT';
   ```

2. **Core Concepts**
   - Database and Table Structure
   - Primary Keys and Foreign Keys
   - Data Types and Constraints
   - Basic SQL Commands (SELECT, INSERT, UPDATE, DELETE)

3. **Essential Clauses**
   ```sql
   -- Sorting Data
   SELECT * FROM products ORDER BY price DESC;
   
   -- Filtering Groups
   SELECT department, COUNT(*) 
   FROM employees 
   GROUP BY department 
   HAVING COUNT(*) > 10;
   ```

4. **Practice Goals**
   - ⭐ Write basic SELECT queries
   - ⭐ Filter data with WHERE
   - ⭐ Sort with ORDER BY
   - ⭐ Group data with GROUP BY
   - ⭐ Use basic functions (COUNT, SUM, AVG, MIN, MAX)

### Intermediate Phase (Weeks 4-6)
1. **Data Manipulation**
   ```sql
   -- Insert Data
   INSERT INTO employees (name, salary, department)
   VALUES ('John Doe', 50000, 'IT');
   
   -- Update Data
   UPDATE employees 
   SET salary = salary * 1.1 
   WHERE performance_rating > 8;
   
   -- Delete Data
   DELETE FROM products 
   WHERE discontinued = TRUE 
   AND last_sale_date < DATEADD(month, -6, GETDATE());
   ```

2. **SQL Functions**
   ```sql
   -- String Functions
   SELECT 
       UPPER(first_name),
       LOWER(last_name),
       LENGTH(email),
       SUBSTRING(phone, 1, 3) as area_code
   FROM contacts;
   
   -- Date Functions
   SELECT 
       order_date,
       DATEADD(day, 7, order_date) as delivery_date,
       DATEDIFF(day, order_date, shipped_date) as days_to_ship
   FROM orders;
   
   -- Numeric Functions
   SELECT 
       product_name,
       ROUND(price * 0.9, 2) as discounted_price,
       CEIL(quantity * 1.5) as reorder_quantity
   FROM products;
   ```

3. **Practice Goals**
   - ⭐ Master DML operations
   - ⭐ Use built-in functions effectively
   - ⭐ Handle data transformations
   - ⭐ Perform complex calculations
   - ⭐ Work with multiple tables

### Advanced Phase (Weeks 7-10)
1. **Table Relationships**
   ```sql
   -- INNER JOIN
   SELECT 
       e.name, d.department_name
   FROM employees e
   INNER JOIN departments d ON e.dept_id = d.id;
   
   -- LEFT JOIN with Multiple Tables
   SELECT 
       c.name, 
       o.order_date,
       p.product_name
   FROM customers c
   LEFT JOIN orders o ON c.id = o.customer_id
   LEFT JOIN order_items oi ON o.id = oi.order_id
   LEFT JOIN products p ON oi.product_id = p.id;
   
   -- FULL OUTER JOIN
   SELECT 
       p.product_name,
       c.category_name
   FROM products p
   FULL OUTER JOIN categories c ON p.category_id = c.id;
   ```

2. **Subqueries & Advanced Queries**
   ```sql
   -- Correlated Subquery
   SELECT 
       department,
       employee_name,
       salary
   FROM employees e1
   WHERE salary > (
       SELECT AVG(salary)
       FROM employees e2
       WHERE e1.department = e2.department
   );
   
   -- EXISTS Example
   SELECT 
       product_name
   FROM products p
   WHERE EXISTS (
       SELECT 1 
       FROM order_items oi
       WHERE oi.product_id = p.id
       AND oi.quantity > 100
   );
   
   -- Common Table Expression (CTE)
   WITH HighValueOrders AS (
       SELECT 
           customer_id,
           SUM(amount) as total_spent
       FROM orders
       GROUP BY customer_id
       HAVING SUM(amount) > 10000
   )
   SELECT 
       c.name,
       hvo.total_spent
   FROM customers c
   INNER JOIN HighValueOrders hvo 
   ON c.id = hvo.customer_id;
   ```

3. **Practice Goals**
   - ⭐ Master different types of JOINs
   - ⭐ Write complex subqueries
   - ⭐ Use CTEs for better readability
   - ⭐ Optimize query performance
   - ⭐ Handle real-world data relationships

### Expert Phase (Weeks 11-14)
1. **Window Functions**
   ```sql
   -- Ranking Functions
   SELECT 
       department,
       employee_name,
       salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as salary_rank,
       DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as row_num
   FROM employees;
   
   -- LAG/LEAD Functions
   SELECT 
       date,
       sales_amount,
       LAG(sales_amount) OVER (ORDER BY date) as previous_day_sales,
       LEAD(sales_amount) OVER (ORDER BY date) as next_day_sales
   FROM daily_sales;
   
   -- Running Totals
   SELECT 
       date,
       amount,
       SUM(amount) OVER (ORDER BY date) as running_total,
       AVG(amount) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as weekly_avg
   FROM transactions;
   ```

2. **Views & Stored Procedures**
   ```sql
   -- Creating Views
   CREATE VIEW employee_details AS
   SELECT 
       e.id,
       e.name,
       d.department_name,
       e.salary,
       e.hire_date
   FROM employees e
   JOIN departments d ON e.dept_id = d.id;
   
   -- Stored Procedure
   CREATE PROCEDURE calculate_bonus
       @department_id INT,
       @bonus_percentage DECIMAL(5,2)
   AS
   BEGIN
       UPDATE employees
       SET bonus = salary * (@bonus_percentage / 100)
       WHERE dept_id = @department_id
       AND performance_rating > 8;
       
       SELECT 
           name,
           salary,
           bonus
       FROM employees
       WHERE dept_id = @department_id;
   END;
   
   -- Trigger Example
   CREATE TRIGGER after_employee_update
   ON employees
   AFTER UPDATE
   AS
   BEGIN
       INSERT INTO employee_audit (
           employee_id,
           changed_field,
           old_value,
           new_value,
           change_date
       )
       SELECT 
           i.id,
           'salary',
           d.salary,
           i.salary,
           GETDATE()
       FROM inserted i
       JOIN deleted d ON i.id = d.id
       WHERE i.salary != d.salary;
   END;
   ```

3. **Practice Goals**
   - ⭐ Master window functions
   - ⭐ Create and maintain views
   - ⭐ Develop stored procedures
   - ⭐ Implement triggers
   - ⭐ Build complex analytical queries

### Optimization Phase (Weeks 15-18)
1. **Query Optimization**
   ```sql
   -- Using EXPLAIN/EXECUTION PLAN
   EXPLAIN
   SELECT 
       c.name,
       COUNT(o.id) as order_count,
       SUM(o.amount) as total_spent
   FROM customers c
   LEFT JOIN orders o ON c.id = o.customer_id
   GROUP BY c.id, c.name
   HAVING COUNT(o.id) > 10;
   
   -- Index Usage
   CREATE INDEX idx_orders_customer 
   ON orders(customer_id, order_date)
   INCLUDE (amount);
   
   -- Statistics Update
   UPDATE STATISTICS orders;
   ```

2. **Performance Monitoring**
   ```sql
   -- Query Performance Stats
   SELECT 
       query_id,
       execution_count,
       total_elapsed_time,
       total_elapsed_time/execution_count as avg_elapsed_time,
       total_logical_reads/execution_count as avg_logical_reads
   FROM sys.dm_exec_query_stats;
   
   -- Index Usage Stats
   SELECT 
       OBJECT_NAME(i.object_id) as table_name,
       i.name as index_name,
       ius.user_seeks,
       ius.user_scans,
       ius.user_lookups,
       ius.user_updates
   FROM sys.dm_db_index_usage_stats ius
   JOIN sys.indexes i 
       ON ius.object_id = i.object_id 
       AND ius.index_id = i.index_id;
   ```

3. **Best Practices**
   - Use appropriate indexes
   - Avoid SELECT *
   - Use parameterized queries
   - Regular maintenance tasks
   - Monitor query performance

4. **Practice Goals**
   - ⭐ Analyze query plans
   - ⭐ Optimize query performance
   - ⭐ Design efficient indexes
   - ⭐ Monitor database health
   - ⭐ Implement best practices

### Level 6: Performance and Optimization (4-6 weeks)
1. **Query Optimization**
   - Execution plans
   - Index optimization
   - Join strategies
   - Query tuning

2. **Performance Monitoring**
   - System statistics
   - Query profiling
   - Resource usage
   - Troubleshooting

3. **Practice Goals**
   - Optimize queries
   - Monitor performance
   - Solve real issues
   - Handle large datasets

## Quick Reference

### Essential Commands
```sql
-- SELECT basics
SELECT column1, column2 FROM table;
SELECT DISTINCT column FROM table;
SELECT * FROM table WHERE condition;
SELECT column FROM table ORDER BY column [ASC|DESC];

-- Aggregation
SELECT COUNT(*), SUM(column), AVG(column) FROM table;
SELECT column, COUNT(*) FROM table GROUP BY column;
SELECT column, COUNT(*) FROM table GROUP BY column HAVING COUNT(*) > 5;

-- JOIN operations
SELECT a.col, b.col FROM table1 a JOIN table2 b ON a.id = b.id;
SELECT a.col, b.col FROM table1 a LEFT JOIN table2 b ON a.id = b.id;

-- Data Modification
INSERT INTO table (col1, col2) VALUES (val1, val2);
UPDATE table SET col1 = val1 WHERE condition;
DELETE FROM table WHERE condition;
```

### Common Functions
```sql
-- String Functions
UPPER(string)
LOWER(string)
SUBSTRING(string, start, length)
CONCAT(string1, string2)

-- Date Functions
CURRENT_DATE
DATE_ADD(date, INTERVAL value unit)
DATEDIFF(date1, date2)
DATE_FORMAT(date, format)

-- Numeric Functions
ROUND(number, decimals)
CEIL(number)
FLOOR(number)
ABS(number)

-- Window Functions
ROW_NUMBER() OVER (PARTITION BY col ORDER BY col)
RANK() OVER (ORDER BY col)
LAG(col) OVER (ORDER BY col)
LEAD(col) OVER (ORDER BY col)
```

### Common Patterns
```sql
-- Find duplicates
SELECT column, COUNT(*)
FROM table
GROUP BY column
HAVING COUNT(*) > 1;

-- Running totals
SELECT column,
       SUM(value) OVER (ORDER BY date) as running_total
FROM table;

-- Ranking within groups
SELECT column,
       RANK() OVER (PARTITION BY group_col ORDER BY value DESC)
FROM table;

-- Pivot data
SELECT *
FROM (
    SELECT category, month, value
    FROM table
) AS source
PIVOT (
    SUM(value)
    FOR month IN (Jan, Feb, Mar)
) AS pivot_table;
```

## Practice Strategy

1. **Daily Practice**
   - Solve 2-3 problems daily
   - Mix easy and medium problems
   - Review solutions

2. **Weekly Projects**
   - Complete one case study
   - Implement learned concepts
   - Review and optimize code

3. **Monthly Review**
   - Revisit complex topics
   - Practice advanced problems
   - Work on optimization

4. **Continuous Learning**
   - Read documentation
   - Study real-world examples
   - Participate in discussions

## Resources

1. **Documentation**
   - MySQL Documentation
   - PostgreSQL Documentation
   - SQL Server Documentation

2. **Practice Platforms**
   - LeetCode
   - HackerRank
   - SQLZoo

3. **Advanced Learning**
   - Database internals
   - Performance tuning
   - System design

## Success Metrics

1. **Basic Level**
   - Write basic queries
   - Understand JOIN operations
   - Use common functions

2. **Intermediate Level**
   - Write complex queries
   - Optimize simple queries
   - Design basic schemas

3. **Advanced Level**
   - Optimize complex queries
   - Design efficient schemas
   - Handle performance issues

4. **Expert Level**
   - System design
   - Performance tuning
   - Advanced optimization
