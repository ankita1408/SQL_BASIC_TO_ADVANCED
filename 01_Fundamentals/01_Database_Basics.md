# Database Basics

## What is a Database?

A database is an organized collection of data stored and accessed electronically. Databases are designed to manage large collections of information by storing, retrieving, and managing data.

## Types of Databases

1. **Relational Databases (RDBMS)**
   - MySQL
   - PostgreSQL
   - Oracle
   - SQL Server
   - SQLite

2. **NoSQL Databases**
   - Document stores (MongoDB)
   - Key-value stores (Redis)
   - Wide-column stores (Cassandra)
   - Graph databases (Neo4j)

## Key Concepts

### 1. Tables (Relations)
- Basic unit of storage in a relational database
- Composed of rows and columns
- Example:
```sql
CREATE TABLE employees (
    id INT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2)
);
```

### 2. Records (Tuples)
- Individual entries in a table
- Example:
```sql
INSERT INTO employees VALUES (1, 'John Doe', 'IT', 75000.00);
```

### 3. Fields (Attributes)
- Columns in a table
- Each field has a specific data type
- Common data types:
  - INTEGER
  - VARCHAR
  - DATE
  - DECIMAL
  - BOOLEAN

### 4. Keys
1. **Primary Key**
   - Unique identifier for each record
   ```sql
   CREATE TABLE users (
       user_id INT PRIMARY KEY,
       username VARCHAR(50)
   );
   ```

2. **Foreign Key**
   - References primary key of another table
   ```sql
   CREATE TABLE orders (
       order_id INT PRIMARY KEY,
       user_id INT,
       FOREIGN KEY (user_id) REFERENCES users(user_id)
   );
   ```

## Database Properties (ACID)

1. **Atomicity**
   - Transactions are all or nothing
   - Example:
   ```sql
   BEGIN TRANSACTION;
       UPDATE accounts SET balance = balance - 100 WHERE id = 1;
       UPDATE accounts SET balance = balance + 100 WHERE id = 2;
   COMMIT;
   ```

2. **Consistency**
   - Database remains in a valid state
   - Enforced through:
     - Constraints
     - Triggers
     - Foreign keys

3. **Isolation**
   - Transactions are independent
   - Levels:
     - READ UNCOMMITTED
     - READ COMMITTED
     - REPEATABLE READ
     - SERIALIZABLE

4. **Durability**
   - Committed transactions are permanent
   - Maintained through:
     - Transaction logs
     - Database backups

## Practice Exercise

Create a simple database schema for a library:

```sql
-- Create tables for a library system
CREATE TABLE books (
    book_id INT PRIMARY KEY,
    title VARCHAR(200),
    author VARCHAR(100),
    isbn VARCHAR(13),
    published_date DATE,
    available BOOLEAN
);

CREATE TABLE members (
    member_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    join_date DATE,
    active BOOLEAN
);

CREATE TABLE loans (
    loan_id INT PRIMARY KEY,
    book_id INT,
    member_id INT,
    loan_date DATE,
    due_date DATE,
    returned_date DATE,
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    FOREIGN KEY (member_id) REFERENCES members(member_id)
);
```

## Interview Questions

1. **What is the difference between a database and a spreadsheet?**
   - Databases handle large amounts of data efficiently
   - Support multiple users simultaneously
   - Ensure data integrity
   - Provide better security

2. **Explain the importance of normalization.**
   - Reduces data redundancy
   - Ensures data integrity
   - Makes database maintenance easier
   - Optimizes storage

3. **What is the purpose of a primary key?**
   - Uniquely identifies each record
   - Ensures data integrity
   - Helps establish relationships between tables
   - Improves query performance
