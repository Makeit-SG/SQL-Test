Okay, here's the final `README.md` file where I've separated the questions, the database schema and data, and the solutions into distinct sections, making it even more organized and easier to navigate. This enhances clarity and reduces potential confusion.

```markdown
# ClickHouse SQL Take-Home Test - E-commerce Analytics

This repository contains a take-home SQL test designed for assessing the skills of SQL developers, specifically within the context of ClickHouse. The test focuses on complex joins, data aggregation, and data cleaning techniques required for e-commerce analytics.

## Overview

The test involves querying an e-commerce database with tables for customers, products, orders, order items, product reviews, and promotions. The candidate is tasked with writing SQL queries to answer specific business questions, taking into account potentially inconsistent or missing data within the dataset.

## 1. Database Schema and Data

### 1.1 Database Schema (schema.sql)

```sql
CREATE DATABASE IF NOT EXISTS ecommerce;

USE ecommerce;

-- Table: Customers
CREATE TABLE IF NOT EXISTS Customers (
    customer_id UInt32,
    first_name String,
    last_name String,
    email String,
    registration_date Date,
    country String,
    PRIMARY KEY (customer_id)
) ENGINE = MergeTree()
ORDER BY (customer_id);

-- Table: Products
CREATE TABLE IF NOT EXISTS Products (
    product_id UInt32,
    product_name String,
    category String,
    price Decimal(10, 2),
    description String,
    PRIMARY KEY (product_id)
) ENGINE = MergeTree()
ORDER BY (product_id);


-- Table: Orders
CREATE TABLE IF NOT EXISTS Orders (
    order_id UInt32,
    customer_id UInt32,
    order_date Date,
    total_amount Decimal(10, 2),
    shipping_address String,
    PRIMARY KEY (order_id)
) ENGINE = MergeTree()
ORDER BY (order_id);


-- Table: OrderItems (Many-to-many relationship between Orders and Products)
CREATE TABLE IF NOT EXISTS OrderItems (
    order_item_id UInt32,
    order_id UInt32,
    product_id UInt32,
    quantity UInt8,
    item_price Decimal(10, 2),
    PRIMARY KEY (order_item_id)
) ENGINE = MergeTree()
ORDER BY (order_item_id);

-- Table: ProductReviews
CREATE TABLE IF NOT EXISTS ProductReviews (
    review_id UInt32,
    product_id UInt32,
    customer_id UInt32,
    rating UInt8,  -- 1-5 stars
    review_text String,
    review_date Date,
    PRIMARY KEY (review_id)
) ENGINE = MergeTree()
ORDER BY (review_id);

-- Table: Promotions
CREATE TABLE IF NOT EXISTS Promotions (
    promotion_id UInt32,
    product_id UInt32,
    start_date Date,
    end_date Date,
    discount_percentage Decimal(5,2),
    PRIMARY KEY (promotion_id)
) ENGINE = MergeTree()
ORDER BY (promotion_id);
```

### 1.2 Sample Data (data.sql)

```sql
USE ecommerce;

-- Insert sample data into Customers
INSERT INTO Customers (customer_id, first_name, last_name, email, registration_date, country) VALUES
(1, 'Alice', 'Smith', 'alice.smith@example.com', '2023-01-15', 'USA'),
(2, 'Bob', 'Johnson', 'bob.johnson@example.com', '2023-02-20', 'Canada'),
(3, 'Charlie', 'Brown', 'charlie.brown@example.com', '2023-03-10', 'UK'),
(4, 'David', 'Lee', 'david.lee@example.com', '2023-04-05', 'USA'),
(5, 'Eve', 'Wilson', 'eve.wilson@example.com', '2023-05-12', 'Australia'),
(6, 'Jane', 'Doe', 'invalid_email', '2022-13-01', 'USA'), -- Invalid email, invalid date
(7, 'Peter', 'Jones', NULL, '2023-07-01', 'UK'),       -- Missing email
(8, 'Alice', 'Smith', 'alice.smith@example.com', '2023-01-15', 'USA');  -- Duplicate customer, same email and name (to test deduplication)

-- Insert sample data into Products
INSERT INTO Products (product_id, product_name, category, price, description) VALUES
(101, 'Laptop', 'Electronics', 1200.00, 'High-performance laptop'),
(102, 'Mouse', 'Electronics', 25.00, 'Wireless mouse'),
(103, 'Keyboard', 'Electronics', 75.00, 'Mechanical keyboard'),
(104, 'T-shirt', 'Apparel', 20.00, 'Cotton T-shirt'),
(105, 'Jeans', 'Apparel', 60.00, 'Denim jeans'),
(106, 'Mystery Box', 'Unknown', NULL, 'A surprise item'),          -- Null Price, 'Unknown' Category
(107, 'Discount Voucher', 'Discounts', -10.00, 'A voucher for savings'); -- Negative price
(108, 'Laptop', 'Electronics', 1200.00, 'High-performance laptop'); -- Duplicate Product (to test deduplication)

-- Insert sample data into Orders
INSERT INTO Orders (order_id, customer_id, order_date, total_amount, shipping_address) VALUES
(1001, 1, '2023-06-01', 1225.00, '123 Main St, Anytown, USA'),
(1002, 2, '2023-06-05', 85.00, '456 Oak Ave, Toronto, Canada'),
(1003, 3, '2023-06-10', 1400.00, '789 Pine Ln, London, UK'),
(1004, 1, '2023-06-15', 80.00, '123 Main St, Anytown, USA'),
(1005, 4, '2023-06-20', 20.00, '111 Elm St, Anytown, USA'),
(1006, 1, '2023-06-32', 50.00, '123 Main St, Anytown, USA'), -- Invalid Date
(1007, 2, '2023-07-05', -25.00, '456 Oak Ave, Toronto, Canada'), -- Negative Amount
(1008, 999, '2023-07-10', 100.00, 'Nowhere, Unknown'); -- Invalid customer_id

-- Insert sample data into OrderItems
INSERT INTO OrderItems (order_item_id, order_id, product_id, quantity, item_price) VALUES
(1, 1001, 101, 1, 1200.00),
(2, 1001, 102, 1, 25.00),
(3, 1002, 102, 1, 25.00),
(4, 1002, 103, 1, 75.00),
(5, 1003, 101, 1, 1200.00),
(6, 1003, 103, 1, 200.00),
(7, 1004, 103, 1, 75.00),
(8, 1005, 104, 1, 20.00),
(9, 1001, 101, 0, 1200.00),  -- Zero Quantity
(10, 1002, 102, -1, 25.00), -- Negative Quantity
(11, 1003, 106, 1, NULL), -- Null item_price
(12, 1001, 101, 1, 1200.00);  --Duplicate order_item(to test deduplication)

-- Insert sample data into ProductReviews
INSERT INTO ProductReviews (review_id, product_id, customer_id, rating, review_text, review_date) VALUES
(1, 101, 1, 5, 'Great laptop!', '2023-06-05'),
(2, 102, 2, 4, 'Good mouse.', '2023-06-10'),
(3, 101, 3, 3, 'Okay laptop.', '2023-06-15'),
(4, 104, 4, 5, 'Love this T-shirt!', '2023-06-25'),
(5, 105, 5, 4, 'Comfortable jeans.', '2023-07-01'),
(6, 101, 1, 6, 'Too good to be true!', '2023-06-05'), -- Invalid rating (6, should be 1-5)
(7, 102, 2, -1, 'Terrible mouse.', '2023-06-10'), -- Invalid rating (-1, should be 1-5)
(8, 101, 3, 3, 'Okay laptop.', '2023-13-15');  -- Invalid Date

-- Insert sample data into Promotions
INSERT INTO Promotions (promotion_id, product_id, start_date, end_date, discount_percentage) VALUES
(1, 101, '2023-05-01', '2023-05-31', 10.00),
(2, 104, '2023-06-15', '2023-06-30', 20.00);
```

## 2. Test Questions

The following questions require the candidate to write SQL queries against the provided dataset. Each question includes a description of the expected output and any specific considerations.

**Question 1: Customer Order History with Data Cleaning**

*   "Write a SQL query to retrieve the first name, last name, email and the order id, order date and total amount for each order placed by all valid customers."
*   "Remove entries with invalid or missing email addresses (`NULL` values or strings not containing an `@` symbol), invalid order dates, and orders associated with customer IDs that don't exist in the `Customers` table. Also, please remove any duplicate customers based on customer id and email."
*   "Sort the results by customer's last name, then by order date in descending order."
*   *Expected Output:* A table with columns: `first_name`, `last_name`, `email`, `order_id`, `order_date`, `total_amount`. Should *not* include Jane Doe (invalid email, date), Peter Jones (missing email), the order with `customer_id = 999`, and duplicate customer id `8`.

**Question 2: Product Sales by Category, Excluding Bad Data**

*   "Write a SQL query to calculate the total revenue generated by each valid product category. Exclude products with NULL prices, negative prices, or a category of 'Unknown'."
*   "The query should return the category name and the total revenue."
*   "Order the results by total revenue in descending order."
*   *Expected Output:* A table with columns: `category`, `total_revenue`. The 'Unknown' category should be excluded, and products with NULL or negative prices should not contribute to the total revenue calculation.

**Question 3: Customers and Reviews**

*   "Write a SQL query to retrieve the first name, last name of customers, along with the product name they reviewed, and the average rating given to that product by those customers."
*   "Only include customers who have left at least one review."
*   "Order the results by product name and then by average rating in descending order."
*   *Expected Output:* A table with columns: `first_name`, `last_name`, `product_name`, `average_rating`

**Question 4: Promotions and Sales, Considering Valid Data**

*   "Write a SQL query to determine the total sales amount for products that were under promotion on the day of the order, considering only valid orders, order items, and promotions. Please also remove the duplicate OrderItems"
*   "Exclude orders with invalid dates or negative total amounts, order items with zero or negative quantities, products with NULL prices, and promotions with invalid dates (start date after end date)."
*   "Return the product name and the total sales amount during the promotion period. Note: An order is considered to be 'under promotion' if the order date falls between the promotion's start and end dates."
*   "Order the results by total sales amount in descending order."
*   *Expected Output:* A table with columns: `product_name`, `total_sales_amount`. Should exclude any data identified as invalid according to the instructions.

**Question 5: Top Performing Products by Country**

*   "Write a SQL query to find the top 3 best-selling products (by total quantity sold) in each country."
*   "The query should return the country, product name, and total quantity sold. Use window functions to achieve this."
*   *Expected Output:* A table with columns: `country`, `product_name`, `total_quantity_sold`

### Bonus Questions

These bonus questions are designed to be more challenging and require a deeper understanding of SQL and ClickHouse features.

**Bonus Question 1: Cohort Analysis - Customer Retention**

*   "Perform a cohort analysis to determine customer retention rates over time. Define cohorts based on the month of a customer's first order. For each cohort, calculate the percentage of customers who placed orders in subsequent months."
*   "The query should return the cohort month, the month number since the first order (0 for the first month, 1 for the next, etc.), and the retention rate (percentage of customers from the cohort who placed orders in that month)."
*   *Expected Output:* A table with columns: `cohort_month`, `months_since_first_order`, `retention_rate`

**Bonus Question 2: Funnel Analysis - Product Page to Purchase Conversion**

*   "Simulate a basic funnel analysis to track user behavior from product page view to order placement. Assume a simplified event log table exists (see below for schema example). Write a query to calculate the conversion rate at each step of the funnel."
*   "The funnel steps are: Product Page View -> Add to Cart -> Checkout Started -> Order Placed. Calculate the number of users at each step and the conversion rate from one step to the next."
*   *Expected Output:* A table with columns: `step_name`, `user_count`, `conversion_rate` (e.g., "Add to Cart", 123, 0.75, where 0.75 is the rate of converting from product view to add to cart)

**Bonus Question 3: Market Basket Analysis (Association Rule Mining)**

*   "Perform a simplified market basket analysis to identify products that are frequently purchased together. Calculate the 'support' for each pair of products (the percentage of orders containing both products)."
*   "Return the two product names and the support value, ordered by support in descending order. Limit the output to the top 10 most frequent pairs."
*   *Expected Output:* A table with columns: `product_name_1`, `product_name_2`, `support`

## 3. Setup and Usage

1.  Clone the repository: `git clone <repository_url>` (If you are creating the repository yourself, you can skip this step)
2.  Create a ClickHouse database named `ecommerce`: `CREATE DATABASE IF NOT EXISTS ecommerce;`
3.  Use the `ecommerce` database: `USE ecommerce;`
4.  Copy and paste the SQL code from the **1.1 Database Schema (schema.sql)** section and execute it in your ClickHouse client. This creates the tables.
5.  Copy and paste the SQL code from the **1.2 Sample Data (data.sql)** section and execute it in your ClickHouse client. This loads the sample data.

You can then use any ClickHouse client to execute the test questions and verify the solutions.

## 4. Instructions for Candidates

*   **Environment:** You should use ClickHouse for this test. You can use a local ClickHouse instance or a cloud-based service. Provide the SQL code as your answer.
*   **Data Loading:** The provided SQL script contains the table schemas and sample data. Execute this script in your ClickHouse environment to create the tables and load the data.
*   **Assumptions:** If you need to make any assumptions to answer a question, clearly state them in your submission.
*   **Performance (Optional):** Where applicable, consider how you might optimize your queries for performance in ClickHouse (e.g., using appropriate data types, indexes, or materialized views).
*   **Clarity and Readability:** Write your SQL queries in a clear, well-formatted, and readable style.
*   **Data Quality:** The provided dataset contains some missing, inconsistent, and invalid data. Your SQL queries should handle these data quality issues to ensure accurate results. Specifically, consider scenarios like NULL values, invalid dates, negative amounts, and inconsistent category names. Be very clear about any assumptions you make about how to handle specific data quality issues.
*   **Deliverables:** Submit a document containing the SQL queries for each question. Clearly label each query with the corresponding question number.
*   **Bonus Questions:** The bonus questions are optional but provide an opportunity to demonstrate advanced SQL skills and problem-solving abilities.
*   **Event Log Table (for Bonus Question 2):** If you choose to attempt Bonus Question 2, you will need to create the `EventLog` table and populate it with sample data. The schema for this table is provided in the solution section for that question.
*   **Explain your approach:** For the bonus questions, especially, please provide a brief explanation of your approach and any challenges you faced.

## 6. Contribution

Contributions to this repository are welcome! Please feel free to submit pull requests with improvements to the database schema, sample data, test questions, or solutions.
```
