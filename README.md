# SQL-project
Online Store Inventory &amp; Sales  Tables: Products, Categories, Customers, Orders, Order_Details.  Show: JOINs to list customer orders, GROUP BY for sales reports, indexes for product search.
-- Online Store Inventory & Sales Database
-- Complete database schema with sample data and queries

-- =============================================
-- 1. DATABASE SETUP
-- =============================================

-- Create database (uncomment if needed)
-- CREATE DATABASE OnlineStore;
-- USE OnlineStore;

-- =============================================
-- 2. TABLE CREATION
-- =============================================

-- Categories table
CREATE TABLE Categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_category_name (category_name)
);

-- Products table with inventory tracking
CREATE TABLE Products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(255) NOT NULL,
    category_id INT,
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    stock_quantity INT NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    description TEXT,
    sku VARCHAR(50) UNIQUE,
    weight DECIMAL(8,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    
    FOREIGN KEY (category_id) REFERENCES Categories(category_id) ON DELETE SET NULL,
    INDEX idx_product_name (product_name),
    INDEX idx_category_id (category_id),
    INDEX idx_sku (sku),
    INDEX idx_price (price),
    INDEX idx_stock (stock_quantity)
);

-- Customers table
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20),
    address TEXT,
    city VARCHAR(50),
    state VARCHAR(50),
    zip_code VARCHAR(10),
    country VARCHAR(50) DEFAULT 'India',
    registration_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    
    INDEX idx_customer_email (email),
    INDEX idx_customer_name (last_name, first_name),
    INDEX idx_customer_city (city)
);

-- Orders table
CREATE TABLE Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    total_amount DECIMAL(12,2) NOT NULL CHECK (total_amount >= 0),
    shipping_address TEXT,
    shipping_cost DECIMAL(8,2) DEFAULT 0,
    tax_amount DECIMAL(10,2) DEFAULT 0,
    notes TEXT,
    
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id) ON DELETE CASCADE,
    INDEX idx_customer_id (customer_id),
    INDEX idx_order_date (order_date),
    INDEX idx_status (status),
    INDEX idx_total_amount (total_amount)
);

-- Order Details table (junction table for orders and products)
CREATE TABLE Order_Details (
    order_detail_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL CHECK (unit_price >= 0),
    discount_amount DECIMAL(8,2) DEFAULT 0 CHECK (discount_amount >= 0),
    line_total DECIMAL(12,2) GENERATED ALWAYS AS ((quantity * unit_price) - discount_amount) STORED,
    
    FOREIGN KEY (order_id) REFERENCES Orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES Products(product_id) ON DELETE CASCADE,
    INDEX idx_order_id (order_id),
    INDEX idx_product_id (product_id),
    UNIQUE KEY unique_order_product (order_id, product_id)
);

-- =============================================
-- 3. SAMPLE DATA INSERTION
-- =============================================

-- Insert Categories
INSERT INTO Categories (category_name, description) VALUES
('Electronics', 'Electronic devices and gadgets'),
('Clothing', 'Apparel and fashion items'),
('Books', 'Books and educational materials'),
('Home & Garden', 'Home improvement and garden supplies'),
('Sports', 'Sports equipment and accessories'),
('Toys', 'Toys and games for all ages');

-- Insert Products
INSERT INTO Products (product_name, category_id, price, stock_quantity, description, sku, weight) VALUES
('Laptop Computer', 1, 899.99, 25, 'High-performance laptop with 16GB RAM', 'LAP001', 2.5),
('Smartphone', 1, 699.99, 50, 'Latest model smartphone with 128GB storage', 'PHN001', 0.4),
('Wireless Headphones', 1, 149.99, 100, 'Noise-cancelling wireless headphones', 'HDP001', 0.3),
('Men\'s T-Shirt', 2, 24.99, 200, '100% cotton comfortable t-shirt', 'TSH001', 0.2),
('Women\'s Jeans', 2, 79.99, 75, 'Premium denim jeans in various sizes', 'JNS001', 0.8),
('Python Programming Book', 3, 39.99, 30, 'Learn Python programming from basics', 'BK001', 0.6),
('Garden Tools Set', 4, 89.99, 15, 'Complete set of essential garden tools', 'GDN001', 3.2),
('Basketball', 5, 29.99, 40, 'Official size basketball', 'BBL001', 0.6),
('Board Game', 6, 34.99, 60, 'Family-friendly strategy board game', 'TOY001', 1.1);

-- Insert Customers
INSERT INTO Customers (first_name, last_name, email, phone, address, city, state, zip_code) VALUES
('Arjun', 'Sharma', 'arjun.sharma@email.com', '555-0101', '15 MG Road', 'Mumbai', 'MH', '400001'),
('Priya', 'Patel', 'priya.patel@email.com', '555-0102', '22 Brigade Road', 'Bangalore', 'KA', '560025'),
('Rajesh', 'Kumar', 'rajesh.kumar@email.com', '555-0103', '45 Park Street', 'Kolkata', 'WB', '700016'),
('Meera', 'Singh', 'meera.singh@email.com', '555-0104', '8 Connaught Place', 'New Delhi', 'DL', '110001'),
('Vikram', 'Gupta', 'vikram.gupta@email.com', '555-0105', '33 Anna Salai', 'Chennai', 'TN', '600002'),
('Kavya', 'Reddy', 'kavya.reddy@email.com', '555-0106', '12 Banjara Hills', 'Hyderabad', 'TS', '500034');

-- Insert Orders
INSERT INTO Orders (customer_id, status, total_amount, shipping_cost, tax_amount, shipping_address) VALUES
(1, 'delivered', 974.97, 15.00, 59.98, '15 MG Road, Mumbai, MH 400001'),
(2, 'shipped', 254.97, 10.00, 19.98, '22 Brigade Road, Bangalore, KA 560025'),
(3, 'processing', 119.98, 8.00, 7.98, '45 Park Street, Kolkata, WB 700016'),
(1, 'pending', 169.98, 12.00, 11.98, '15 MG Road, Mumbai, MH 400001'),
(4, 'delivered', 89.99, 5.00, 4.99, '8 Connaught Place, New Delhi, DL 110001'),
(5, 'cancelled', 0.00, 0.00, 0.00, '33 Anna Salai, Chennai, TN 600002');

-- Insert Order Details
INSERT INTO Order_Details (order_id, product_id, quantity, unit_price, discount_amount) VALUES
-- Order 1: Arjun Sharma
(1, 1, 1, 899.99, 0.00),  -- Laptop
(1, 3, 1, 149.99, 25.00), -- Headphones with discount

-- Order 2: Priya Patel
(2, 2, 1, 699.99, 50.00), -- Smartphone with discount
(2, 4, 2, 24.99, 0.00),   -- T-shirts

-- Order 3: Rajesh Kumar
(3, 3, 1, 149.99, 30.00), -- Headphones with discount
(3, 6, 1, 39.99, 0.00),   -- Book

-- Order 4: Arjun Sharma (second order)
(4, 5, 1, 79.99, 0.00),   -- Jeans
(4, 8, 1, 29.99, 0.00),   -- Basketball
(4, 9, 2, 34.99, 0.00),   -- Board games

-- Order 5: Meera Singh
(5, 7, 1, 89.99, 0.00);   -- Garden tools

-- =============================================
-- 4. ADVANCED QUERIES & REPORTS
-- =============================================

-- Query 1: Customer Orders with JOINs
-- List all customer orders with customer details and order information
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.email,
    o.order_id,
    o.order_date,
    o.status,
    o.total_amount,
    COUNT(od.product_id) as items_count
FROM Customers c
JOIN Orders o ON c.customer_id = o.customer_id
JOIN Order_Details od ON o.order_id = od.order_id
GROUP BY c.customer_id, o.order_id
ORDER BY o.order_date DESC;

-- Query 2: Detailed Order Information
-- Show complete order details including products
SELECT 
    o.order_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    o.order_date,
    o.status,
    p.product_name,
    cat.category_name,
    od.quantity,
    od.unit_price,
    od.discount_amount,
    od.line_total
FROM Orders o
JOIN Customers c ON o.customer_id = c.customer_id
JOIN Order_Details od ON o.order_id = od.order_id
JOIN Products p ON od.product_id = p.product_id
JOIN Categories cat ON p.category_id = cat.category_id
ORDER BY o.order_id, od.order_detail_id;

-- Query 3: Sales Reports with GROUP BY
-- Monthly sales report
SELECT 
    YEAR(o.order_date) AS sales_year,
    MONTH(o.order_date) AS sales_month,
    MONTHNAME(o.order_date) AS month_name,
    COUNT(DISTINCT o.order_id) AS total_orders,
    COUNT(DISTINCT o.customer_id) AS unique_customers,
    SUM(o.total_amount) AS total_revenue,
    AVG(o.total_amount) AS avg_order_value
FROM Orders o
WHERE o.status != 'cancelled'
GROUP BY YEAR(o.order_date), MONTH(o.order_date)
ORDER BY sales_year DESC, sales_month DESC;

-- Query 4: Product Sales Analysis
-- Top-selling products by quantity and revenue
SELECT 
    p.product_id,
    p.product_name,
    cat.category_name,
    SUM(od.quantity) AS total_sold,
    SUM(od.line_total) AS total_revenue,
    AVG(od.unit_price) AS avg_selling_price,
    p.stock_quantity AS current_stock
FROM Products p
JOIN Categories cat ON p.category_id = cat.category_id
JOIN Order_Details od ON p.product_id = od.product_id
JOIN Orders o ON od.order_id = o.order_id
WHERE o.status != 'cancelled'
GROUP BY p.product_id, p.product_name, cat.category_name, p.stock_quantity
ORDER BY total_revenue DESC;

-- Query 5: Category Performance
-- Sales performance by category
SELECT 
    cat.category_name,
    COUNT(DISTINCT p.product_id) AS products_count,
    COUNT(DISTINCT od.order_id) AS orders_count,
    SUM(od.quantity) AS total_items_sold,
    SUM(od.line_total) AS total_revenue,
    AVG(od.line_total) AS avg_line_value
FROM Categories cat
LEFT JOIN Products p ON cat.category_id = p.category_id
LEFT JOIN Order_Details od ON p.product_id = od.product_id
LEFT JOIN Orders o ON od.order_id = o.order_id AND o.status != 'cancelled'
GROUP BY cat.category_id, cat.category_name
ORDER BY total_revenue DESC;

-- Query 6: Customer Analysis
-- Customer purchase behavior and value
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.email,
    COUNT(DISTINCT o.order_id) AS total_orders,
    SUM(CASE WHEN o.status != 'cancelled' THEN o.total_amount ELSE 0 END) AS total_spent,
    AVG(CASE WHEN o.status != 'cancelled' THEN o.total_amount END) AS avg_order_value,
    MAX(o.order_date) AS last_order_date,
    DATEDIFF(CURDATE(), MAX(o.order_date)) AS days_since_last_order
FROM Customers c
LEFT JOIN Orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.email
HAVING total_orders > 0
ORDER BY total_spent DESC;

-- Query 7: Inventory Management
-- Low stock alert and inventory valuation
SELECT 
    p.product_id,
    p.product_name,
    p.sku,
    cat.category_name,
    p.stock_quantity,
    p.price,
    (p.stock_quantity * p.price) AS inventory_value,
    CASE 
        WHEN p.stock_quantity = 0 THEN 'OUT OF STOCK'
        WHEN p.stock_quantity <= 10 THEN 'LOW STOCK'
        WHEN p.stock_quantity <= 25 THEN 'MEDIUM STOCK'
        ELSE 'HIGH STOCK'
    END AS stock_status
FROM Products p
JOIN Categories cat ON p.category_id = cat.category_id
WHERE p.is_active = TRUE
ORDER BY p.stock_quantity ASC, inventory_value DESC;

-- Query 8: Order Status Summary
-- Summary of orders by status
SELECT 
    status,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_value,
    AVG(total_amount) AS avg_order_value,
    MIN(order_date) AS earliest_order,
    MAX(order_date) AS latest_order
FROM Orders
GROUP BY status
ORDER BY 
    CASE status 
        WHEN 'pending' THEN 1 
        WHEN 'processing' THEN 2 
        WHEN 'shipped' THEN 3 
        WHEN 'delivered' THEN 4 
        WHEN 'cancelled' THEN 5 
    END;

-- =============================================
-- 5. USEFUL VIEWS FOR REPORTING
-- =============================================

-- Create a view for easy order reporting
CREATE VIEW OrderSummary AS
SELECT 
    o.order_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.email,
    o.order_date,
    o.status,
    o.total_amount,
    COUNT(od.product_id) as item_count,
    SUM(od.quantity) as total_quantity
FROM Orders o
JOIN Customers c ON o.customer_id = c.customer_id
JOIN Order_Details od ON o.order_id = od.order_id
GROUP BY o.order_id, customer_name, c.email, o.order_date, o.status, o.total_amount;

-- Create a view for product sales summary
CREATE VIEW ProductSales AS
SELECT 
    p.product_id,
    p.product_name,
    p.sku,
    cat.category_name,
    p.price,
    p.stock_quantity,
    COALESCE(SUM(od.quantity), 0) as total_sold,
    COALESCE(SUM(od.line_total), 0) as total_revenue,
    (p.stock_quantity * p.price) as inventory_value
FROM Products p
JOIN Categories cat ON p.category_id = cat.category_id
LEFT JOIN Order_Details od ON p.product_id = od.product_id
LEFT JOIN Orders o ON od.order_id = o.order_id AND o.status != 'cancelled'
GROUP BY p.product_id, p.product_name, p.sku, cat.category_name, p.price, p.stock_quantity;

-- =============================================
-- 6. PERFORMANCE INDEXES
-- =============================================

-- Additional indexes for better query performance
CREATE INDEX idx_orders_customer_date ON Orders(customer_id, order_date);
CREATE INDEX idx_order_details_composite ON Order_Details(order_id, product_id, quantity);
CREATE INDEX idx_products_category_price ON Products(category_id, price);
CREATE INDEX idx_customers_name_email ON Customers(last_name, first_name, email);

-- =============================================
-- 7. SAMPLE USAGE OF VIEWS
-- =============================================

-- Use the OrderSummary view
SELECT * FROM OrderSummary WHERE status = 'delivered' ORDER BY order_date DESC;

-- Use the ProductSales view
SELECT * FROM ProductSales WHERE total_sold > 0 ORDER BY total_revenue DESC LIMIT 5;
