# Advanced Relationships and Data Modeling

## Database Relationships

### 1. One-to-One Relationships
```sql
-- Example: Employee and Employee_Detail tables
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);

CREATE TABLE employee_details (
    employee_id INT PRIMARY KEY,
    ssn VARCHAR(11),
    birth_date DATE,
    address TEXT,
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);

-- Querying one-to-one relationship
SELECT 
    e.first_name,
    e.last_name,
    ed.birth_date,
    ed.address
FROM employees e
INNER JOIN employee_details ed ON e.employee_id = ed.employee_id;
```

### 2. One-to-Many Relationships
```sql
-- Example: Department and Employees
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

-- Querying one-to-many relationship
SELECT 
    d.department_name,
    COUNT(e.employee_id) as employee_count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;
```

### 3. Many-to-Many Relationships
```sql
-- Example: Students and Courses
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

-- Querying many-to-many relationship
SELECT 
    s.student_name,
    GROUP_CONCAT(c.course_name) as enrolled_courses
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
GROUP BY s.student_name;
```

## Advanced Data Modeling Concepts

### 1. Inheritance
```sql
-- Example: Vehicle hierarchy
CREATE TABLE vehicles (
    vehicle_id INT PRIMARY KEY,
    manufacturer VARCHAR(50),
    model VARCHAR(50),
    year INT
);

CREATE TABLE cars (
    vehicle_id INT PRIMARY KEY,
    num_doors INT,
    body_style VARCHAR(50),
    FOREIGN KEY (vehicle_id) REFERENCES vehicles(vehicle_id)
);

CREATE TABLE trucks (
    vehicle_id INT PRIMARY KEY,
    cargo_capacity DECIMAL(10,2),
    num_axles INT,
    FOREIGN KEY (vehicle_id) REFERENCES vehicles(vehicle_id)
);
```

### 2. Polymorphic Associations
```sql
-- Example: Comments on different entities
CREATE TABLE comments (
    comment_id INT PRIMARY KEY,
    content TEXT,
    commentable_id INT,
    commentable_type VARCHAR(50)
);

-- Querying polymorphic associations
SELECT 
    CASE 
        WHEN c.commentable_type = 'post' THEN p.title
        WHEN c.commentable_type = 'product' THEN pr.name
    END as item_name,
    c.content as comment
FROM comments c
LEFT JOIN posts p ON c.commentable_id = p.id 
    AND c.commentable_type = 'post'
LEFT JOIN products pr ON c.commentable_id = pr.id 
    AND c.commentable_type = 'product';
```

### 3. Recursive Relationships
```sql
-- Example: Employee hierarchy
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

-- Query organization hierarchy
WITH RECURSIVE emp_hierarchy AS (
    -- Base case
    SELECT 
        employee_id,
        first_name,
        last_name,
        manager_id,
        0 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        eh.level + 1
    FROM employees e
    INNER JOIN emp_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM emp_hierarchy;
```

## Best Practices

### 1. Referential Integrity
```sql
-- Example: ON DELETE and ON UPDATE actions
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

### 2. Indexing Relationships
```sql
-- Example: Index foreign keys
CREATE INDEX idx_employee_dept 
ON employees(department_id);

CREATE INDEX idx_enrollment_composite 
ON enrollments(student_id, course_id);
```

## Practice Problems

### Problem 1: Complex Hierarchy
```sql
-- Create and query a category hierarchy
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50),
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES categories(category_id)
);

-- Query full category path
WITH RECURSIVE category_path AS (
    SELECT 
        category_id,
        category_name,
        parent_id,
        category_name as path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    SELECT 
        c.category_id,
        c.category_name,
        c.parent_id,
        CONCAT(cp.path, ' > ', c.category_name)
    FROM categories c
    INNER JOIN category_path cp ON c.parent_id = cp.category_id
)
SELECT * FROM category_path;
```

## Interview Questions

1. **How do you choose between different types of relationships?**
   - Based on business requirements
   - Data integrity needs
   - Performance considerations
   - Scalability requirements

2. **Explain the trade-offs of denormalization**
   - Improved query performance
   - Data redundancy
   - Update anomalies
   - Storage requirements

3. **How do you handle data consistency in relationships?**
   - Foreign key constraints
   - Cascading actions
   - Transaction management
   - Application-level validation
