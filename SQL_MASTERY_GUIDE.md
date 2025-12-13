# SQL Mastery Guide: From Beginner to Expert

## Table of Contents
1. [Introduction](#introduction)
2. [Basic SQL Commands](#basic-sql-commands)
3. [Intermediate SQL Commands](#intermediate-sql-commands)
4. [Advanced SQL Commands](#advanced-sql-commands)
5. [Expert-Level SQL](#expert-level-sql)
6. [Real-World Examples](#real-world-examples)
7. [Best Practices](#best-practices)

---

## Introduction

This guide covers all essential SQL commands needed to perform simple to advanced queries. Each section includes:
- Command syntax
- Purpose and use cases
- Real-world examples
- Best practices

---

## Basic SQL Commands

### 1. SELECT - Retrieve Data

**Syntax:**
```sql
SELECT column1, column2, ...
FROM table_name;
```

**Real-World Example: E-commerce Store**
```sql
-- Get all products
SELECT * FROM products;

-- Get specific columns
SELECT product_name, price, stock_quantity 
FROM products;

-- Get unique categories
SELECT DISTINCT category 
FROM products;
```

### 2. WHERE - Filter Data

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

**Real-World Example: Customer Database**
```sql
-- Find customers from a specific city
SELECT customer_name, email, phone
FROM customers
WHERE city = 'New York';

-- Find products under $50
SELECT product_name, price
FROM products
WHERE price < 50;

-- Multiple conditions
SELECT customer_name, email
FROM customers
WHERE city = 'Mumbai' AND age >= 25;
```

### 3. ORDER BY - Sort Results

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 [ASC|DESC];
```

**Real-World Example: Employee Management**
```sql
-- Sort employees by salary (highest first)
SELECT employee_name, salary, department
FROM employees
ORDER BY salary DESC;

-- Sort by multiple columns
SELECT employee_name, department, hire_date
FROM employees
ORDER BY department ASC, hire_date DESC;
```

### 4. INSERT - Add New Data

**Syntax:**
```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

**Real-World Example: User Registration**
```sql
-- Add a new user
INSERT INTO users (username, email, password_hash, created_at)
VALUES ('john_doe', 'john@example.com', 'hashed_password', NOW());

-- Insert multiple rows
INSERT INTO orders (customer_id, order_date, total_amount, status)
VALUES 
    (101, '2025-12-13', 1500.00, 'pending'),
    (102, '2025-12-13', 2300.50, 'confirmed'),
    (103, '2025-12-13', 890.00, 'shipped');
```

### 5. UPDATE - Modify Existing Data

**Syntax:**
```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

**Real-World Example: Inventory Management**
```sql
-- Update product stock after sale
UPDATE products
SET stock_quantity = stock_quantity - 5
WHERE product_id = 1001;

-- Update customer information
UPDATE customers
SET email = 'newemail@example.com', 
    phone = '+91-9876543210',
    updated_at = NOW()
WHERE customer_id = 250;

-- Bulk update - apply discount
UPDATE products
SET price = price * 0.9
WHERE category = 'Electronics' AND stock_quantity > 100;
```

### 6. DELETE - Remove Data

**Syntax:**
```sql
DELETE FROM table_name
WHERE condition;
```

**Real-World Example: Data Cleanup**
```sql
-- Delete inactive users
DELETE FROM users
WHERE last_login < '2024-01-01' AND status = 'inactive';

-- Delete cancelled orders
DELETE FROM orders
WHERE status = 'cancelled' AND order_date < '2024-01-01';

-- ⚠️ WARNING: This deletes ALL rows
DELETE FROM temp_data;
```

### 7. LIMIT - Restrict Number of Results

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
LIMIT number;
```

**Real-World Example: Pagination**
```sql
-- Get top 10 best-selling products
SELECT product_name, total_sales
FROM products
ORDER BY total_sales DESC
LIMIT 10;

-- Pagination: Get records 21-30
SELECT * FROM products
ORDER BY product_id
LIMIT 10 OFFSET 20;
```

---

## Intermediate SQL Commands

### 8. JOINS - Combine Data from Multiple Tables

#### INNER JOIN
**Syntax:**
```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

**Real-World Example: Order Management System**
```sql
-- Get order details with customer information
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    c.email,
    o.total_amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;

-- Get product sales with category
SELECT 
    p.product_name,
    c.category_name,
    p.price,
    p.stock_quantity
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id;
```

#### LEFT JOIN
**Real-World Example: Customer Orders Report**
```sql
-- Get all customers with their orders (including customers with no orders)
SELECT 
    c.customer_name,
    c.email,
    o.order_id,
    o.order_date,
    o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- Find customers who haven't placed orders
SELECT 
    c.customer_name,
    c.email,
    c.registration_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

#### RIGHT JOIN
**Real-World Example: Product Inventory**
```sql
-- Get all products with their suppliers (including products without suppliers)
SELECT 
    p.product_name,
    s.supplier_name,
    s.contact_email
FROM suppliers s
RIGHT JOIN products p ON s.supplier_id = p.supplier_id;
```

#### FULL OUTER JOIN
**Real-World Example: Employee-Department Matching**
```sql
-- Get all employees and departments (including unassigned)
SELECT 
    e.employee_name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;
```

### 9. GROUP BY - Aggregate Data

**Syntax:**
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;
```

**Real-World Example: Sales Analytics**
```sql
-- Total sales by category
SELECT 
    category,
    COUNT(*) as total_products,
    SUM(stock_quantity) as total_stock,
    AVG(price) as average_price
FROM products
GROUP BY category;

-- Monthly revenue report
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') as month,
    COUNT(order_id) as total_orders,
    SUM(total_amount) as monthly_revenue
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month DESC;

-- Customer purchase frequency
SELECT 
    customer_id,
    COUNT(order_id) as total_orders,
    SUM(total_amount) as total_spent,
    AVG(total_amount) as average_order_value
FROM orders
GROUP BY customer_id
HAVING total_orders > 5
ORDER BY total_spent DESC;
```

### 10. HAVING - Filter Grouped Data

**Syntax:**
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition;
```

**Real-World Example: Business Intelligence**
```sql
-- Find high-value customers (spent more than $10,000)
SELECT 
    c.customer_name,
    COUNT(o.order_id) as order_count,
    SUM(o.total_amount) as total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING SUM(o.total_amount) > 10000
ORDER BY total_spent DESC;

-- Products with low average ratings
SELECT 
    product_id,
    product_name,
    AVG(rating) as avg_rating,
    COUNT(review_id) as review_count
FROM products p
JOIN reviews r ON p.product_id = r.product_id
GROUP BY p.product_id, p.product_name
HAVING AVG(rating) < 3.0 AND COUNT(review_id) >= 10;
```

### 11. Aggregate Functions

**Real-World Example: Dashboard Statistics**
```sql
-- COUNT: Total number of records
SELECT COUNT(*) as total_customers FROM customers;
SELECT COUNT(DISTINCT city) as unique_cities FROM customers;

-- SUM: Total revenue
SELECT SUM(total_amount) as total_revenue FROM orders;

-- AVG: Average order value
SELECT AVG(total_amount) as average_order_value FROM orders;

-- MIN/MAX: Price range
SELECT 
    MIN(price) as lowest_price,
    MAX(price) as highest_price,
    MAX(price) - MIN(price) as price_range
FROM products;

-- Combined analytics
SELECT 
    category,
    COUNT(*) as product_count,
    MIN(price) as min_price,
    MAX(price) as max_price,
    AVG(price) as avg_price,
    SUM(stock_quantity) as total_stock
FROM products
GROUP BY category;
```

### 12. LIKE - Pattern Matching

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE column LIKE pattern;
```

**Real-World Example: Search Functionality**
```sql
-- Search customers by name (starts with 'John')
SELECT * FROM customers
WHERE customer_name LIKE 'John%';

-- Search products containing 'phone'
SELECT product_name, price
FROM products
WHERE product_name LIKE '%phone%';

-- Search emails from Gmail
SELECT customer_name, email
FROM customers
WHERE email LIKE '%@gmail.com';

-- Search with specific pattern (3 characters, then 'son')
SELECT * FROM customers
WHERE customer_name LIKE '___son';
```

### 13. IN - Multiple Values

**Real-World Example: Filtering**
```sql
-- Get orders from specific cities
SELECT * FROM customers
WHERE city IN ('Mumbai', 'Delhi', 'Bangalore', 'Chennai');

-- Get products in specific categories
SELECT product_name, category, price
FROM products
WHERE category IN ('Electronics', 'Computers', 'Mobile');

-- Exclude specific statuses
SELECT order_id, customer_id, status
FROM orders
WHERE status NOT IN ('cancelled', 'refunded');
```

### 14. BETWEEN - Range Queries

**Real-World Example: Date and Price Ranges**
```sql
-- Orders in a date range
SELECT order_id, order_date, total_amount
FROM orders
WHERE order_date BETWEEN '2025-01-01' AND '2025-12-31';

-- Products in price range
SELECT product_name, price
FROM products
WHERE price BETWEEN 1000 AND 5000;

-- Employees hired in specific years
SELECT employee_name, hire_date, salary
FROM employees
WHERE YEAR(hire_date) BETWEEN 2020 AND 2023;
```

### 15. CASE - Conditional Logic

**Syntax:**
```sql
SELECT column1,
    CASE 
        WHEN condition1 THEN result1
        WHEN condition2 THEN result2
        ELSE result3
    END as alias
FROM table_name;
```

**Real-World Example: Data Categorization**
```sql
-- Categorize products by price
SELECT 
    product_name,
    price,
    CASE 
        WHEN price < 1000 THEN 'Budget'
        WHEN price BETWEEN 1000 AND 5000 THEN 'Mid-Range'
        WHEN price > 5000 THEN 'Premium'
    END as price_category
FROM products;

-- Customer segmentation
SELECT 
    customer_name,
    total_orders,
    total_spent,
    CASE 
        WHEN total_spent > 50000 THEN 'VIP'
        WHEN total_spent > 20000 THEN 'Gold'
        WHEN total_spent > 5000 THEN 'Silver'
        ELSE 'Bronze'
    END as customer_tier
FROM (
    SELECT 
        c.customer_id,
        c.customer_name,
        COUNT(o.order_id) as total_orders,
        COALESCE(SUM(o.total_amount), 0) as total_spent
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
) customer_stats;

-- Order status labels
SELECT 
    order_id,
    order_date,
    status,
    CASE status
        WHEN 'pending' THEN '⏳ Pending'
        WHEN 'confirmed' THEN '✅ Confirmed'
        WHEN 'shipped' THEN '🚚 Shipped'
        WHEN 'delivered' THEN '📦 Delivered'
        WHEN 'cancelled' THEN '❌ Cancelled'
        ELSE '❓ Unknown'
    END as status_label
FROM orders;
```

---

## Advanced SQL Commands

### 16. Subqueries

**Real-World Example: Complex Filtering**
```sql
-- Find customers who spent more than average
SELECT customer_name, email
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING SUM(total_amount) > (
        SELECT AVG(total_spent)
        FROM (
            SELECT SUM(total_amount) as total_spent
            FROM orders
            GROUP BY customer_id
        ) avg_calc
    )
);

-- Products more expensive than category average
SELECT 
    p1.product_name,
    p1.category,
    p1.price,
    (SELECT AVG(price) FROM products p2 WHERE p2.category = p1.category) as category_avg
FROM products p1
WHERE p1.price > (
    SELECT AVG(price)
    FROM products p2
    WHERE p2.category = p1.category
);

-- Latest order for each customer
SELECT 
    c.customer_name,
    o.order_date,
    o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date = (
    SELECT MAX(order_date)
    FROM orders o2
    WHERE o2.customer_id = c.customer_id
);
```

### 17. Common Table Expressions (CTE)

**Syntax:**
```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

**Real-World Example: Readable Complex Queries**
```sql
-- Sales analysis with CTEs
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') as month,
        SUM(total_amount) as revenue,
        COUNT(order_id) as order_count
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
),
sales_growth AS (
    SELECT 
        month,
        revenue,
        order_count,
        LAG(revenue) OVER (ORDER BY month) as prev_month_revenue,
        revenue - LAG(revenue) OVER (ORDER BY month) as revenue_change
    FROM monthly_sales
)
SELECT 
    month,
    revenue,
    order_count,
    prev_month_revenue,
    revenue_change,
    ROUND((revenue_change / prev_month_revenue) * 100, 2) as growth_percentage
FROM sales_growth
WHERE prev_month_revenue IS NOT NULL
ORDER BY month DESC;

-- Customer lifetime value calculation
WITH customer_orders AS (
    SELECT 
        customer_id,
        COUNT(order_id) as total_orders,
        SUM(total_amount) as total_spent,
        MIN(order_date) as first_order,
        MAX(order_date) as last_order
    FROM orders
    GROUP BY customer_id
),
customer_metrics AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        c.email,
        co.total_orders,
        co.total_spent,
        co.first_order,
        co.last_order,
        DATEDIFF(co.last_order, co.first_order) as customer_lifetime_days,
        co.total_spent / NULLIF(co.total_orders, 0) as avg_order_value
    FROM customers c
    JOIN customer_orders co ON c.customer_id = co.customer_id
)
SELECT 
    customer_name,
    email,
    total_orders,
    total_spent,
    avg_order_value,
    customer_lifetime_days,
    CASE 
        WHEN customer_lifetime_days > 365 THEN 'Loyal'
        WHEN customer_lifetime_days > 180 THEN 'Regular'
        ELSE 'New'
    END as customer_type
FROM customer_metrics
ORDER BY total_spent DESC;
```

### 18. Window Functions

**Real-World Example: Rankings and Running Totals**
```sql
-- Rank products by sales within each category
SELECT 
    product_name,
    category,
    total_sales,
    RANK() OVER (PARTITION BY category ORDER BY total_sales DESC) as category_rank,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY total_sales DESC) as dense_rank,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY total_sales DESC) as row_num
FROM products;

-- Running total of daily sales
SELECT 
    order_date,
    daily_revenue,
    SUM(daily_revenue) OVER (ORDER BY order_date) as running_total,
    AVG(daily_revenue) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as moving_avg_7days
FROM (
    SELECT 
        DATE(order_date) as order_date,
        SUM(total_amount) as daily_revenue
    FROM orders
    GROUP BY DATE(order_date)
) daily_sales;

-- Employee salary percentile
SELECT 
    employee_name,
    department,
    salary,
    PERCENT_RANK() OVER (PARTITION BY department ORDER BY salary) as salary_percentile,
    NTILE(4) OVER (PARTITION BY department ORDER BY salary) as salary_quartile
FROM employees;

-- Compare with previous/next values
SELECT 
    product_name,
    price,
    LAG(price) OVER (ORDER BY price) as previous_price,
    LEAD(price) OVER (ORDER BY price) as next_price,
    price - LAG(price) OVER (ORDER BY price) as price_diff
FROM products;
```

### 19. UNION / UNION ALL

**Real-World Example: Combining Results**
```sql
-- Combine active and archived customers
SELECT customer_id, customer_name, email, 'Active' as status
FROM customers
WHERE status = 'active'
UNION ALL
SELECT customer_id, customer_name, email, 'Archived' as status
FROM archived_customers;

-- All transactions (orders + refunds)
SELECT 
    order_id as transaction_id,
    customer_id,
    order_date as transaction_date,
    total_amount,
    'Order' as transaction_type
FROM orders
UNION ALL
SELECT 
    refund_id as transaction_id,
    customer_id,
    refund_date as transaction_date,
    -refund_amount as total_amount,
    'Refund' as transaction_type
FROM refunds
ORDER BY transaction_date DESC;
```

### 20. EXISTS / NOT EXISTS

**Real-World Example: Conditional Checks**
```sql
-- Find customers who have placed orders
SELECT customer_name, email
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- Find products never ordered
SELECT product_name, category, price
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.product_id
);

-- Customers who ordered in 2025 but not in 2024
SELECT customer_name, email
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
    AND YEAR(o.order_date) = 2025
)
AND NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
    AND YEAR(o.order_date) = 2024
);
```

---

## Expert-Level SQL

### 21. Recursive CTEs

**Real-World Example: Organizational Hierarchy**
```sql
-- Employee hierarchy (manager-employee chain)
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Top-level managers
    SELECT 
        employee_id,
        employee_name,
        manager_id,
        1 as level,
        CAST(employee_name AS CHAR(1000)) as hierarchy_path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Employees reporting to previous level
    SELECT 
        e.employee_id,
        e.employee_name,
        e.manager_id,
        eh.level + 1,
        CONCAT(eh.hierarchy_path, ' > ', e.employee_name)
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT 
    employee_id,
    employee_name,
    level,
    hierarchy_path
FROM employee_hierarchy
ORDER BY level, employee_name;

-- Category tree (parent-child categories)
WITH RECURSIVE category_tree AS (
    SELECT 
        category_id,
        category_name,
        parent_category_id,
        0 as depth,
        CAST(category_name AS CHAR(500)) as path
    FROM categories
    WHERE parent_category_id IS NULL
    
    UNION ALL
    
    SELECT 
        c.category_id,
        c.category_name,
        c.parent_category_id,
        ct.depth + 1,
        CONCAT(ct.path, ' / ', c.category_name)
    FROM categories c
    JOIN category_tree ct ON c.parent_category_id = ct.category_id
)
SELECT * FROM category_tree ORDER BY path;
```

### 22. Pivot Tables (Dynamic Reporting)

**Real-World Example: Sales by Month and Category**
```sql
-- Pivot: Monthly sales by category
SELECT 
    category,
    SUM(CASE WHEN MONTH(order_date) = 1 THEN total_amount ELSE 0 END) as Jan,
    SUM(CASE WHEN MONTH(order_date) = 2 THEN total_amount ELSE 0 END) as Feb,
    SUM(CASE WHEN MONTH(order_date) = 3 THEN total_amount ELSE 0 END) as Mar,
    SUM(CASE WHEN MONTH(order_date) = 4 THEN total_amount ELSE 0 END) as Apr,
    SUM(CASE WHEN MONTH(order_date) = 5 THEN total_amount ELSE 0 END) as May,
    SUM(CASE WHEN MONTH(order_date) = 6 THEN total_amount ELSE 0 END) as Jun,
    SUM(CASE WHEN MONTH(order_date) = 7 THEN total_amount ELSE 0 END) as Jul,
    SUM(CASE WHEN MONTH(order_date) = 8 THEN total_amount ELSE 0 END) as Aug,
    SUM(CASE WHEN MONTH(order_date) = 9 THEN total_amount ELSE 0 END) as Sep,
    SUM(CASE WHEN MONTH(order_date) = 10 THEN total_amount ELSE 0 END) as Oct,
    SUM(CASE WHEN MONTH(order_date) = 11 THEN total_amount ELSE 0 END) as Nov,
    SUM(CASE WHEN MONTH(order_date) = 12 THEN total_amount ELSE 0 END) as Dec
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE YEAR(order_date) = 2025
GROUP BY category;
```

### 23. Advanced Indexing Strategies

**Real-World Example: Performance Optimization**
```sql
-- Create indexes for common queries
CREATE INDEX idx_customers_email ON customers(email);
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_products_category_price ON products(category, price);
CREATE INDEX idx_order_items_product ON order_items(product_id);

-- Composite index for complex queries
CREATE INDEX idx_orders_status_date ON orders(status, order_date DESC);

-- Full-text search index
CREATE FULLTEXT INDEX idx_products_search ON products(product_name, description);

-- Use full-text search
SELECT product_name, description
FROM products
WHERE MATCH(product_name, description) AGAINST ('smartphone camera' IN NATURAL LANGUAGE MODE);
```

### 24. Transactions and ACID Properties

**Real-World Example: Order Processing**
```sql
-- Start transaction for order placement
START TRANSACTION;

-- Insert order
INSERT INTO orders (customer_id, order_date, total_amount, status)
VALUES (101, NOW(), 2500.00, 'pending');

SET @order_id = LAST_INSERT_ID();

-- Insert order items
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES 
    (@order_id, 1001, 2, 500.00),
    (@order_id, 1005, 1, 1500.00);

-- Update product stock
UPDATE products
SET stock_quantity = stock_quantity - 2
WHERE product_id = 1001;

UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 1005;

-- Check if stock is sufficient
SELECT 
    product_id,
    stock_quantity
FROM products
WHERE product_id IN (1001, 1005) AND stock_quantity < 0;

-- If stock is negative, rollback; otherwise commit
-- ROLLBACK; -- If error
COMMIT; -- If success
```

### 25. Stored Procedures

**Real-World Example: Reusable Business Logic**
```sql
-- Create procedure for customer registration
DELIMITER //
CREATE PROCEDURE RegisterCustomer(
    IN p_name VARCHAR(100),
    IN p_email VARCHAR(100),
    IN p_phone VARCHAR(20),
    IN p_city VARCHAR(50),
    OUT p_customer_id INT
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_customer_id = -1;
    END;
    
    START TRANSACTION;
    
    -- Check if email already exists
    IF EXISTS (SELECT 1 FROM customers WHERE email = p_email) THEN
        SET p_customer_id = -2;
        ROLLBACK;
    ELSE
        INSERT INTO customers (customer_name, email, phone, city, registration_date, status)
        VALUES (p_name, p_email, p_phone, p_city, NOW(), 'active');
        
        SET p_customer_id = LAST_INSERT_ID();
        COMMIT;
    END IF;
END //
DELIMITER ;

-- Use the procedure
CALL RegisterCustomer('John Doe', 'john@example.com', '+91-9876543210', 'Mumbai', @new_id);
SELECT @new_id as customer_id;
```

### 26. Triggers

**Real-World Example: Audit Trail**
```sql
-- Create audit table
CREATE TABLE order_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    action VARCHAR(20),
    old_status VARCHAR(50),
    new_status VARCHAR(50),
    changed_by VARCHAR(100),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create trigger for order status changes
DELIMITER //
CREATE TRIGGER order_status_audit
AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
    IF OLD.status != NEW.status THEN
        INSERT INTO order_audit (order_id, action, old_status, new_status, changed_by)
        VALUES (NEW.order_id, 'UPDATE', OLD.status, NEW.status, USER());
    END IF;
END //
DELIMITER ;

-- Trigger to update stock on order
DELIMITER //
CREATE TRIGGER update_stock_on_order
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock_quantity = stock_quantity - NEW.quantity
    WHERE product_id = NEW.product_id;
END //
DELIMITER ;
```

### 27. Views

**Real-World Example: Simplified Data Access**
```sql
-- Create view for customer summary
CREATE VIEW customer_summary AS
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    c.city,
    COUNT(o.order_id) as total_orders,
    COALESCE(SUM(o.total_amount), 0) as total_spent,
    MAX(o.order_date) as last_order_date,
    CASE 
        WHEN COALESCE(SUM(o.total_amount), 0) > 50000 THEN 'VIP'
        WHEN COALESCE(SUM(o.total_amount), 0) > 20000 THEN 'Gold'
        WHEN COALESCE(SUM(o.total_amount), 0) > 5000 THEN 'Silver'
        ELSE 'Bronze'
    END as customer_tier
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.email, c.city;

-- Use the view
SELECT * FROM customer_summary WHERE customer_tier = 'VIP';

-- Materialized view concept (using table + scheduled refresh)
CREATE TABLE mv_daily_sales AS
SELECT 
    DATE(order_date) as sale_date,
    COUNT(order_id) as order_count,
    SUM(total_amount) as total_revenue,
    AVG(total_amount) as avg_order_value
FROM orders
GROUP BY DATE(order_date);

-- Refresh materialized view (run daily)
TRUNCATE TABLE mv_daily_sales;
INSERT INTO mv_daily_sales
SELECT 
    DATE(order_date) as sale_date,
    COUNT(order_id) as order_count,
    SUM(total_amount) as total_revenue,
    AVG(total_amount) as avg_order_value
FROM orders
GROUP BY DATE(order_date);
```

---

## Real-World Examples

### Example 1: E-Commerce Analytics Dashboard

```sql
-- Complete dashboard query
WITH daily_metrics AS (
    SELECT 
        DATE(order_date) as date,
        COUNT(DISTINCT customer_id) as unique_customers,
        COUNT(order_id) as total_orders,
        SUM(total_amount) as revenue,
        AVG(total_amount) as avg_order_value
    FROM orders
    WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
    GROUP BY DATE(order_date)
),
product_performance AS (
    SELECT 
        p.product_id,
        p.product_name,
        p.category,
        COUNT(oi.order_item_id) as times_ordered,
        SUM(oi.quantity) as units_sold,
        SUM(oi.quantity * oi.unit_price) as revenue
    FROM products p
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    LEFT JOIN orders o ON oi.order_id = o.order_id
    WHERE o.order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
    GROUP BY p.product_id, p.product_name, p.category
),
customer_segments AS (
    SELECT 
        CASE 
            WHEN total_spent > 50000 THEN 'VIP'
            WHEN total_spent > 20000 THEN 'Gold'
            WHEN total_spent > 5000 THEN 'Silver'
            ELSE 'Bronze'
        END as segment,
        COUNT(*) as customer_count,
        SUM(total_spent) as segment_revenue
    FROM (
        SELECT 
            customer_id,
            SUM(total_amount) as total_spent
        FROM orders
        GROUP BY customer_id
    ) customer_totals
    GROUP BY segment
)
SELECT 
    'Daily Metrics' as report_section,
    JSON_OBJECT(
        'avg_daily_revenue', (SELECT AVG(revenue) FROM daily_metrics),
        'avg_daily_orders', (SELECT AVG(total_orders) FROM daily_metrics),
        'total_30day_revenue', (SELECT SUM(revenue) FROM daily_metrics)
    ) as metrics
UNION ALL
SELECT 
    'Top Products' as report_section,
    JSON_ARRAYAGG(
        JSON_OBJECT(
            'product_name', product_name,
            'revenue', revenue,
            'units_sold', units_sold
        )
    ) as metrics
FROM (SELECT * FROM product_performance ORDER BY revenue DESC LIMIT 5) top_products
UNION ALL
SELECT 
    'Customer Segments' as report_section,
    JSON_ARRAYAGG(
        JSON_OBJECT(
            'segment', segment,
            'customer_count', customer_count,
            'revenue', segment_revenue
        )
    ) as metrics
FROM customer_segments;
```

### Example 2: Inventory Management System

```sql
-- Low stock alert with reorder suggestions
WITH inventory_status AS (
    SELECT 
        p.product_id,
        p.product_name,
        p.category,
        p.stock_quantity,
        p.reorder_level,
        p.reorder_quantity,
        COALESCE(AVG(oi.quantity), 0) as avg_daily_sales
    FROM products p
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    LEFT JOIN orders o ON oi.order_id = o.order_id
    WHERE o.order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
    GROUP BY p.product_id, p.product_name, p.category, p.stock_quantity, p.reorder_level, p.reorder_quantity
),
reorder_recommendations AS (
    SELECT 
        product_id,
        product_name,
        category,
        stock_quantity,
        reorder_level,
        avg_daily_sales,
        CEIL(stock_quantity / NULLIF(avg_daily_sales, 0)) as days_until_stockout,
        CASE 
            WHEN stock_quantity <= reorder_level THEN 'URGENT'
            WHEN stock_quantity <= reorder_level * 1.5 THEN 'WARNING'
            ELSE 'OK'
        END as status,
        GREATEST(reorder_quantity, CEIL(avg_daily_sales * 30)) as suggested_reorder_qty
    FROM inventory_status
)
SELECT 
    product_name,
    category,
    stock_quantity,
    avg_daily_sales,
    days_until_stockout,
    status,
    suggested_reorder_qty,
    suggested_reorder_qty * (SELECT AVG(price) FROM products WHERE category = reorder_recommendations.category) as estimated_cost
FROM reorder_recommendations
WHERE status IN ('URGENT', 'WARNING')
ORDER BY 
    CASE status
        WHEN 'URGENT' THEN 1
        WHEN 'WARNING' THEN 2
        ELSE 3
    END,
    days_until_stockout;
```

### Example 3: Customer Churn Analysis

```sql
-- Identify at-risk customers
WITH customer_activity AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        c.email,
        c.registration_date,
        COUNT(o.order_id) as total_orders,
        MAX(o.order_date) as last_order_date,
        DATEDIFF(CURDATE(), MAX(o.order_date)) as days_since_last_order,
        SUM(o.total_amount) as lifetime_value,
        AVG(DATEDIFF(
            o.order_date,
            LAG(o.order_date) OVER (PARTITION BY c.customer_id ORDER BY o.order_date)
        )) as avg_days_between_orders
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name, c.email, c.registration_date
),
churn_risk AS (
    SELECT 
        *,
        CASE 
            WHEN days_since_last_order > avg_days_between_orders * 3 THEN 'High Risk'
            WHEN days_since_last_order > avg_days_between_orders * 2 THEN 'Medium Risk'
            WHEN days_since_last_order > avg_days_between_orders * 1.5 THEN 'Low Risk'
            ELSE 'Active'
        END as churn_risk_level,
        CASE 
            WHEN lifetime_value > 50000 THEN 'High Value'
            WHEN lifetime_value > 20000 THEN 'Medium Value'
            ELSE 'Low Value'
        END as customer_value
    FROM customer_activity
    WHERE total_orders > 0
)
SELECT 
    customer_name,
    email,
    total_orders,
    last_order_date,
    days_since_last_order,
    ROUND(avg_days_between_orders, 1) as avg_days_between_orders,
    lifetime_value,
    churn_risk_level,
    customer_value,
    CONCAT('Send ', 
        CASE 
            WHEN churn_risk_level = 'High Risk' AND customer_value = 'High Value' THEN 'personalized offer + phone call'
            WHEN churn_risk_level = 'High Risk' THEN 'win-back email campaign'
            WHEN churn_risk_level = 'Medium Risk' THEN 'engagement email'
            ELSE 'regular newsletter'
        END
    ) as recommended_action
FROM churn_risk
WHERE churn_risk_level != 'Active'
ORDER BY 
    CASE customer_value
        WHEN 'High Value' THEN 1
        WHEN 'Medium Value' THEN 2
        ELSE 3
    END,
    CASE churn_risk_level
        WHEN 'High Risk' THEN 1
        WHEN 'Medium Risk' THEN 2
        ELSE 3
    END;
```

---

## Best Practices

### 1. Query Optimization
```sql
-- ❌ BAD: SELECT *
SELECT * FROM orders;

-- ✅ GOOD: Select only needed columns
SELECT order_id, customer_id, total_amount FROM orders;

-- ❌ BAD: No WHERE clause on large table
SELECT * FROM orders ORDER BY order_date DESC LIMIT 10;

-- ✅ GOOD: Filter first, then sort
SELECT order_id, order_date, total_amount 
FROM orders 
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
ORDER BY order_date DESC 
LIMIT 10;

-- ❌ BAD: Function on indexed column
SELECT * FROM customers WHERE YEAR(registration_date) = 2025;

-- ✅ GOOD: Range query on indexed column
SELECT * FROM customers 
WHERE registration_date >= '2025-01-01' 
AND registration_date < '2026-01-01';
```

### 2. Use Proper Indexes
```sql
-- Analyze query performance
EXPLAIN SELECT * FROM orders WHERE customer_id = 101;

-- Create appropriate indexes
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_products_category ON products(category);

-- Composite index for multi-column queries
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
```

### 3. Avoid N+1 Queries
```sql
-- ❌ BAD: N+1 query problem (requires multiple queries)
-- Query 1: Get all orders
SELECT * FROM orders;
-- Query 2-N: For each order, get customer (N queries)
SELECT * FROM customers WHERE customer_id = ?;

-- ✅ GOOD: Single query with JOIN
SELECT 
    o.order_id,
    o.order_date,
    o.total_amount,
    c.customer_name,
    c.email
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

### 4. Use Transactions for Data Integrity
```sql
-- Always use transactions for multi-step operations
START TRANSACTION;

-- Step 1: Deduct from account A
UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;

-- Step 2: Add to account B
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;

-- Step 3: Log transaction
INSERT INTO transaction_log (from_account, to_account, amount, timestamp)
VALUES (1, 2, 1000, NOW());

-- Commit if all steps succeed
COMMIT;

-- Rollback on error
-- ROLLBACK;
```

### 5. Parameterized Queries (Prevent SQL Injection)
```sql
-- ❌ BAD: String concatenation (SQL injection risk)
-- query = "SELECT * FROM users WHERE email = '" + userInput + "'";

-- ✅ GOOD: Parameterized query (safe)
-- Using prepared statements in your application
PREPARE stmt FROM 'SELECT * FROM users WHERE email = ?';
SET @email = 'user@example.com';
EXECUTE stmt USING @email;
DEALLOCATE PREPARE stmt;
```

### 6. Naming Conventions
```sql
-- Use clear, descriptive names
-- Tables: plural nouns (customers, orders, products)
-- Columns: singular nouns (customer_id, order_date, product_name)
-- Indexes: idx_tablename_columnname
-- Foreign keys: fk_tablename_columnname
-- Primary keys: Always use 'id' or 'table_name_id'

CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 7. Data Types Best Practices
```sql
-- Use appropriate data types
CREATE TABLE products (
    product_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,  -- UNSIGNED for IDs
    product_name VARCHAR(200) NOT NULL,                  -- VARCHAR for variable text
    description TEXT,                                     -- TEXT for long content
    price DECIMAL(10, 2) NOT NULL,                       -- DECIMAL for money
    stock_quantity INT UNSIGNED DEFAULT 0,               -- INT for counts
    is_active BOOLEAN DEFAULT TRUE,                      -- BOOLEAN for flags
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,      -- TIMESTAMP for dates
    category ENUM('Electronics', 'Clothing', 'Food')     -- ENUM for fixed options
);
```

---

## Summary: Essential SQL Commands Checklist

### Basic (Must Know)
- ✅ SELECT - Retrieve data
- ✅ WHERE - Filter data
- ✅ ORDER BY - Sort results
- ✅ INSERT - Add new records
- ✅ UPDATE - Modify records
- ✅ DELETE - Remove records
- ✅ LIMIT - Restrict results

### Intermediate (Should Know)
- ✅ JOINS (INNER, LEFT, RIGHT, FULL)
- ✅ GROUP BY - Aggregate data
- ✅ HAVING - Filter grouped data
- ✅ Aggregate functions (COUNT, SUM, AVG, MIN, MAX)
- ✅ LIKE - Pattern matching
- ✅ IN - Multiple values
- ✅ BETWEEN - Range queries
- ✅ CASE - Conditional logic

### Advanced (Important)
- ✅ Subqueries
- ✅ CTEs (WITH clause)
- ✅ Window functions (ROW_NUMBER, RANK, LAG, LEAD)
- ✅ UNION / UNION ALL
- ✅ EXISTS / NOT EXISTS

### Expert (For Complex Systems)
- ✅ Recursive CTEs
- ✅ Pivot tables
- ✅ Indexes and optimization
- ✅ Transactions (START, COMMIT, ROLLBACK)
- ✅ Stored procedures
- ✅ Triggers
- ✅ Views

---

**Practice Tip**: Start with basic queries on a sample database, then gradually move to complex scenarios. Focus on understanding WHEN to use each command, not just HOW.

**Next Steps**:
1. Set up a practice database (MySQL, PostgreSQL, or SQLite)
2. Create sample tables (customers, orders, products)
3. Practice each query type with real data
4. Analyze query performance with EXPLAIN
5. Build a complete project (e-commerce, blog, etc.)

Good luck with your SQL journey! 🚀
