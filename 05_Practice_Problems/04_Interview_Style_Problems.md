# SQL Interview Problems and Solutions

## Table of Contents
1. [Aggregation Problems](#aggregation-problems)
2. [Window Function Problems](#window-function-problems)
3. [Join and Subquery Problems](#join-and-subquery-problems)
4. [Date Manipulation Problems](#date-manipulation-problems)
5. [Advanced Pattern Matching](#advanced-pattern-matching)

## Aggregation Problems

### 1. Employee Department Analysis
```sql
/* Problem: Find departments where the average salary is higher than 
   the company's overall average salary */

Table Structure:
Employees:
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| emp_id      | int      |
| dept_id     | int      |
| salary      | decimal  |
+-------------+----------+

Solution:
WITH company_avg AS (
    SELECT AVG(salary) as avg_salary
    FROM Employees
)
SELECT 
    d.dept_name,
    AVG(e.salary) as dept_avg_salary,
    c.avg_salary as company_avg_salary
FROM Employees e
JOIN Departments d ON e.dept_id = d.dept_id
CROSS JOIN company_avg c
GROUP BY d.dept_name, c.avg_salary
HAVING AVG(e.salary) > c.avg_salary
ORDER BY dept_avg_salary DESC;
```

### 2. Sales Performance Metrics
```sql
/* Problem: Calculate month-over-month sales growth percentage by product category */

Tables:
Sales:
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| sale_id     | int      |
| prod_id     | int      |
| sale_date   | date     |
| amount      | decimal  |
+-------------+----------+

Solution:
WITH monthly_sales AS (
    SELECT 
        p.category,
        DATE_TRUNC('month', s.sale_date) as month,
        SUM(s.amount) as total_sales
    FROM Sales s
    JOIN Products p ON s.prod_id = p.prod_id
    GROUP BY p.category, DATE_TRUNC('month', s.sale_date)
)
SELECT 
    category,
    month,
    total_sales,
    LAG(total_sales) OVER (PARTITION BY category ORDER BY month) as prev_month_sales,
    CASE 
        WHEN LAG(total_sales) OVER (PARTITION BY category ORDER BY month) IS NULL THEN NULL
        ELSE ROUND(
            ((total_sales - LAG(total_sales) OVER (PARTITION BY category ORDER BY month)) * 100.0 / 
            NULLIF(LAG(total_sales) OVER (PARTITION BY category ORDER BY month), 0))::numeric, 
            2
        )
    END as growth_percentage
FROM monthly_sales
ORDER BY category, month;
```

## Window Function Problems

### 1. Ranking Customer Orders
```sql
/* Problem: Rank customers based on their order value in each quarter */

Tables:
Orders:
+---------------+----------+
| Column Name   | Type     |
+---------------+----------+
| order_id      | int      |
| customer_id   | int      |
| order_date    | date     |
| order_amount  | decimal  |
+---------------+----------+

Solution:
WITH quarterly_orders AS (
    SELECT 
        customer_id,
        DATE_TRUNC('quarter', order_date) as quarter,
        SUM(order_amount) as total_amount
    FROM Orders
    GROUP BY customer_id, DATE_TRUNC('quarter', order_date)
)
SELECT 
    c.customer_name,
    q.quarter,
    q.total_amount,
    RANK() OVER (PARTITION BY q.quarter ORDER BY q.total_amount DESC) as rank_in_quarter,
    DENSE_RANK() OVER (PARTITION BY q.quarter ORDER BY q.total_amount DESC) as dense_rank_in_quarter
FROM quarterly_orders q
JOIN Customers c ON q.customer_id = c.customer_id
ORDER BY q.quarter, q.total_amount DESC;
```

### 2. Running Total Calculation
```sql
/* Problem: Calculate running total of transactions for each account */

Tables:
Transactions:
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| txn_id         | int      |
| account_id     | int      |
| txn_date       | date     |
| amount         | decimal  |
+----------------+----------+

Solution:
SELECT 
    t.account_id,
    t.txn_date,
    t.amount,
    SUM(t.amount) OVER (
        PARTITION BY t.account_id 
        ORDER BY t.txn_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total,
    AVG(t.amount) OVER (
        PARTITION BY t.account_id 
        ORDER BY t.txn_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3_days
FROM Transactions t
ORDER BY t.account_id, t.txn_date;
```

## Join and Subquery Problems

### 1. Complex Sales Analysis
```sql
/* Problem: Find products that have higher than average sales in their category */

Tables:
Sales:
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| sale_id     | int      |
| prod_id     | int      |
| sale_date   | date     |
| quantity    | int      |
+-------------+----------+

Products:
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| prod_id     | int      |
| category    | varchar  |
| price       | decimal  |
+-------------+----------+

Solution:
WITH category_avg_sales AS (
    SELECT 
        p.category,
        AVG(s.quantity * p.price) as avg_category_sales
    FROM Sales s
    JOIN Products p ON s.prod_id = p.prod_id
    GROUP BY p.category
)
SELECT 
    p.prod_id,
    p.prod_name,
    p.category,
    SUM(s.quantity * p.price) as total_sales,
    c.avg_category_sales,
    ROUND(
        (SUM(s.quantity * p.price) - c.avg_category_sales) * 100.0 / c.avg_category_sales,
        2
    ) as sales_diff_percentage
FROM Sales s
JOIN Products p ON s.prod_id = p.prod_id
JOIN category_avg_sales c ON p.category = c.category
GROUP BY 
    p.prod_id, 
    p.prod_name, 
    p.category, 
    c.avg_category_sales
HAVING SUM(s.quantity * p.price) > c.avg_category_sales
ORDER BY sales_diff_percentage DESC;
```

## Date Manipulation Problems

### 1. Continuous Date Range Analysis
```sql
/* Problem: Find gaps in continuous date ranges for user sessions */

Tables:
Sessions:
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| user_id        | int      |
| session_start  | datetime |
| session_end    | datetime |
+----------------+----------+

Solution:
WITH numbered_sessions AS (
    SELECT 
        user_id,
        session_start,
        session_end,
        LAG(session_end) OVER (
            PARTITION BY user_id 
            ORDER BY session_start
        ) as prev_session_end
    FROM Sessions
)
SELECT 
    user_id,
    prev_session_end as gap_start,
    session_start as gap_end,
    EXTRACT(EPOCH FROM (session_start - prev_session_end))/3600 as gap_hours
FROM numbered_sessions
WHERE 
    prev_session_end IS NOT NULL 
    AND session_start > prev_session_end + INTERVAL '1 hour'
ORDER BY user_id, gap_start;
```

## Advanced Pattern Matching

### 1. String Pattern Analysis
```sql
/* Problem: Find products where description matches specific patterns */

Tables:
Products:
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| prod_id        | int      |
| description    | text     |
+----------------+----------+

Solution:
SELECT 
    prod_id,
    description,
    CASE 
        WHEN description ~ '^\d{3}-\d{2}-\d{4}$' THEN 'SSN Format'
        WHEN description ~ '^[A-Z]{2}\d{6}$' THEN 'License Format'
        WHEN description ~ '^[\w\.-]+@[\w\.-]+\.\w+$' THEN 'Email Format'
        ELSE 'Other Format'
    END as pattern_type,
    REGEXP_MATCHES(
        description, 
        '(\w+@\w+\.\w+)',
        'g'
    ) as extracted_emails
FROM Products
WHERE 
    description ~ '^\d{3}-\d{2}-\d{4}$'
    OR description ~ '^[A-Z]{2}\d{6}$'
    OR description ~ '^[\w\.-]+@[\w\.-]+\.\w+$'
ORDER BY pattern_type, prod_id;
```
Activity(player_id, device_id, event_date, games_played)

Solution:
WITH FirstLogins AS (
    SELECT 
        player_id, 
        MIN(event_date) as first_login
    FROM Activity
    GROUP BY player_id
)
SELECT ROUND(
    COUNT(DISTINCT a.player_id) / 
    COUNT(DISTINCT f.player_id),
    2
) as fraction
FROM FirstLogins f
LEFT JOIN Activity a 
    ON f.player_id = a.player_id 
    AND a.event_date = DATE_ADD(f.first_login, INTERVAL 1 DAY);
```

### 2. Active Users (LeetCode #1454)
```sql
Problem:
Find all users who were active for 5 consecutive days.

Table:
Logins(id, login_date)

Solution:
WITH ConsecutiveDates AS (
    SELECT 
        id,
        login_date,
        DATE_SUB(
            login_date, 
            INTERVAL ROW_NUMBER() OVER (
                PARTITION BY id 
                ORDER BY login_date
            ) DAY
        ) as group_date
    FROM (
        SELECT DISTINCT id, login_date
        FROM Logins
    ) distinct_logins
)
SELECT DISTINCT id
FROM ConsecutiveDates
GROUP BY id, group_date
HAVING COUNT(*) >= 5;
```

## Complex Join Problems

### 1. Project Employees III (LeetCode #1077)
```sql
Problem:
Report the most experienced employees in each project.

Tables:
Project(project_id, employee_id)
Employee(employee_id, name, experience_years)

Solution:
WITH RankedEmployees AS (
    SELECT 
        p.project_id,
        e.employee_id,
        e.name,
        e.experience_years,
        RANK() OVER (
            PARTITION BY p.project_id 
            ORDER BY e.experience_years DESC
        ) as exp_rank
    FROM Project p
    JOIN Employee e ON p.employee_id = e.employee_id
)
SELECT 
    project_id,
    employee_id
FROM RankedEmployees
WHERE exp_rank = 1;
```

### 2. Students Report By Geography (LeetCode #618)
```sql
Problem:
Pivot the continent column to get max name in one row.

Table:
Student(name, continent)

Solution:
WITH RankedStudents AS (
    SELECT 
        name,
        continent,
        ROW_NUMBER() OVER (
            PARTITION BY continent 
            ORDER BY name
        ) as rn
    FROM Student
)
SELECT 
    MAX(CASE WHEN continent = 'America' THEN name END) as America,
    MAX(CASE WHEN continent = 'Asia' THEN name END) as Asia,
    MAX(CASE WHEN continent = 'Europe' THEN name END) as Europe
FROM RankedStudents
GROUP BY rn;
```

## Window Function Problems

### 1. Median Employee Salary (LeetCode #569)
```sql
Problem:
Find the median salary of each company.

Table:
Employee(id, company, salary)

Solution:
WITH RankedSalaries AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY company 
            ORDER BY salary
        ) as rn,
        COUNT(*) OVER (
            PARTITION BY company
        ) as cnt
    FROM Employee
)
SELECT 
    id,
    company,
    salary
FROM RankedSalaries
WHERE rn IN ((cnt+1)/2, (cnt+2)/2);
```

### 2. Get Highest Answer Rate Question (LeetCode #578)
```sql
Problem:
Get the question with highest answer rate.

Tables:
SurveyLog(id, action, question_id, answer_id, q_num, timestamp)

Solution:
WITH QuestionStats AS (
    SELECT 
        question_id,
        SUM(CASE WHEN action = 'answer' THEN 1 ELSE 0 END) as answers,
        SUM(CASE WHEN action = 'show' THEN 1 ELSE 0 END) as shows
    FROM SurveyLog
    GROUP BY question_id
)
SELECT question_id
FROM QuestionStats
ORDER BY 
    (answers/shows) DESC,
    question_id ASC
LIMIT 1;
```

## Recursive CTE Problems

### 1. Human Traffic of Stadium (LeetCode #601)
```sql
Problem:
Find consecutive dates where people >= 100.

Table:
Stadium(id, visit_date, people)

Solution:
WITH ConsecutiveDates AS (
    SELECT 
        *,
        id - ROW_NUMBER() OVER (ORDER BY visit_date) as group_id
    FROM Stadium
    WHERE people >= 100
)
SELECT 
    id,
    visit_date,
    people
FROM ConsecutiveDates c1
WHERE EXISTS (
    SELECT 1
    FROM ConsecutiveDates c2
    WHERE c1.group_id = c2.group_id
    GROUP BY c2.group_id
    HAVING COUNT(*) >= 3
)
ORDER BY visit_date;
```

## Advanced String Manipulation

### 1. Find Users With Valid E-Mails (LeetCode #1517)
```sql
Problem:
Find users with valid email addresses.

Table:
Users(user_id, name, mail)

Solution:
SELECT *
FROM Users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode\\.com$';
```

## Interview Tips

1. **Understanding the Problem**
   - Read carefully
   - Identify key requirements
   - Consider edge cases
   - Ask clarifying questions

2. **Solution Approach**
   - Start with simple solution
   - Optimize step by step
   - Consider performance
   - Test with sample data

3. **Common Techniques**
   - Window functions
   - CTEs for readability
   - Proper joins
   - Efficient filtering

4. **Performance Considerations**
   - Index usage
   - Join optimization
   - Subquery efficiency
   - Data volume impact

5. **Best Practices**
   - Clean, readable code
   - Proper indentation
   - Meaningful aliases
   - Clear comments when needed
