# Advanced LeetCode SQL Problems and Solutions

## Advanced Problems Collection

### 1. Market Analysis (LeetCode #1158)
```sql
Problem:
Find for each user, the join date and the number of orders they made as a buyer in 2019.

Tables:
Users(user_id, join_date, favorite_brand)
Orders(order_id, order_date, item_id, buyer_id, seller_id)
Items(item_id, item_brand)

Solution:
SELECT 
    u.user_id as buyer_id,
    u.join_date,
    COUNT(CASE 
        WHEN YEAR(o.order_date) = 2019 
        THEN o.order_id 
        ELSE NULL 
    END) as orders_in_2019
FROM Users u
LEFT JOIN Orders o ON u.user_id = o.buyer_id
GROUP BY u.user_id, u.join_date;
```

### 2. Product Sales Analysis III (LeetCode #1070)
```sql
Problem:
Find the first year of sale for each product and the total quantity and price.

Tables:
Sales(sale_id, product_id, year, quantity, price)
Product(product_id, product_name)

Solution:
WITH FirstSales AS (
    SELECT 
        product_id,
        MIN(year) as first_year
    FROM Sales
    GROUP BY product_id
)
SELECT 
    s.product_id,
    f.first_year,
    s.quantity,
    s.price
FROM Sales s
JOIN FirstSales f 
    ON s.product_id = f.product_id 
    AND s.year = f.first_year;
```

### 3. Tournament Winners (LeetCode #1194)
```sql
Problem:
Find the winner in each group.

Tables:
Players(player_id, group_id)
Matches(match_id, first_player, second_player, first_score, second_score)

Solution:
WITH PlayerScores AS (
    -- First player scores
    SELECT 
        first_player as player_id,
        first_score as score
    FROM Matches
    UNION ALL
    -- Second player scores
    SELECT 
        second_player as player_id,
        second_score as score
    FROM Matches
),
TotalScores AS (
    SELECT 
        p.player_id,
        p.group_id,
        COALESCE(SUM(ps.score), 0) as total_score,
        RANK() OVER (
            PARTITION BY p.group_id 
            ORDER BY COALESCE(SUM(ps.score), 0) DESC, p.player_id
        ) as score_rank
    FROM Players p
    LEFT JOIN PlayerScores ps ON p.player_id = ps.player_id
    GROUP BY p.player_id, p.group_id
)
SELECT 
    group_id,
    player_id
FROM TotalScores
WHERE score_rank = 1;
```

### 4. Report Contiguous Dates (LeetCode #1225)
```sql
Problem:
Find periods of continuous 'failed' and 'succeeded' states.

Tables:
Failed(fail_date)
Succeeded(success_date)

Solution:
WITH AllDates AS (
    SELECT fail_date as date, 'failed' as state
    FROM Failed
    UNION ALL
    SELECT success_date as date, 'succeeded' as state
    FROM Succeeded
),
RankedDates AS (
    SELECT 
        *,
        DATE_SUB(date, 
            INTERVAL ROW_NUMBER() OVER (
                PARTITION BY state 
                ORDER BY date
            ) DAY
        ) as group_date
    FROM AllDates
)
SELECT 
    state,
    MIN(date) as start_date,
    MAX(date) as end_date
FROM RankedDates
GROUP BY state, group_date
ORDER BY start_date;
```

### 5. Last Person to Fit in Bus (LeetCode #1204)
```sql
Problem:
Find the person who is last to fit on a bus with weight limit of 1000 pounds.

Table:
Queue(person_id, person_name, weight, turn)

Solution:
WITH RunningWeight AS (
    SELECT 
        person_name,
        turn,
        SUM(weight) OVER (
            ORDER BY turn
        ) as total_weight
    FROM Queue
)
SELECT person_name
FROM RunningWeight
WHERE total_weight <= 1000
ORDER BY turn DESC
LIMIT 1;
```

### 6. Find Cumulative Salary (LeetCode #579)
```sql
Problem:
Calculate cumulative salary for each employee over 3 months excluding most recent month.

Table:
Employee(id, month, salary)

Solution:
WITH SalaryRank AS (
    SELECT 
        id,
        month,
        salary,
        RANK() OVER (
            PARTITION BY id 
            ORDER BY month DESC
        ) as month_rank
    FROM Employee
),
CumulativeSalary AS (
    SELECT 
        id,
        month,
        SUM(salary) OVER (
            PARTITION BY id 
            ORDER BY month 
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ) as cumulative_salary
    FROM SalaryRank
    WHERE month_rank > 1
)
SELECT 
    id,
    month,
    cumulative_salary
FROM CumulativeSalary;
```

### 7. Activity Participants (LeetCode #1355)
```sql
Problem:
Find activities that have neither maximum nor minimum number of participants.

Table:
Activities(activity, participants)

Solution:
WITH ActivityStats AS (
    SELECT 
        activity,
        participants,
        MAX(participants) OVER () as max_participants,
        MIN(participants) OVER () as min_participants
    FROM Activities
)
SELECT activity
FROM ActivityStats
WHERE participants NOT IN (max_participants, min_participants);
```

### 8. Monthly Transactions II (LeetCode #1205)
```sql
Problem:
Find for each month and country: approved transactions count and amount, chargeback count and amount.

Tables:
Transactions(id, country, state, amount, trans_date)
Chargebacks(trans_id, trans_date)

Solution:
WITH AllTransactions AS (
    -- Approved transactions
    SELECT 
        DATE_FORMAT(trans_date, '%Y-%m') as month,
        country,
        COUNT(*) as approved_count,
        SUM(amount) as approved_amount,
        0 as chargeback_count,
        0 as chargeback_amount
    FROM Transactions
    WHERE state = 'approved'
    GROUP BY DATE_FORMAT(trans_date, '%Y-%m'), country
    
    UNION ALL
    
    -- Chargebacks
    SELECT 
        DATE_FORMAT(c.trans_date, '%Y-%m') as month,
        t.country,
        0 as approved_count,
        0 as approved_amount,
        COUNT(*) as chargeback_count,
        SUM(t.amount) as chargeback_amount
    FROM Chargebacks c
    JOIN Transactions t ON c.trans_id = t.id
    GROUP BY DATE_FORMAT(c.trans_date, '%Y-%m'), t.country
)
SELECT 
    month,
    country,
    SUM(approved_count) as approved_count,
    SUM(approved_amount) as approved_amount,
    SUM(chargeback_count) as chargeback_count,
    SUM(chargeback_amount) as chargeback_amount
FROM AllTransactions
GROUP BY month, country
HAVING 
    SUM(approved_count) > 0 
    OR SUM(chargeback_count) > 0
ORDER BY month;
```

## Problem-Solving Tips

### 1. Window Functions
- Use for running totals
- Calculate ranks and dense ranks
- Create moving averages
- Handle gaps in sequences

### 2. Common Table Expressions (CTEs)
- Break down complex problems
- Improve readability
- Reuse intermediate results
- Handle recursive queries

### 3. Advanced Grouping
- GROUP BY with multiple columns
- HAVING for filtered aggregates
- ROLLUP for subtotals
- CUBE for cross-tabulations

### 4. Date/Time Handling
- Use proper date functions
- Consider time zones
- Handle fiscal periods
- Calculate date ranges

### 5. Performance Considerations
- Proper indexing
- Avoid correlated subqueries
- Use appropriate joins
- Consider data volume
