# SQL Practice Problems: LeetCode and HackerRank Solutions

## Easy Level Problems

### 1. Combine Two Tables (LeetCode #175)
```sql
Problem:
Write a query to report the first name, last name, city, and state of each person.

Tables:
Person(personId, firstName, lastName)
Address(addressId, personId, city, state)

Solution:
SELECT 
    p.firstName,
    p.lastName,
    a.city,
    a.state
FROM Person p
LEFT JOIN Address a ON p.personId = a.personId;
```

### 2. Employees Earning More Than Their Managers (LeetCode #181)
```sql
Problem:
Find employees who earn more than their managers.

Table:
Employee(id, name, salary, managerId)

Solution:
SELECT e1.name as Employee
FROM Employee e1
JOIN Employee e2 ON e1.managerId = e2.id
WHERE e1.salary > e2.salary;
```

### 3. Duplicate Emails (LeetCode #182)
```sql
Problem:
Find all duplicate emails in the Person table.

Table:
Person(id, email)

Solution:
SELECT email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

## Medium Level Problems

### 1. Department Highest Salary (LeetCode #184)
```sql
Problem:
Find employees who have the highest salary in each department.

Tables:
Employee(id, name, salary, departmentId)
Department(id, name)

Solution:
WITH RankedSalaries AS (
    SELECT 
        e.name as Employee,
        e.salary as Salary,
        d.name as Department,
        RANK() OVER (
            PARTITION BY e.departmentId 
            ORDER BY e.salary DESC
        ) as salary_rank
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
)
SELECT 
    Department,
    Employee,
    Salary
FROM RankedSalaries
WHERE salary_rank = 1;
```

### 2. Consecutive Numbers (LeetCode #180)
```sql
Problem:
Find numbers that appear at least three times consecutively.

Table:
Logs(id, num)

Solution:
WITH ConsecutiveNums AS (
    SELECT 
        num,
        LEAD(num) OVER (ORDER BY id) as next_num,
        LEAD(num, 2) OVER (ORDER BY id) as next_next_num
    FROM Logs
)
SELECT DISTINCT num as ConsecutiveNums
FROM ConsecutiveNums
WHERE num = next_num 
AND num = next_next_num;
```

## Hard Level Problems

### 1. Department Top Three Salaries (LeetCode #185)
```sql
Problem:
Find the top 3 salaries for each department.

Tables:
Employee(id, name, salary, departmentId)
Department(id, name)

Solution:
WITH RankedSalaries AS (
    SELECT 
        e.name as Employee,
        e.salary as Salary,
        d.name as Department,
        DENSE_RANK() OVER (
            PARTITION BY e.departmentId 
            ORDER BY e.salary DESC
        ) as salary_rank
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
)
SELECT 
    Department,
    Employee,
    Salary
FROM RankedSalaries
WHERE salary_rank <= 3;
```

### 2. Human Traffic of Stadium (LeetCode #601)
```sql
Problem:
Find consecutive days where people >= 100.

Table:
Stadium(id, visit_date, people)

Solution:
WITH ConsecutiveDays AS (
    SELECT *,
        COUNT(*) OVER (
            PARTITION BY dateadd(day, -ROW_NUMBER() 
            OVER (ORDER BY visit_date), visit_date)
        ) as consecutive_count
    FROM Stadium
    WHERE people >= 100
)
SELECT 
    id,
    visit_date,
    people
FROM ConsecutiveDays
WHERE consecutive_count >= 3
ORDER BY visit_date;
```

## HackerRank Advanced Problems

### 1. Weather Observation Station
```sql
Problem:
Find the median city latitude rounded to 4 decimal places.

Solution:
SELECT ROUND(
    AVG(LAT_N),
    4
) as median_lat
FROM (
    SELECT LAT_N,
           ROW_NUMBER() OVER (ORDER BY LAT_N) as row_num,
           COUNT(*) OVER () as total_rows
    FROM STATION
) ranked
WHERE row_num IN (
    (total_rows + 1)/2,
    (total_rows + 2)/2
);
```

### 2. Interviews
```sql
Problem:
Calculate contest statistics including view_stats and submission_stats.

Solution:
SELECT 
    con.contest_id,
    con.hacker_id,
    con.name,
    SUM(sg.total_submissions),
    SUM(sg.total_accepted_submissions),
    SUM(vg.total_views),
    SUM(vg.total_unique_views)
FROM Contests con
JOIN Colleges col ON con.contest_id = col.contest_id
JOIN Challenges cha ON col.college_id = cha.college_id
LEFT JOIN (
    SELECT 
        challenge_id,
        SUM(total_submissions) as total_submissions,
        SUM(total_accepted_submissions) as total_accepted_submissions
    FROM Submission_Stats
    GROUP BY challenge_id
) sg ON cha.challenge_id = sg.challenge_id
LEFT JOIN (
    SELECT 
        challenge_id,
        SUM(total_views) as total_views,
        SUM(total_unique_views) as total_unique_views
    FROM View_Stats
    GROUP BY challenge_id
) vg ON cha.challenge_id = vg.challenge_id
GROUP BY 
    con.contest_id,
    con.hacker_id,
    con.name
HAVING 
    SUM(sg.total_submissions) +
    SUM(sg.total_accepted_submissions) +
    SUM(vg.total_views) +
    SUM(vg.total_unique_views) > 0
ORDER BY con.contest_id;
```

## Tips for Solving SQL Problems

1. **Understand the Schema**
   - Review table structures
   - Identify relationships
   - Check data types
   - Consider constraints

2. **Break Down Complex Problems**
   - Start with simpler subqueries
   - Use CTEs for clarity
   - Test intermediate results
   - Build solution incrementally

3. **Optimization Considerations**
   - Choose appropriate joins
   - Use window functions effectively
   - Consider indexing
   - Test with different data sizes

4. **Common Techniques**
   - Window functions for ranking
   - Self joins for hierarchical data
   - CTEs for complex logic
   - Subqueries for filtering
