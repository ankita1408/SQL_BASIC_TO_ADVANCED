# Advanced SQL Functions Guide

## NULL Handling Functions

### 1. COALESCE
Returns the first non-null value in a list.

```sql
-- Basic usage
SELECT 
    name,
    COALESCE(phone, email, 'No Contact') as contact
FROM users;

-- With calculations
SELECT 
    product_name,
    COALESCE(discount, 0) * price as discounted_price
FROM products;

-- Multiple levels
SELECT 
    COALESCE(
        preferred_contact,
        phone,
        email,
        'No contact available'
    ) as contact_method
FROM customers;
```

### 2. NULLIF
Returns NULL if two expressions are equal, otherwise returns the first expression.

```sql
-- Avoid division by zero
SELECT 
    sales_amount,
    costs,
    sales_amount / NULLIF(costs, 0) as profit_ratio
FROM sales;

-- Handle special values
SELECT 
    NULLIF(status, 'N/A') as filtered_status
FROM orders;
```

### 3. IFNULL (MySQL) / NVL (Oracle)
Returns a specified value if the expression is NULL.

```sql
-- Basic usage
SELECT 
    name,
    IFNULL(department, 'Unassigned') as dept
FROM employees;

-- With calculations
SELECT 
    product_id,
    IFNULL(quantity * price, 0) as total_value
FROM inventory;
```

## Window Functions for Analysis

### 1. LAG
Access data from previous rows in the result set.

```sql
-- Basic LAG
SELECT 
    date,
    sales_amount,
    LAG(sales_amount) OVER (ORDER BY date) as previous_day_sales
FROM daily_sales;

-- LAG with PARTITION
SELECT 
    department,
    employee_name,
    salary,
    LAG(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) as next_highest_salary
FROM employees;

-- Multiple LAG periods
SELECT 
    date,
    stock_price,
    LAG(stock_price, 1) OVER (ORDER BY date) as prev_day,
    LAG(stock_price, 7) OVER (ORDER BY date) as prev_week,
    LAG(stock_price, 30) OVER (ORDER BY date) as prev_month
FROM stock_prices;
```

### 2. LEAD
Access data from subsequent rows in the result set.

```sql
-- Basic LEAD
SELECT 
    date,
    sales_amount,
    LEAD(sales_amount) OVER (ORDER BY date) as next_day_sales
FROM daily_sales;

-- LEAD with PARTITION
SELECT 
    department,
    employee_name,
    salary,
    LEAD(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) as next_lower_salary
FROM employees;

-- Calculating future differences
SELECT 
    date,
    closing_price,
    LEAD(closing_price) OVER (ORDER BY date) - closing_price as price_change
FROM stock_prices;
```

## Common Use Cases

### 1. Time Series Analysis
```sql
-- Month-over-Month Growth
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) as prev_month_revenue,
    ((revenue - LAG(revenue) OVER (ORDER BY month)) / 
     LAG(revenue) OVER (ORDER BY month)) * 100 as growth_percentage
FROM monthly_revenue;

-- Detecting Value Changes
SELECT 
    customer_id,
    status,
    status_date,
    LAG(status) OVER (
        PARTITION BY customer_id 
        ORDER BY status_date
    ) as previous_status,
    CASE 
        WHEN status != LAG(status) OVER (
            PARTITION BY customer_id 
            ORDER BY status_date
        ) THEN 'Changed'
        ELSE 'No Change'
    END as status_change
FROM customer_status;
```

### 2. Gap Analysis
```sql
-- Finding Gaps in Sequences
SELECT 
    date,
    LEAD(date) OVER (ORDER BY date) as next_date,
    DATEDIFF(
        LEAD(date) OVER (ORDER BY date),
        date
    ) - 1 as gap_days
FROM dates
WHERE 
    DATEDIFF(
        LEAD(date) OVER (ORDER BY date),
        date
    ) - 1 > 0;

-- Identifying Missing Values
SELECT 
    t1.number + 1 as start_gap,
    MIN(t2.number) - 1 as end_gap
FROM sequence t1
JOIN sequence t2 ON t1.number < t2.number
WHERE t2.number > t1.number + 1
GROUP BY t1.number;
```

### 3. Default Values and Coalescing Multiple Columns
```sql
-- Complex Contact Preference
SELECT 
    customer_id,
    COALESCE(
        NULLIF(primary_contact, 'none'),
        NULLIF(mobile_phone, 'none'),
        NULLIF(work_phone, 'none'),
        NULLIF(email, 'none'),
        'No contact method available'
    ) as best_contact_method
FROM customer_contacts;

-- Handling Multiple Payment Methods
SELECT 
    order_id,
    COALESCE(
        credit_card_amount,
        debit_card_amount,
        cash_amount,
        gift_card_amount,
        0
    ) as payment_amount,
    CASE 
        WHEN credit_card_amount IS NOT NULL THEN 'Credit Card'
        WHEN debit_card_amount IS NOT NULL THEN 'Debit Card'
        WHEN cash_amount IS NOT NULL THEN 'Cash'
        WHEN gift_card_amount IS NOT NULL THEN 'Gift Card'
        ELSE 'No Payment'
    END as payment_method
FROM order_payments;
```

## Performance Considerations

1. **Indexing for Window Functions**
   - Create indexes on columns used in PARTITION BY and ORDER BY clauses
   - Consider covering indexes for frequently used queries

2. **NULL Handling**
   - COALESCE can be more efficient than multiple CASE statements
   - Use appropriate indexes on columns commonly checked for NULL

3. **Window Function Optimization**
   - Use specific window frame clauses when possible
   - Minimize the number of window functions in a single query
   - Consider materializing results for frequently used window calculations

## Best Practices

1. **NULL Handling**
   - Always use COALESCE/IFNULL for consistent default values
   - Document expected NULL behavior in comments
   - Consider business rules when choosing default values

2. **Window Functions**
   - Always specify ORDER BY in window functions
   - Use appropriate PARTITION BY clauses for better performance
   - Consider the impact of NULL values in window function calculations

3. **General**
   - Use meaningful alias names
   - Document complex window function logic
   - Test with various data scenarios including NULL values
   - Consider edge cases in your data
