# SQL Joins and Relationships

## Types of Joins

### INNER JOIN
```sql
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### LEFT JOIN
```sql
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### RIGHT JOIN
```sql
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

### FULL OUTER JOIN
```sql
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

## Subqueries

### Single Row Subqueries
```sql
-- Find employees with salary greater than average
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Multiple Row Subqueries
```sql
-- Find employees in departments with avg salary > 50000
SELECT * FROM employees
WHERE dept_id IN (
    SELECT dept_id 
    FROM employees 
    GROUP BY dept_id 
    HAVING AVG(salary) > 50000
);
```

### Correlated Subqueries
```sql
-- Find employees with highest salary in their department
SELECT * 
FROM employees e1
WHERE salary = (
    SELECT MAX(salary)
    FROM employees e2
    WHERE e1.dept_id = e2.dept_id
);
```

## Practice Problems

### LeetCode Problems
1. [#175 Combine Two Tables](https://leetcode.com/problems/combine-two-tables/)
2. [#181 Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/)

### Complex Join Scenarios
1. Multi-table joins
2. Self-joins
3. Cross joins

## Interview Questions
1. Explain the difference between INNER and LEFT JOIN
2. When to use subqueries vs joins
3. Performance implications of different types of joins
