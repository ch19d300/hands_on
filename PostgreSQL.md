# PostgreSQL Hands-On Workshop

## Introduction to RDBMS with PostgreSQL

This workshop will guide you through practical exercises using PostgreSQL, a powerful open-source relational database management system.

## Workshop Objectives
- Understand relational database design concepts
- Practice SQL query writing and optimization
- Implement database constraints and relationships
- Work with transactions and database functions

## Connecting to PostgreSQL

### Using psql (Command Line)
```bash
# From Docker
docker exec -it workshop-postgres psql -U workshop -d workshop

# Direct connection (if PostgreSQL client is installed)
psql -h localhost -p 5432 -U workshop -d workshop
```

### Using pgAdmin (GUI)
1. Download and install pgAdmin from [https://www.pgadmin.org/download/](https://www.pgadmin.org/download/)
2. Create a new server connection:
   - Name: Workshop
   - Host: localhost
   - Port: 5432
   - Username: workshop
   - Password: workshop
   - Database: workshop

## Part 1: Database Design and Creation

### Database Normalization
Before we start creating tables, let's review normalization principles:

- **First Normal Form (1NF)**: No repeating groups, atomic values
- **Second Normal Form (2NF)**: 1NF + No partial dependencies
- **Third Normal Form (3NF)**: 2NF + No transitive dependencies

### Creating Database Schema

```sql
-- Create tables for an e-commerce database
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE product_categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    inventory_count INTEGER NOT NULL DEFAULT 0,
    category_id INTEGER REFERENCES product_categories(category_id)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(customer_id),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    total_amount DECIMAL(10, 2)
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL
);
```

### Exercise 1: Schema Modification
Add a shipping_address table that links to customers and orders:

```sql
-- Your solution here
```

## Part 2: Data Manipulation

### Basic INSERT Operations

```sql
-- Insert sample data
INSERT INTO product_categories (name, description) VALUES 
('Electronics', 'Electronic devices and accessories'),
('Clothing', 'Apparel and fashion items'),
('Books', 'Books, e-books and publications');

INSERT INTO products (name, description, price, inventory_count, category_id) VALUES 
('Smartphone XL', '6.5 inch screen, 128GB storage', 699.99, 50, 1),
('Laptop Pro', '15 inch, 16GB RAM, 512GB SSD', 1299.99, 20, 1),
('Cotton T-Shirt', 'Comfortable cotton t-shirt, available in multiple colors', 19.99, 100, 2),
('SQL Database Guide', 'Comprehensive guide to SQL databases', 39.99, 30, 3);

INSERT INTO customers (first_name, last_name, email, phone) VALUES 
('John', 'Doe', 'john.doe@example.com', '555-123-4567'),
('Jane', 'Smith', 'jane.smith@example.com', '555-987-6543');
```

### SELECT Queries

```sql
-- Basic queries
SELECT * FROM products;

-- Filtering
SELECT name, price FROM products WHERE category_id = 1;

-- Joining tables
SELECT p.name, p.price, c.name as category
FROM products p
JOIN product_categories c ON p.category_id = c.category_id;

-- Aggregation
SELECT c.name, COUNT(p.product_id) as product_count, AVG(p.price) as avg_price
FROM products p
JOIN product_categories c ON p.category_id = c.category_id
GROUP BY c.name;
```

### Exercise 2: Complex Queries
Write a query to find all products with inventory less than 25, showing their category and inventory status:

```sql
-- Your solution here
```

## Part 3: Transactions and Integrity

### Working with Transactions

```sql
-- Create a new order with multiple items
BEGIN;

-- Insert order
INSERT INTO orders (customer_id, status, total_amount)
VALUES (1, 'pending', 0)
RETURNING order_id;

-- Let's assume the returned order_id is 1
-- Insert order items
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES 
(1, 1, 2, 699.99),  -- 2 Smartphones
(1, 3, 3, 19.99);   -- 3 T-shirts

-- Update order total
UPDATE orders 
SET total_amount = (
    SELECT SUM(quantity * unit_price) 
    FROM order_items 
    WHERE order_id = 1
)
WHERE order_id = 1;

-- Update inventory
UPDATE products SET inventory_count = inventory_count - 2 WHERE product_id = 1;
UPDATE products SET inventory_count = inventory_count - 3 WHERE product_id = 3;

COMMIT;
```

### Exercise 3: Create a Transaction
Write a transaction to:
1. Create a new customer
2. Create a new order for that customer
3. Add two different products to the order
4. Ensure proper inventory management

```sql
-- Your solution here
```

## Part 4: Advanced PostgreSQL Features

### Creating Views

```sql
-- Create a view for order details
CREATE VIEW order_details AS
SELECT 
    o.order_id,
    c.first_name || ' ' || c.last_name AS customer_name,
    o.order_date,
    o.status,
    o.total_amount,
    COUNT(oi.order_item_id) AS total_items
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, c.first_name, c.last_name, o.order_date, o.status, o.total_amount;

-- Query the view
SELECT * FROM order_details;
```

### Stored Procedures and Functions

```sql
-- Create a function to calculate order total
CREATE OR REPLACE FUNCTION calculate_order_total(order_id_param INTEGER)
RETURNS DECIMAL AS $$
DECLARE
    total DECIMAL(10,2);
BEGIN
    SELECT SUM(quantity * unit_price) INTO total
    FROM order_items
    WHERE order_id = order_id_param;
    
    RETURN total;
END;
$$ LANGUAGE plpgsql;

-- Use the function
SELECT calculate_order_total(1);
```

### Exercise 4: Create a Function
Create a function that returns the top N products by sales volume:

```sql
-- Your solution here
```

## Part 5: Performance and Indexing

### Creating Indexes

```sql
-- Create an index for product searches
CREATE INDEX idx_products_name ON products(name);

-- Create composite index for order lookup
CREATE INDEX idx_order_items_composite ON order_items(order_id, product_id);
```

### Query Performance Analysis

```sql
-- Use EXPLAIN to analyze query performance
EXPLAIN ANALYZE
SELECT p.name, p.price, c.name as category
FROM products p
JOIN product_categories c ON p.category_id = c.category_id
WHERE p.price > 50;
```

### Exercise 5: Optimize a Query
Create appropriate indexes and optimize this query:

```sql
SELECT c.first_name, c.last_name, COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date > '2023-01-01'
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING COUNT(o.order_id) > 2;
```

## Sample Dataset

For more extensive practice, we'll use the DVD Rental sample database:

1. Download the database file: [dvdrental.tar](https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip)
2. Extract the tar file
3. Import using:
   ```bash
   # Copy file to container
   docker cp dvdrental.tar workshop-postgres:/tmp/
   
   # Restore the database
   docker exec -it workshop-postgres pg_restore -U workshop -d workshop /tmp/dvdrental.tar
   ```

This dataset includes 15 tables with realistic sample data for a DVD rental business.

## Additional Exercises

1. Find the top 5 customers who have spent the most money renting films
2. Identify films that have never been rented
3. Calculate the average rental duration by film category
4. Create a stored procedure to generate a customer rental history report
5. Design and implement a late fee calculation system

## Resources
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [SQL Style Guide](https://www.sqlstyle.guide/)