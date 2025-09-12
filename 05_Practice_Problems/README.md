# SQL Practice Problems

## LeetCode Problems

### Easy Level
1. [#175 Combine Two Tables](https://leetcode.com/problems/combine-two-tables/)
2. [#181 Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/)
3. [#182 Duplicate Emails](https://leetcode.com/problems/duplicate-emails/)

### Medium Level
1. [#176 Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)
2. [#177 Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/)
3. [#178 Rank Scores](https://leetcode.com/problems/rank-scores/)

### Hard Level
1. [#185 Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/)
2. [#262 Trips and Users](https://leetcode.com/problems/trips-and-users/)

## HackerRank Problems

### Basic Select
1. Weather Observation Station series
2. Employee Names
3. Higher Than 75 Marks

### Advanced Select
1. Type of Triangle
2. The PADS
3. Occupations

### Aggregation
1. Top Earners
2. Weather Observation Station series
3. Population Census

## Interview Questions

### Basic Questions
1. What is the difference between DELETE and TRUNCATE?
2. Explain ACID properties
3. What is the difference between UNION and UNION ALL?

### Advanced Questions
1. How to optimize a slow running query?
2. Explain different types of indexes
3. What are the different types of joins?

## Real-world Scenarios

### E-commerce Database
```sql
-- Find top selling products
SELECT 
    p.name,
    SUM(oi.quantity) as total_sold
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.name
ORDER BY total_sold DESC;

-- Calculate customer lifetime value
SELECT 
    c.customer_id,
    c.first_name,
    SUM(oi.quantity * oi.price_at_time) as total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name
ORDER BY total_spent DESC;
```

### Social Media Analytics
```sql
-- Find most active users
SELECT 
    u.username,
    COUNT(p.post_id) as post_count,
    COUNT(DISTINCT f.follower_id) as follower_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
LEFT JOIN followers f ON u.user_id = f.user_id
GROUP BY u.user_id, u.username
ORDER BY post_count DESC;
```

## Practice Tips
1. Start with simple queries and progress to complex ones
2. Use real-world scenarios for practice
3. Focus on query optimization
4. Practice writing clean and maintainable code
