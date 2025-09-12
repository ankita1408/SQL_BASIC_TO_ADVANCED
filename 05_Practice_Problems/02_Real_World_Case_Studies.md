# Real-World SQL Case Studies

## E-commerce Database

### 1. Order Processing System
```sql
-- Tables
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    address TEXT
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    description TEXT,
    price DECIMAL(10,2),
    stock_quantity INT
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date TIMESTAMP,
    status VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Common Queries

-- 1. Sales Analysis
SELECT 
    p.name as product_name,
    SUM(oi.quantity) as total_quantity,
    SUM(oi.quantity * oi.unit_price) as total_revenue
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
GROUP BY p.product_id, p.name
ORDER BY total_revenue DESC;

-- 2. Customer Purchase History
SELECT 
    c.name,
    COUNT(DISTINCT o.order_id) as total_orders,
    SUM(oi.quantity * oi.unit_price) as total_spent,
    AVG(oi.quantity * oi.unit_price) as avg_order_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.name;

-- 3. Inventory Management
SELECT 
    p.name,
    p.stock_quantity,
    SUM(oi.quantity) as ordered_quantity
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 7 DAY)
GROUP BY p.product_id, p.name, p.stock_quantity
HAVING p.stock_quantity < COALESCE(SUM(oi.quantity), 0);
```

## Social Media Platform

### 1. User Interaction System
```sql
-- Tables
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100),
    created_at TIMESTAMP
);

CREATE TABLE posts (
    post_id INT PRIMARY KEY,
    user_id INT,
    content TEXT,
    posted_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE comments (
    comment_id INT PRIMARY KEY,
    post_id INT,
    user_id INT,
    content TEXT,
    commented_at TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE likes (
    user_id INT,
    post_id INT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id)
);

-- Common Queries

-- 1. User Engagement Analysis
SELECT 
    u.username,
    COUNT(DISTINCT p.post_id) as posts,
    COUNT(DISTINCT c.comment_id) as comments,
    COUNT(DISTINCT l.post_id) as likes_given
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
LEFT JOIN comments c ON u.user_id = c.user_id
LEFT JOIN likes l ON u.user_id = l.user_id
GROUP BY u.user_id, u.username;

-- 2. Popular Posts
SELECT 
    p.post_id,
    p.content,
    u.username,
    COUNT(DISTINCT l.user_id) as like_count,
    COUNT(DISTINCT c.comment_id) as comment_count
FROM posts p
JOIN users u ON p.user_id = u.user_id
LEFT JOIN likes l ON p.post_id = l.post_id
LEFT JOIN comments c ON p.post_id = c.post_id
GROUP BY p.post_id, p.content, u.username
ORDER BY like_count DESC, comment_count DESC;

-- 3. User Activity Timeline
SELECT 
    u.username,
    COALESCE(p.content, c.content) as activity_content,
    CASE 
        WHEN p.post_id IS NOT NULL THEN 'post'
        ELSE 'comment'
    END as activity_type,
    COALESCE(p.posted_at, c.commented_at) as activity_time
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
LEFT JOIN comments c ON u.user_id = c.user_id
WHERE COALESCE(p.posted_at, c.commented_at) >= DATE_SUB(CURRENT_DATE, INTERVAL 7 DAY)
ORDER BY activity_time DESC;
```

## Healthcare System

### 1. Patient Records Management
```sql
-- Tables
CREATE TABLE patients (
    patient_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    date_of_birth DATE,
    gender CHAR(1)
);

CREATE TABLE doctors (
    doctor_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    specialty VARCHAR(100)
);

CREATE TABLE appointments (
    appointment_id INT PRIMARY KEY,
    patient_id INT,
    doctor_id INT,
    appointment_date TIMESTAMP,
    status VARCHAR(20),
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id)
);

CREATE TABLE medical_records (
    record_id INT PRIMARY KEY,
    patient_id INT,
    doctor_id INT,
    diagnosis TEXT,
    prescription TEXT,
    visit_date DATE,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id)
);

-- Common Queries

-- 1. Patient Visit History
SELECT 
    CONCAT(p.first_name, ' ', p.last_name) as patient_name,
    CONCAT(d.first_name, ' ', d.last_name) as doctor_name,
    d.specialty,
    mr.diagnosis,
    mr.prescription,
    mr.visit_date
FROM patients p
JOIN medical_records mr ON p.patient_id = mr.patient_id
JOIN doctors d ON mr.doctor_id = d.doctor_id
WHERE p.patient_id = 123
ORDER BY mr.visit_date DESC;

-- 2. Doctor's Schedule
SELECT 
    CONCAT(d.first_name, ' ', d.last_name) as doctor_name,
    COUNT(a.appointment_id) as total_appointments,
    STRING_AGG(
        CONCAT(
            p.first_name, ' ', 
            p.last_name, ' - ', 
            FORMAT(a.appointment_date, 'HH:mm')
        ),
        '; '
    ) as appointment_details
FROM doctors d
JOIN appointments a ON d.doctor_id = a.doctor_id
JOIN patients p ON a.patient_id = p.patient_id
WHERE DATE(a.appointment_date) = CURRENT_DATE
GROUP BY d.doctor_id, d.first_name, d.last_name;

-- 3. Patient Statistics
SELECT 
    d.specialty,
    COUNT(DISTINCT mr.patient_id) as total_patients,
    COUNT(mr.record_id) as total_visits,
    AVG(DATEDIFF(DAY, LAG(mr.visit_date) OVER (
        PARTITION BY mr.patient_id 
        ORDER BY mr.visit_date
    ), mr.visit_date)) as avg_days_between_visits
FROM doctors d
JOIN medical_records mr ON d.doctor_id = mr.doctor_id
GROUP BY d.specialty;
```

## Best Practices for Real-World Applications

1. **Data Integrity**
   - Use appropriate constraints
   - Implement proper relationships
   - Handle NULL values correctly
   - Maintain referential integrity

2. **Performance Optimization**
   - Create necessary indexes
   - Write efficient queries
   - Use appropriate data types
   - Regular maintenance

3. **Security Considerations**
   - Implement proper access control
   - Use parameterized queries
   - Regular security audits
   - Data encryption when necessary

4. **Scalability**
   - Proper database design
   - Efficient indexing strategy
   - Query optimization
   - Partitioning when necessary
