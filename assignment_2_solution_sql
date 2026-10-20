-- =====================================================================
-- ASSIGNMENT-2: RETAIL BUSINESS DATABASE MANAGEMENT SYSTEM
-- =====================================================================

-- 0. CLEANUP (DROPPING EXISTING TABLES SAFELY)
DROP TABLE IF EXISTS returns CASCADE;
DROP TABLE IF EXISTS order_items CASCADE;
DROP TABLE IF EXISTS orders CASCADE;
DROP TABLE IF EXISTS employees CASCADE;
DROP TABLE IF EXISTS products CASCADE;
DROP TABLE IF EXISTS customers CASCADE;


-- 1. DATABASE SCHEMA CREATION (TABLES)

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    city VARCHAR(100),
    state VARCHAR(100),
    signup_date DATE
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(100),
    current_price DECIMAL(10,2),
    stock_quantity INT
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100) NOT NULL,
    branch_city VARCHAR(100),
    monthly_salary DECIMAL(10,2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE,
    customer_id INT,
    assigned_employee_id INT,
    order_status VARCHAR(50),
    payment_method VARCHAR(50),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (assigned_employee_id) REFERENCES employees(employee_id)
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price_at_sale DECIMAL(10,2),
    discount_rate DECIMAL(4,2),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE returns (
    return_id INT PRIMARY KEY,
    order_id INT,
    return_date DATE,
    return_reason VARCHAR(255),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);


-- =====================================================================
-- 2. INSERTING SAMPLE DATA
-- =====================================================================

INSERT INTO customers (customer_id, customer_name, email, city, state, signup_date) VALUES 
(1001, 'Aarav Sharma', 'aarav.sharma@example.com', 'Lucknow', 'Uttar Pradesh', '2024-01-15'),
(1002, 'Diya Verma', 'diya.verma@example.com', 'Kanpur', 'Uttar Pradesh', '2024-02-10'),
(1003, 'Kabir Singh', 'kabir.singh@example.com', 'Delhi', 'Delhi', '2023-11-22');

INSERT INTO products (product_id, product_name, category, current_price, stock_quantity) VALUES 
(101, 'Laptop', 'Electronics', 45000.00, 15),
(102, 'Smartphone', 'Electronics', 20000.00, 35),
(103, 'Office Chair', 'Furniture', 5000.00, 50);

INSERT INTO employees (employee_id, employee_name, branch_city, monthly_salary) VALUES 
(1, 'Suresh Rao', 'Delhi', 40000.00),
(2, 'Anjali Mehta', 'Mumbai', 45000.00);

INSERT INTO orders (order_id, order_date, customer_id, assigned_employee_id, order_status, payment_method) VALUES 
(2001, '2024-11-05', 1001, 1, 'Delivered', 'Credit Card'),
(2002, '2024-12-10', 1002, 2, 'Delivered', 'UPI'),
(2003, '2025-01-15', 1001, 1, 'Delivered', 'Cash');

INSERT INTO order_items (order_item_id, order_id, product_id, quantity, unit_price_at_sale, discount_rate) VALUES 
(1, 2001, 101, 1, 45000.00, 0.10),
(2, 2002, 102, 2, 20000.00, 0.05),
(3, 2003, 103, 3, 5000.00, 0.00);

INSERT INTO returns (return_id, order_id, return_date, return_reason) VALUES 
(1, 2003, '2025-01-18', 'Defective product');


-- =====================================================================
-- 3. QUERIES TO PERFORM (Q1 TO Q15)
-- =====================================================================

-- Q1. Basic filtering and sorting: List delivered orders placed between 1 October 2024 and 31 March 2025, showing order_id, order_date, customer_id, and payment_method. Sort by order_date descending.
SELECT order_id, order_date, customer_id, payment_method
FROM orders
WHERE order_status = 'Delivered' 
  AND order_date BETWEEN '2024-10-01' AND '2025-03-31'
ORDER BY order_date DESC;


-- Q2. Multi-table JOIN: Display each delivered order with customer_name, order_id, order_date, employee_name, and order_status. Include only orders with matching customer and employee records.
SELECT 
    customers.customer_name, 
    orders.order_id, 
    orders.order_date, 
    employees.employee_name, 
    orders.order_status
FROM orders
JOIN customers ON orders.customer_id = customers.customer_id
JOIN employees ON orders.assigned_employee_id = employees.employee_id
WHERE orders.order_status = 'Delivered';


-- Q3. Aggregation and HAVING: For each product category, calculate total units sold and net revenue from delivered orders. Show only categories with net revenue greater than ₹10,000; sort by net revenue descending.
SELECT 
    products.category,
    SUM(order_items.quantity) AS total_units_sold,
    SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)) AS net_revenue
FROM order_items
JOIN orders ON order_items.order_id = orders.order_id
JOIN products ON order_items.product_id = products.product_id
WHERE orders.order_status = 'Delivered'
GROUP BY products.category
HAVING SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)) > 10000
ORDER BY net_revenue DESC;


-- Q4. Customers with repeat purchases: Find customers who have placed at least two delivered orders. Show customer_id, customer_name, number of delivered orders, and total net spend.
SELECT 
    customers.customer_id,
    customers.customer_name,
    COUNT(orders.order_id) AS number_of_delivered_orders,
    SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)) AS total_net_spend
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
JOIN order_items ON orders.order_id = order_items.order_id
WHERE orders.order_status = 'Delivered'
GROUP BY customers.customer_id, customers.customer_name
HAVING COUNT(orders.order_id) >= 2;


-- Q5. LEFT JOIN and zero activity: List all customers and the number of delivered orders each has placed, including customers with zero delivered orders.
SELECT 
    customers.customer_id,
    customers.customer_name,
    COUNT(orders.order_id) AS delivered_orders
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id AND orders.order_status = 'Delivered'
GROUP BY customers.customer_id, customers.customer_name;


-- Q6. Subquery: Find products whose unit_price is greater than the average unit_price of all products. Display product name, category, unit price, and the overall average price.
SELECT 
    product_name,
    category,
    current_price AS unit_price,
    (SELECT AVG(current_price) FROM products) AS overall_average_price
FROM products
WHERE current_price > (SELECT AVG(current_price) FROM products);


-- Q7. NOT EXISTS: Find customers who have never placed a delivered order. Return their customer_id, customer_name, and city.
SELECT 
    customer_id,
    customer_name,
    city
FROM customers
WHERE NOT EXISTS (
    SELECT 1 
    FROM orders 
    WHERE orders.customer_id = customers.customer_id 
      AND orders.order_status = 'Delivered'
);


-- Q8. Date functions: Group delivered orders by year and month, showing the number of orders and net revenue for each month. Sort chronologically.
SELECT 
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(DISTINCT orders.order_id) AS total_orders,
    SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)) AS net_revenue
FROM orders
JOIN order_items ON orders.order_id = order_items.order_id
WHERE orders.order_status = 'Delivered'
GROUP BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)
ORDER BY year ASC, month ASC;


-- Q9. CASE expression: Classify each product as 'Low Stock' when stock_quantity < 20, 'Moderate Stock' when stock_quantity is between 20 and 40 inclusive, and 'High Stock' when stock_quantity > 40.
SELECT 
    product_name,
    stock_quantity,
    CASE 
        WHEN stock_quantity < 20 THEN 'Low Stock'
        WHEN stock_quantity BETWEEN 20 AND 40 THEN 'Moderate Stock'
        WHEN stock_quantity > 40 THEN 'High Stock'
    END AS stock_label
FROM products;


-- Q10. Conditional aggregation: For each employee, calculate delivered, returned, and cancelled order counts using conditional aggregation. Include employees with no orders.
SELECT 
    employees.employee_id,
    employees.employee_name,
    SUM(CASE WHEN orders.order_status = 'Delivered' THEN 1 ELSE 0 END) AS delivered_orders,
    SUM(CASE WHEN orders.order_status = 'Returned' THEN 1 ELSE 0 END) AS returned_orders,
    SUM(CASE WHEN orders.order_status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled_orders
FROM employees
LEFT JOIN orders ON employees.employee_id = orders.assigned_employee_id
GROUP BY employees.employee_id, employees.employee_name;


-- Q11. Window function – ranking: Rank products within each category by net revenue from delivered orders, highest first. Show category, product_name, net revenue, and rank.
SELECT 
    products.category,
    products.product_name,
    COALESCE(SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)), 0) AS net_revenue,
    DENSE_RANK() OVER (
        PARTITION BY products.category 
        ORDER BY COALESCE(SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)), 0) DESC
    ) AS rank
FROM products
LEFT JOIN order_items ON products.product_id = order_items.product_id
LEFT JOIN orders ON order_items.order_id = orders.order_id AND orders.order_status = 'Delivered'
GROUP BY products.category, products.product_id, products.product_name;


-- Q12. Running total: For each month, calculate delivered-order net revenue and a cumulative running total of revenue across months in chronological order.
WITH MonthlyData AS (
    SELECT 
        EXTRACT(YEAR FROM orders.order_date) AS year,
        EXTRACT(MONTH FROM orders.order_date) AS month,
        SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)) AS monthly_net_revenue
    FROM orders
    JOIN order_items ON orders.order_id = order_items.order_id
    WHERE orders.order_status = 'Delivered'
    GROUP BY EXTRACT(YEAR FROM orders.order_date), EXTRACT(MONTH FROM orders.order_date)
)
SELECT 
    year,
    month,
    monthly_net_revenue,
    SUM(monthly_net_revenue) OVER (ORDER BY year, month) AS running_total
FROM MonthlyData
ORDER BY year, month;


-- Q13. Return analysis: List each returned order with customer name, return date, return reason, and the total pre-discount value of its order items.
SELECT 
    returns.order_id,
    customers.customer_name,
    returns.return_date,
    returns.return_reason,
    SUM(order_items.quantity * order_items.unit_price_at_sale) AS total_pre_discount_value
FROM returns
JOIN orders ON returns.order_id = orders.order_id
JOIN customers ON orders.customer_id = customers.customer_id
JOIN order_items ON orders.order_id = order_items.order_id
GROUP BY returns.order_id, customers.customer_name, returns.return_date, returns.return_reason;


-- Q14. CTE: Using a CTE, calculate net revenue per customer from delivered orders, then return customers whose net revenue is above the average customer net revenue.
WITH CustomerSpend AS (
    SELECT 
        orders.customer_id,
        SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)) AS net_spend
    FROM orders
    JOIN order_items ON orders.order_id = order_items.order_id
    WHERE orders.order_status = 'Delivered'
    GROUP BY orders.customer_id
)
SELECT 
    customers.customer_id,
    customers.customer_name,
    CustomerSpend.net_spend
FROM CustomerSpend
JOIN customers ON CustomerSpend.customer_id = customers.customer_id
WHERE CustomerSpend.net_spend > (SELECT AVG(net_spend) FROM CustomerSpend);


-- Q15. Business insight query: Identify the top three customers by delivered net spend. Show customer name, number of delivered orders, net spend, and each customer's percentage contribution to total delivered net revenue.
WITH TopCustomers AS (
    SELECT 
        customers.customer_name,
        COUNT(DISTINCT orders.order_id) AS delivered_orders,
        SUM(order_items.quantity * order_items.unit_price_at_sale * (1 - order_items.discount_rate)) AS net_spend
    FROM customers
    JOIN orders ON customers.customer_id = orders.customer_id
    JOIN order_items ON orders.order_id = order_items.order_id
    WHERE orders.order_status = 'Delivered'
    GROUP BY customers.customer_id, customers.customer_name
)
SELECT 
    customer_name,
    delivered_orders,
    net_spend,
    ROUND(CAST((net_spend / (SELECT SUM(net_spend) FROM TopCustomers)) * 100 AS NUMERIC), 2) AS percentage_contribution
FROM TopCustomers
ORDER BY net_spend DESC
LIMIT 3;
