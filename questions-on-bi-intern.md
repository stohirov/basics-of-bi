# BI Intern Interview Questions & Answers
**Complete Preparation Guide**

---

## Table of Contents
1. [SQL Technical Questions](#sql-technical)
2. [Database Concepts](#database-concepts)
3. [BI Tools & Concepts](#bi-tools-concepts)
4. [Data Analysis Questions](#data-analysis)
5. [Scenario-Based Questions](#scenario-based)
6. [Behavioral Questions](#behavioral)
7. [Problem-Solving Questions](#problem-solving)
8. [Common Coding Challenges](#coding-challenges)

---

## 1. SQL Technical Questions {#sql-technical}

### Q1: What is the difference between WHERE and HAVING?

**Answer:**
- **WHERE** filters rows BEFORE grouping (works on individual rows)
- **HAVING** filters groups AFTER grouping (works on aggregated results)

**Example:**
```sql
-- WHERE: Filter employees before grouping
SELECT department_id, COUNT(*) AS emp_count
FROM employees
WHERE salary > 50000  -- Filter individual rows first
GROUP BY department_id;

-- HAVING: Filter departments after grouping
SELECT department_id, COUNT(*) AS emp_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;  -- Filter groups

-- BOTH: Use together
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= '2020-01-01'  -- First filter rows
GROUP BY department_id
HAVING AVG(salary) > 60000;  -- Then filter groups
```

---

### Q2: Explain different types of JOINs with examples

**Answer:**

**INNER JOIN** - Returns only matching rows from both tables
```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
-- Only employees who have departments
```

**LEFT JOIN** - Returns all rows from left table, matching rows from right
```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
-- All employees, even those without departments (NULL for dept_name)
```

**RIGHT JOIN** - Returns all rows from right table, matching rows from left
```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
-- All departments, even those without employees
```

**FULL OUTER JOIN** - Returns all rows from both tables
```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
-- All employees AND all departments
```

**CROSS JOIN** - Cartesian product (every combination)
```sql
SELECT s.size, c.color
FROM sizes s
CROSS JOIN colors c;
-- Every size with every color
```

**SELF JOIN** - Join table to itself
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
-- Each employee with their manager
```

---

### Q3: What is the difference between RANK(), DENSE_RANK(), and ROW_NUMBER()?

**Answer:**

```sql
SELECT 
    name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
    RANK() OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students;
```

**Results:**
```
name    | score | row_num | rank | dense_rank
--------|-------|---------|------|------------
Alice   | 95    | 1       | 1    | 1
Bob     | 95    | 2       | 1    | 1
Carol   | 90    | 3       | 3    | 2
Dave    | 90    | 4       | 3    | 2
Eve     | 85    | 5       | 5    | 3
```

- **ROW_NUMBER()**: Unique sequential number (1, 2, 3, 4, 5)
- **RANK()**: Same rank for ties, skips numbers (1, 1, 3, 3, 5)
- **DENSE_RANK()**: Same rank for ties, no gaps (1, 1, 2, 2, 3)

---

### Q4: How do you find duplicate records in a table?

**Answer:**

**Method 1: Using GROUP BY and HAVING**
```sql
-- Find duplicate emails
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Show all duplicate records
SELECT *
FROM users
WHERE email IN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
)
ORDER BY email;
```

**Method 2: Using Window Functions**
```sql
WITH duplicates AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY user_id) AS rn
    FROM users
)
SELECT *
FROM duplicates
WHERE rn > 1;  -- All duplicates except first occurrence
```

---

### Q5: How do you find the second highest salary?

**Answer:**

**Method 1: Using LIMIT and OFFSET**
```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

**Method 2: Using Subquery**
```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Method 3: Using DENSE_RANK (Best for handling ties)**
```sql
WITH ranked_salaries AS (
    SELECT 
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT DISTINCT salary
FROM ranked_salaries
WHERE rank = 2;
```

**Method 4: Nth highest salary (more flexible)**
```sql
-- For any Nth highest
WITH ranked AS (
    SELECT 
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT salary
FROM ranked
WHERE rank = 2;  -- Change to N for Nth highest
```

---

### Q6: What is a CTE and why use it?

**Answer:**

**CTE (Common Table Expression)** is a temporary named result set that exists only within a single query.

**Advantages:**
- More readable than nested subqueries
- Can be referenced multiple times
- Supports recursion
- Easier to debug and maintain

**Example:**
```sql
-- Without CTE (Hard to read)
SELECT *
FROM (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
) dept_avg
WHERE avg_sal > 60000;

-- With CTE (Much clearer)
WITH dept_averages AS (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT *
FROM dept_averages
WHERE avg_salary > 60000;

-- Multiple CTEs
WITH 
high_performers AS (
    SELECT * FROM employees WHERE performance_score > 4
),
dept_stats AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM high_performers
    GROUP BY department_id
)
SELECT * FROM dept_stats;
```

---

### Q7: Explain the difference between UNION and UNION ALL

**Answer:**

- **UNION**: Removes duplicate rows (slower, requires sorting)
- **UNION ALL**: Keeps all rows including duplicates (faster)

```sql
-- UNION (removes duplicates)
SELECT name FROM employees_2023
UNION
SELECT name FROM employees_2024;
-- If "John" exists in both tables, appears once

-- UNION ALL (keeps duplicates)
SELECT name FROM employees_2023
UNION ALL
SELECT name FROM employees_2024;
-- If "John" exists in both tables, appears twice

-- Performance tip: Use UNION ALL when you know there are no duplicates
-- or when duplicates are acceptable
```

**When to use:**
- **UNION**: When you need unique results across tables
- **UNION ALL**: When you want all records or know there are no duplicates (better performance)

---

### Q8: How do you calculate running totals?

**Answer:**

```sql
-- Running total of sales by date
SELECT 
    order_date,
    daily_sales,
    SUM(daily_sales) OVER (ORDER BY order_date) AS running_total
FROM daily_sales
ORDER BY order_date;

-- Running total by category
SELECT 
    category,
    order_date,
    sales,
    SUM(sales) OVER (
        PARTITION BY category 
        ORDER BY order_date
    ) AS category_running_total
FROM sales;

-- Running average
SELECT 
    date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7days
FROM daily_revenue;
```

---

### Q9: How do you pivot data in SQL?

**Answer:**

**Pivoting** transforms rows into columns.

```sql
-- Original data:
-- product | quarter | sales
-- A       | Q1      | 100
-- A       | Q2      | 150
-- B       | Q1      | 200

-- Pivot to:
-- product | Q1  | Q2  | Q3  | Q4
-- A       | 100 | 150 | ... | ...
-- B       | 200 | ... | ... | ...

SELECT 
    product,
    SUM(CASE WHEN quarter = 'Q1' THEN sales ELSE 0 END) AS Q1,
    SUM(CASE WHEN quarter = 'Q2' THEN sales ELSE 0 END) AS Q2,
    SUM(CASE WHEN quarter = 'Q3' THEN sales ELSE 0 END) AS Q3,
    SUM(CASE WHEN quarter = 'Q4' THEN sales ELSE 0 END) AS Q4
FROM quarterly_sales
GROUP BY product;

-- Using CROSSTAB (PostgreSQL extension)
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT *
FROM CROSSTAB(
    'SELECT product, quarter, sales FROM quarterly_sales ORDER BY 1,2',
    'SELECT DISTINCT quarter FROM quarterly_sales ORDER BY 1'
) AS ct(product TEXT, Q1 INT, Q2 INT, Q3 INT, Q4 INT);
```

---

### Q10: What are window functions and give examples?

**Answer:**

**Window functions** perform calculations across rows related to the current row without collapsing results like GROUP BY.

**Common Window Functions:**

```sql
-- 1. ROW_NUMBER(): Unique sequential number
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- 2. LAG/LEAD: Access previous/next rows
SELECT 
    date,
    sales,
    LAG(sales) OVER (ORDER BY date) AS prev_day_sales,
    sales - LAG(sales) OVER (ORDER BY date) AS daily_change
FROM daily_sales;

-- 3. NTILE: Divide into N buckets
SELECT 
    customer_id,
    total_spent,
    NTILE(4) OVER (ORDER BY total_spent) AS quartile
FROM customer_spending;

-- 4. FIRST_VALUE/LAST_VALUE
SELECT 
    employee_id,
    department,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS highest_dept_salary
FROM employees;

-- 5. Cumulative distribution
SELECT 
    product,
    revenue,
    PERCENT_RANK() OVER (ORDER BY revenue) AS percentile
FROM product_revenue;
```

---

## 2. Database Concepts {#database-concepts}

### Q11: What is normalization and why is it important?

**Answer:**

**Normalization** is organizing database tables to reduce redundancy and improve data integrity.

**Normal Forms:**

**1NF (First Normal Form):**
- Each cell contains atomic (single) values
- Each column has unique name
- Order doesn't matter

```sql
-- NOT 1NF (multiple values in one cell)
CREATE TABLE orders (
    order_id INT,
    products VARCHAR  -- "Apple, Banana, Orange"
);

-- 1NF (atomic values)
CREATE TABLE order_items (
    order_id INT,
    product VARCHAR
);
```

**2NF (Second Normal Form):**
- Meets 1NF
- No partial dependencies (all non-key attributes depend on entire primary key)

```sql
-- NOT 2NF
CREATE TABLE order_details (
    order_id INT,
    product_id INT,
    product_name VARCHAR,  -- Depends only on product_id
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- 2NF (separate tables)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR
);
```

**3NF (Third Normal Form):**
- Meets 2NF
- No transitive dependencies (non-key attributes don't depend on other non-key attributes)

**Benefits:**
- Reduces data redundancy
- Easier to maintain data integrity
- More efficient updates
- Prevents anomalies

**When to denormalize:**
- For reporting/analytics (BI use case)
- When read performance is critical
- In data warehouses

---

### Q12: What is the difference between DELETE, TRUNCATE, and DROP?

**Answer:**

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| **What it does** | Removes rows | Removes all rows | Removes table |
| **WHERE clause** | Yes | No | N/A |
| **Rollback** | Yes (with transaction) | No (in most DBs) | No |
| **Speed** | Slower | Faster | Fastest |
| **Triggers** | Fires triggers | Doesn't fire | Doesn't fire |
| **Identity reset** | No | Yes | Yes |
| **Space** | Doesn't release | Releases | Releases |

```sql
-- DELETE: Remove specific rows
DELETE FROM employees WHERE department = 'Sales';
-- Can be rolled back
-- Logs each row deletion
-- Keeps table structure

-- TRUNCATE: Remove all rows quickly
TRUNCATE TABLE employees;
-- Cannot be rolled back (in most databases)
-- Faster than DELETE
-- Resets auto-increment
-- Keeps table structure

-- DROP: Remove entire table
DROP TABLE employees;
-- Cannot be rolled back
-- Removes table definition and data
-- Cannot undo
```

---

### Q13: What is an index and when should you use it?

**Answer:**

An **index** is a database object that speeds up data retrieval (like a book index).

**Types:**
- **B-tree index**: Default, good for most queries
- **Hash index**: Fast for equality comparisons
- **Unique index**: Ensures no duplicates
- **Partial index**: Index subset of rows
- **Composite index**: Multiple columns

```sql
-- Create index
CREATE INDEX idx_employee_lastname ON employees(last_name);

-- Composite index
CREATE INDEX idx_emp_dept_salary ON employees(department_id, salary);

-- Unique index
CREATE UNIQUE INDEX idx_email ON users(email);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
```

**When to use indexes:**
- ✅ Columns in WHERE clauses
- ✅ Columns in JOIN conditions
- ✅ Columns in ORDER BY
- ✅ Foreign keys
- ✅ Columns used in GROUP BY

**When NOT to use:**
- ❌ Small tables (< 1000 rows)
- ❌ Frequently updated columns
- ❌ Columns with low cardinality (few distinct values like gender)
- ❌ Tables with heavy INSERT/UPDATE/DELETE operations

**Trade-off:**
- Faster SELECTs
- Slower INSERT/UPDATE/DELETE
- More storage space

---

### Q14: What is the difference between a primary key and a foreign key?

**Answer:**

**Primary Key:**
- Uniquely identifies each row
- Cannot be NULL
- Only one per table
- Automatically creates an index

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,  -- Primary key
    email VARCHAR UNIQUE,
    name VARCHAR
);
```

**Foreign Key:**
- Links to primary key in another table
- Enforces referential integrity
- Can be NULL (if optional relationship)
- Can have multiple per table

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,  -- Foreign key
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**Referential Integrity:**
```sql
-- Cascade delete: Delete orders when customer is deleted
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    ON DELETE CASCADE;

-- Restrict delete: Prevent deleting customer if orders exist
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    ON DELETE RESTRICT;

-- Set NULL: Set foreign key to NULL when referenced record is deleted
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    ON DELETE SET NULL;
```

---

### Q15: What is ACID in databases?

**Answer:**

**ACID** ensures reliable database transactions:

**A - Atomicity:**
- All operations in transaction succeed or all fail
- "All or nothing"
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both succeed or both fail
```

**C - Consistency:**
- Database moves from one valid state to another
- All rules/constraints are satisfied
```sql
-- Constraint ensures consistency
ALTER TABLE accounts ADD CONSTRAINT check_positive_balance 
CHECK (balance >= 0);
```

**I - Isolation:**
- Concurrent transactions don't interfere
- Transactions appear to run sequentially
```sql
-- Transaction 1 doesn't see uncommitted changes from Transaction 2
BEGIN;
SELECT * FROM accounts;  -- Sees consistent snapshot
```

**D - Durability:**
- Committed transactions persist even after system failure
- Changes are written to permanent storage

---

## 3. BI Tools & Concepts {#bi-tools-concepts}

### Q16: What is the difference between OLTP and OLAP?

**Answer:**

| Feature | OLTP | OLAP |
|---------|------|------|
| **Purpose** | Transaction Processing | Analytical Processing |
| **Focus** | Daily operations | Business intelligence |
| **Queries** | Simple, fast | Complex, slower |
| **Data** | Current, detailed | Historical, aggregated |
| **Users** | Many concurrent | Few analysts |
| **Normalization** | Highly normalized (3NF) | Denormalized (star/snowflake) |
| **Database** | Operational DB | Data Warehouse |
| **Examples** | Order processing, Banking | Sales analysis, Trends |
| **Updates** | Frequent | Periodic (ETL) |

**OLTP Example:**
```sql
-- Insert a new order (fast, simple)
INSERT INTO orders (customer_id, order_date, total)
VALUES (123, CURRENT_DATE, 99.99);
```

**OLAP Example:**
```sql
-- Analyze sales trends (complex, analytical)
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    product_category,
    SUM(total) AS revenue,
    COUNT(*) AS order_count,
    AVG(total) AS avg_order_value
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY DATE_TRUNC('month', order_date), product_category
ORDER BY month, revenue DESC;
```

---

### Q17: What is ETL?

**Answer:**

**ETL = Extract, Transform, Load**

**Extract:**
- Get data from source systems
- Sources: Databases, APIs, files, CRM, ERP

```sql
-- Extract from source database
SELECT * FROM production_db.orders
WHERE order_date >= CURRENT_DATE - INTERVAL '1 day';
```

**Transform:**
- Clean, validate, format data
- Apply business rules
- Aggregate, join, filter

```sql
-- Transform: Clean and standardize
SELECT 
    UPPER(TRIM(customer_name)) AS customer_name,
    COALESCE(email, 'unknown@example.com') AS email,
    CASE 
        WHEN country_code = 'US' THEN 'USA'
        WHEN country_code = 'GB' THEN 'UK'
        ELSE country_code
    END AS country,
    order_total::DECIMAL(10,2) AS order_total
FROM staging.raw_orders;
```

**Load:**
- Insert into data warehouse/data mart
- Can be full load or incremental

```sql
-- Load into warehouse
INSERT INTO warehouse.fact_orders
SELECT * FROM staging.transformed_orders;
```

**ETL Tools:**
- Apache Airflow
- Talend
- Informatica
- AWS Glue
- Azure Data Factory

---

### Q18: What is a Data Warehouse vs Data Lake?

**Answer:**

**Data Warehouse:**
- Structured, processed data
- Schema-on-write
- SQL queries
- Business intelligence
- Expensive storage
- Examples: Snowflake, Redshift, BigQuery

**Data Lake:**
- Raw, unstructured data
- Schema-on-read
- All file types
- Big data analytics, ML
- Cheaper storage
- Examples: AWS S3, Azure Data Lake

```
Data Pipeline:
[Data Sources] → [Data Lake] → [ETL] → [Data Warehouse] → [BI Tools]
   (Raw)         (Store all)   (Process)  (Structured)    (Reports)
```

---

### Q19: What is a Star Schema vs Snowflake Schema?

**Answer:**

**Star Schema:**
- One central fact table
- Dimension tables directly connected
- Denormalized dimensions
- Simpler queries, faster
- More redundancy

```
        [Dimension: Customer]
                |
                |
        [Fact: Sales] --- [Dimension: Product]
                |
                |
        [Dimension: Time]
```

**Snowflake Schema:**
- Fact table with normalized dimensions
- Dimensions have sub-dimensions
- Less redundancy
- More complex queries

```
[Dimension: Country]
        |
[Dimension: City]
        |
[Dimension: Customer]
        |
[Fact: Sales]
```

**Example:**

**Star Schema:**
```sql
-- Simple join
SELECT 
    c.customer_name,
    p.product_name,
    t.year,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_time t ON f.time_key = t.time_key
GROUP BY c.customer_name, p.product_name, t.year;
```

---

### Q20: What KPIs would you track for an e-commerce business?

**Answer:**

**Revenue Metrics:**
- Total Revenue
- Revenue Growth Rate (MoM, YoY)
- Average Order Value (AOV)
- Revenue per User

**Customer Metrics:**
- Customer Acquisition Cost (CAC)
- Customer Lifetime Value (CLV)
- Repeat Purchase Rate
- Customer Churn Rate
- Net Promoter Score (NPS)

**Sales Metrics:**
- Conversion Rate
- Cart Abandonment Rate
- Sales by Channel
- Sales by Category

**Operational Metrics:**
- Inventory Turnover
- Order Fulfillment Time
- Return Rate
- Website Traffic

**SQL Examples:**

```sql
-- Average Order Value
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    AVG(total_amount) AS average_order_value
FROM orders
WHERE status = 'completed'
GROUP BY DATE_TRUNC('month', order_date);

-- Customer Lifetime Value
SELECT 
    customer_id,
    COUNT(DISTINCT order_id) AS total_orders,
    SUM(total_amount) AS lifetime_value,
    MIN(order_date) AS first_order_date,
    MAX(order_date) AS last_order_date
FROM orders
GROUP BY customer_id;

-- Conversion Rate
SELECT 
    DATE_TRUNC('day', date) AS date,
    SUM(CASE WHEN event = 'view' THEN 1 ELSE 0 END) AS views,
    SUM(CASE WHEN event = 'purchase' THEN 1 ELSE 0 END) AS purchases,
    ROUND(100.0 * SUM(CASE WHEN event = 'purchase' THEN 1 ELSE 0 END) / 
          NULLIF(SUM(CASE WHEN event = 'view' THEN 1 ELSE 0 END), 0), 2) AS conversion_rate
FROM events
GROUP BY DATE_TRUNC('day', date);

-- Repeat Purchase Rate
WITH customer_orders AS (
    SELECT 
        customer_id,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id
)
SELECT 
    COUNT(CASE WHEN order_count > 1 THEN 1 END) AS repeat_customers,
    COUNT(*) AS total_customers,
    ROUND(100.0 * COUNT(CASE WHEN order_count > 1 THEN 1 END) / COUNT(*), 2) AS repeat_rate
FROM customer_orders;
```

---

## 4. Data Analysis Questions {#data-analysis}

### Q21: How would you identify and handle outliers in data?

**Answer:**

**Methods to Identify Outliers:**

**1. IQR (Interquartile Range) Method:**
```sql
WITH stats AS (
    SELECT 
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) AS q1,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) AS q3
    FROM employees
),
iqr_bounds AS (
    SELECT 
        q1,
        q3,
        q3 - q1 AS iqr,
        q1 - 1.5 * (q3 - q1) AS lower_bound,
        q3 + 1.5 * (q3 - q1) AS upper_bound
    FROM stats
)
SELECT 
    e.employee_id,
    e.salary,
    CASE 
        WHEN e.salary < ib.lower_bound THEN 'Below Outlier'
        WHEN e.salary > ib.upper_bound THEN 'Above Outlier'
        ELSE 'Normal'
    END AS outlier_status
FROM employees e
CROSS JOIN iqr_bounds ib;
```

**2. Z-Score Method:**
```sql
WITH stats AS (
    SELECT 
        AVG(salary) AS mean_salary,
        STDDEV(salary) AS std_salary
    FROM employees
)
SELECT 
    employee_id,
    salary,
    (salary - stats.mean_salary) / stats.std_salary AS z_score,
    CASE 
        WHEN ABS((salary - stats.mean_salary) / stats.std_salary) > 3 
        THEN 'Outlier'
        ELSE 'Normal'
    END AS status
FROM employees, stats;
```

**Handling Outliers:**
- **Remove**: If data error
- **Cap**: Set to threshold value
- **Transform**: Log transformation
- **Keep**: If legitimate extreme value
- **Investigate**: Understand cause

---

### Q22: How do you calculate customer retention rate?

**Answer:**

**Retention Rate Formula:**
```
Retention Rate = (Customers at End - New Customers) / Customers at Start × 100
```

**SQL Implementation:**

```sql
-- Method 1: Basic retention (monthly)
WITH 
monthly_customers AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        customer_id
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date), customer_id
),
retention_calc AS (
    SELECT 
        curr.month AS current_month,
        COUNT(DISTINCT curr.customer_id) AS customers_this_month,
        COUNT(DISTINCT prev.customer_id) AS retained_from_prev_month,
        ROUND(100.0 * COUNT(DISTINCT prev.customer_id) / 
              COUNT(DISTINCT curr.customer_id), 2) AS retention_rate
    FROM monthly_customers curr
    LEFT JOIN monthly_customers prev 
        ON curr.customer_id = prev.customer_id 
        AND prev.month = curr.month - INTERVAL '1 month'
    GROUP BY curr.month
)
SELECT * FROM retention_calc
ORDER BY current_month;

-- Method 2: Cohort retention analysis
WITH 
first_purchase AS (
    SELECT 
        customer_id,
        DATE_TRUNC('month', MIN(order_date)) AS cohort_month
    FROM orders
    GROUP BY customer_id
),
customer_orders AS (
    SELECT 
        o.customer_id,
        fp.cohort_month,
        DATE_TRUNC('month', o.order_date) AS order_month,
        EXTRACT(MONTH FROM AGE(o.order_date, fp.cohort_month)) AS months_since_first
    FROM orders o
    JOIN first_purchase fp ON o.customer_id = fp.customer_id
)
SELECT 
    cohort_month,
    months_since_first AS month_number,
    COUNT(DISTINCT customer_id) AS active_customers,
    ROUND(100.0 * COUNT(DISTINCT customer_id) / 
          FIRST_VALUE(COUNT(DISTINCT customer_id)) OVER (
              PARTITION BY cohort_month ORDER BY months_since_first
          ), 2) AS retention_rate
FROM customer_orders
GROUP BY cohort_month, months_since_first
ORDER BY cohort_month, months_since_first;
```

---

### Q23: How would you perform an A/B test analysis?

**Answer:**

**A/B Test Steps:**
1. Define hypothesis
2. Split users into control (A) and variant (B)
3. Collect data
4. Analyze results
5. Calculate statistical significance

**SQL Analysis:**

```sql
-- 1. Basic metrics comparison
SELECT 
    test_group,
    COUNT(DISTINCT user_id) AS users,
    SUM(converted) AS conversions,
    ROUND(100.0 * SUM(converted) / COUNT(DISTINCT user_id), 2) AS conversion_rate,
    AVG(revenue) AS avg_revenue,
    SUM(revenue) AS total_revenue
FROM ab_test_results
GROUP BY test_group;

-- Result:
-- test_group | users | conversions | conversion_rate | avg_revenue
-- A          | 10000 | 1200        | 12.00           | 25.50
-- B          | 10000 | 1350        | 13.50           | 27.20

-- 2. Statistical significance (simplified)
WITH group_stats AS (
    SELECT 
        test_group,
        COUNT(*) AS n,
        AVG(CASE WHEN converted = 1 THEN 1.0 ELSE 0.0 END) AS conv_rate,
        STDDEV(CASE WHEN converted = 1 THEN 1.0 ELSE 0.0 END) AS std_dev
    FROM ab_test_results
    GROUP BY test_group
),
comparison AS (
    SELECT 
        (SELECT conv_rate FROM group_stats WHERE test_group = 'B') - 
        (SELECT conv_rate FROM group_stats WHERE test_group = 'A') AS diff,
        SQRT(
            (SELECT std_dev FROM group_stats WHERE test_group = 'A')^2 / 
            (SELECT n FROM group_stats WHERE test_group = 'A') +
            (SELECT std_dev FROM group_stats WHERE test_group = 'B')^2 / 
            (SELECT n FROM group_stats WHERE test_group = 'B')
        ) AS std_error
)
SELECT 
    diff,
    std_error,
    diff / std_error AS z_score,
    CASE 
        WHEN ABS(diff / std_error) > 1.96 THEN 'Statistically Significant (95%)'
        ELSE 'Not Significant'
    END AS result
FROM comparison;

-- 3. Time-series analysis
SELECT 
    DATE_TRUNC('day', test_date) AS date,
    test_group,
    COUNT(DISTINCT user_id) AS daily_users,
    SUM(converted) AS conversions,
    ROUND(100.0 * SUM(converted) / COUNT(DISTINCT user_id), 2) AS conversion_rate
FROM ab_test_results
GROUP BY DATE_TRUNC('day', test_date), test_group
ORDER BY date, test_group;
```

**Interpretation:**
- If z-score > 1.96 (or < -1.96), result is statistically significant at 95% confidence
- Variant B is better if conversion rate is significantly higher
- Check for: sample size adequacy, test duration, external factors

---

### Q24: Explain RFM Analysis

**Answer:**

**RFM = Recency, Frequency, Monetary**

Segments customers based on:
- **Recency**: How recently they purchased
- **Frequency**: How often they purchase
- **Monetary**: How much they spend

**SQL Implementation:**

```sql
WITH customer_rfm AS (
    SELECT 
        customer_id,
        -- Recency: Days since last purchase
        CURRENT_DATE - MAX(order_date) AS recency_days,
        -- Frequency: Number of orders
        COUNT(DISTINCT order_id) AS frequency,
        -- Monetary: Total spending
        SUM(total_amount) AS monetary
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '1 year'
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT 
        customer_id,
        recency_days,
        frequency,
        monetary,
        -- Score 1-5 (5 is best)
        NTILE(5) OVER (ORDER BY recency_days ASC) AS r_score,  -- Lower is better
        NTILE(5) OVER (ORDER BY frequency DESC) AS f_score,    -- Higher is better
        NTILE(5) OVER (ORDER BY monetary DESC) AS m_score      -- Higher is better
    FROM customer_rfm
),
rfm_segments AS (
    SELECT 
        *,
        CONCAT(r_score, f_score, m_score) AS rfm_score,
        CASE 
            WHEN r_score >= 4 AND f_score >= 4 THEN 'Champions'
            WHEN r_score >= 3 AND f_score >= 3 THEN 'Loyal Customers'
            WHEN r_score >= 4 AND f_score <= 2 THEN 'Potential Loyalists'
            WHEN r_score >= 3 AND f_score <= 2 THEN 'Recent Customers'
            WHEN r_score <= 2 AND f_score >= 4 THEN 'At Risk'
            WHEN r_score <= 2 AND f_score >= 2 THEN 'Needs Attention'
            WHEN r_score <= 2 AND f_score <= 2 THEN 'Lost Customers'
            ELSE 'Other'
        END AS customer_segment
    FROM rfm_scores
)
SELECT 
    customer_segment,
    COUNT(*) AS customer_count,
    AVG(recency_days) AS avg_recency,
    AVG(frequency) AS avg_frequency,
    AVG(monetary) AS avg_monetary
FROM rfm_segments
GROUP BY customer_segment
ORDER BY customer_count DESC;
```

**Business Actions:**
- **Champions**: Reward, ask for referrals
- **At Risk**: Special offers, re-engagement
- **Lost Customers**: Win-back campaigns

---

## 5. Scenario-Based Questions {#scenario-based}

### Q25: You notice a sudden drop in daily active users. How would you investigate?

**Answer:**

**Step-by-step Investigation:**

**1. Verify the data:**
```sql
-- Check for data collection issues
SELECT 
    date,
    COUNT(DISTINCT user_id) AS daily_active_users,
    COUNT(*) AS total_events
FROM user_events
WHERE date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY date
ORDER BY date;

-- Check for NULL values or data quality issues
SELECT 
    date,
    COUNT(*) AS total_rows,
    COUNT(user_id) AS non_null_users,
    COUNT(*) - COUNT(user_id) AS null_users
FROM user_events
WHERE date >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY date;
```

**2. Segment the analysis:**
```sql
-- By platform
SELECT 
    date,
    platform,
    COUNT(DISTINCT user_id) AS dau
FROM user_events
WHERE date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY date, platform
ORDER BY date, platform;

-- By user cohort
SELECT 
    date,
    user_type,  -- new vs returning
    COUNT(DISTINCT user_id) AS dau
FROM user_events
WHERE date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY date, user_type
ORDER BY date;

-- By geography
SELECT 
    date,
    country,
    COUNT(DISTINCT user_id) AS dau
FROM user_events
WHERE date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY date, country
ORDER BY date, country;
```

**3. Check for external factors:**
- App updates/releases
- Marketing campaigns ended
- Seasonality (weekends, holidays)
- Technical issues (API downtime)
- Competitor actions

**4. Compare to historical patterns:**
```sql
-- Year-over-year comparison
SELECT 
    EXTRACT(DOY FROM date) AS day_of_year,
    EXTRACT(YEAR FROM date) AS year,
    COUNT(DISTINCT user_id) AS dau
FROM user_events
WHERE date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY EXTRACT(DOY FROM date), EXTRACT(YEAR FROM date)
ORDER BY day_of_year, year;
```

**5. Report findings:**
- Present data with visualizations
- Highlight affected segments
- Propose hypotheses
- Suggest next steps

---

### Q26: How would you design a dashboard for sales executives?

**Answer:**

**Key Components:**

**1. High-level KPIs (Top of dashboard):**
- Total Revenue (current period)
- Revenue vs Target
- Revenue Growth (vs previous period)
- Average Deal Size
- Number of Deals Closed

```sql
-- KPI queries
WITH current_period AS (
    SELECT 
        SUM(amount) AS total_revenue,
        COUNT(*) AS deals_closed,
        AVG(amount) AS avg_deal_size
    FROM sales
    WHERE sale_date >= DATE_TRUNC('month', CURRENT_DATE)
),
previous_period AS (
    SELECT SUM(amount) AS prev_revenue
    FROM sales
    WHERE sale_date >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
        AND sale_date < DATE_TRUNC('month', CURRENT_DATE)
)
SELECT 
    cp.total_revenue,
    cp.deals_closed,
    cp.avg_deal_size,
    ROUND(100.0 * (cp.total_revenue - pp.prev_revenue) / pp.prev_revenue, 2) AS growth_rate
FROM current_period cp, previous_period pp;
```

**2. Revenue Trend (Line chart):**
```sql
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS revenue,
    COUNT(*) AS deals
FROM sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '12 months'
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

**3. Sales by Region (Bar chart):**
```sql
SELECT 
    region,
    SUM(amount) AS revenue,
    COUNT(*) AS deals,
    AVG(amount) AS avg_deal_size
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
WHERE sale_date >= DATE_TRUNC('month', CURRENT_DATE)
GROUP BY region
ORDER BY revenue DESC;
```

**4. Sales Pipeline (Funnel):**
```sql
SELECT 
    stage,
    COUNT(*) AS opportunities,
    SUM(expected_value) AS total_value
FROM sales_pipeline
WHERE status = 'active'
GROUP BY stage
ORDER BY 
    CASE stage
        WHEN 'Prospecting' THEN 1
        WHEN 'Qualification' THEN 2
        WHEN 'Proposal' THEN 3
        WHEN 'Negotiation' THEN 4
        WHEN 'Closed Won' THEN 5
    END;
```

**5. Top Performers (Table):**
```sql
SELECT 
    sales_rep,
    COUNT(*) AS deals_closed,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_deal_size,
    RANK() OVER (ORDER BY SUM(amount) DESC) AS revenue_rank
FROM sales
WHERE sale_date >= DATE_TRUNC('quarter', CURRENT_DATE)
GROUP BY sales_rep
ORDER BY total_revenue DESC
LIMIT 10;
```

**6. Win Rate by Product:**
```sql
SELECT 
    product_category,
    COUNT(*) AS total_opportunities,
    SUM(CASE WHEN status = 'won' THEN 1 ELSE 0 END) AS won_deals,
    ROUND(100.0 * SUM(CASE WHEN status = 'won' THEN 1 ELSE 0 END) / COUNT(*), 2) AS win_rate
FROM opportunities
WHERE created_date >= CURRENT_DATE - INTERVAL '6 months'
GROUP BY product_category
ORDER BY win_rate DESC;
```

**Design Principles:**
- Clean, uncluttered layout
- Most important metrics at top
- Interactive filters (date range, region, product)
- Drill-down capability
- Mobile-responsive
- Auto-refresh

**Tools:** Tableau, Power BI, Looker, Metabase

---

### Q27: How would you detect fraudulent transactions?

**Answer:**

**Red Flags to Look For:**

**1. Unusual transaction patterns:**
```sql
-- High-value transactions compared to user history
WITH user_stats AS (
    SELECT 
        user_id,
        AVG(amount) AS avg_transaction,
        STDDEV(amount) AS std_transaction
    FROM transactions
    WHERE transaction_date >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY user_id
)
SELECT 
    t.transaction_id,
    t.user_id,
    t.amount,
    us.avg_transaction,
    (t.amount - us.avg_transaction) / us.std_transaction AS z_score
FROM transactions t
JOIN user_stats us ON t.user_id = us.user_id
WHERE t.transaction_date >= CURRENT_DATE - INTERVAL '7 days'
    AND (t.amount - us.avg_transaction) / us.std_transaction > 3;  -- 3 standard deviations
```

**2. Rapid transaction velocity:**
```sql
-- Multiple transactions in short time window
SELECT 
    user_id,
    COUNT(*) AS transaction_count,
    SUM(amount) AS total_amount,
    MIN(transaction_date) AS first_transaction,
    MAX(transaction_date) AS last_transaction
FROM transactions
WHERE transaction_date >= CURRENT_DATE - INTERVAL '1 hour'
GROUP BY user_id
HAVING COUNT(*) >= 5  -- 5+ transactions in 1 hour
ORDER BY transaction_count DESC;
```

**3. Geographic anomalies:**
```sql
-- Transactions from multiple locations in short time
WITH location_changes AS (
    SELECT 
        user_id,
        transaction_id,
        transaction_date,
        country,
        LAG(country) OVER (PARTITION BY user_id ORDER BY transaction_date) AS prev_country,
        LAG(transaction_date) OVER (PARTITION BY user_id ORDER BY transaction_date) AS prev_transaction_date
    FROM transactions
)
SELECT *
FROM location_changes
WHERE country != prev_country
    AND transaction_date - prev_transaction_date < INTERVAL '2 hours';
```

**4. First-time behaviors:**
```sql
-- New accounts making large purchases
WITH first_transactions AS (
    SELECT 
        user_id,
        MIN(transaction_date) AS first_transaction_date
    FROM transactions
    GROUP BY user_id
)
SELECT 
    t.user_id,
    t.transaction_id,
    t.amount,
    ft.first_transaction_date,
    t.transaction_date - ft.first_transaction_date AS days_since_first
FROM transactions t
JOIN first_transactions ft ON t.user_id = ft.user_id
WHERE t.amount > 1000  -- Large transaction
    AND t.transaction_date - ft.first_transaction_date < INTERVAL '1 day';  -- Very new account
```

**5. Pattern matching known fraud:**
```sql
-- Transactions matching known fraudulent patterns
SELECT 
    t.*,
    fp.pattern_name,
    fp.risk_score
FROM transactions t
JOIN fraud_patterns fp ON 
    t.merchant_id = fp.merchant_id
    AND t.amount BETWEEN fp.min_amount AND fp.max_amount
    AND t.country = fp.country
WHERE t.transaction_date >= CURRENT_DATE - INTERVAL '1 day'
    AND fp.risk_score >= 0.8;
```

**Fraud Score Model:**
```sql
WITH fraud_signals AS (
    SELECT 
        transaction_id,
        user_id,
        amount,
        -- Velocity signal
        COUNT(*) OVER (
            PARTITION BY user_id 
            ORDER BY transaction_date 
            RANGE BETWEEN INTERVAL '1 hour' PRECEDING AND CURRENT ROW
        ) AS recent_transaction_count,
        -- Amount signal
        AVG(amount) OVER (PARTITION BY user_id) AS user_avg_amount,
        -- Time signal
        EXTRACT(HOUR FROM transaction_date) AS hour_of_day
    FROM transactions
)
SELECT 
    transaction_id,
    user_id,
    amount,
    -- Calculate fraud score
    (CASE WHEN recent_transaction_count > 5 THEN 30 ELSE 0 END +
     CASE WHEN amount > user_avg_amount * 3 THEN 40 ELSE 0 END +
     CASE WHEN hour_of_day BETWEEN 1 AND 5 THEN 20 ELSE 0 END +
     CASE WHEN ... THEN 10 ELSE 0 END) AS fraud_score
FROM fraud_signals
WHERE fraud_score > 50;  -- Threshold for review
```

**Response Actions:**
- Flag for manual review
- Request additional verification
- Temporary hold on transaction
- Alert user
- Block if high confidence

---

## 6. Behavioral Questions {#behavioral}

### Q28: Tell me about a time you found an interesting insight in data

**Answer Structure (STAR method):**

**Situation:**
"During my coursework/previous role, I was analyzing customer purchase data for an e-commerce company."

**Task:**
"I was asked to identify why sales were declining in a specific product category."

**Action:**
"I performed a deep dive analysis:
- Segmented data by customer demographics, time periods, and purchase channels
- Used SQL to identify patterns in purchase behavior
- Created cohort analysis to track customer retention
- Found that customers who purchased Product X had 40% lower repeat purchase rates"

**Result:**
"Discovered that Product X had quality issues reported in reviews during that period. Shared findings with product team. After addressing quality issues, repeat purchase rate increased by 25% within 3 months."

**Key Points:**
- Be specific with numbers
- Show your analytical process
- Demonstrate business impact
- Show collaboration with teams

---

### Q29: How do you handle tight deadlines?

**Answer:**

"I prioritize effectively and communicate clearly:

1. **Clarify requirements** - Ensure I understand what's truly urgent
2. **Break down tasks** - Divide work into manageable chunks
3. **Quick wins first** - Deliver partial results if full analysis takes time
4. **Communicate progress** - Keep stakeholders updated
5. **Ask for help** - Leverage team when needed

**Example:**
Recently had to deliver a sales report by EOD. I:
- Confirmed which metrics were most critical (revenue, top products)
- Delivered those first within 2 hours
- Completed additional analysis next day
- Set up automation to prevent rush next time"

---

### Q30: Why do you want to work in Business Intelligence?

**Answer (customize to your story):**

"I'm passionate about turning data into actionable insights because:

1. **Love problem-solving** - BI combines analytical thinking with business strategy
2. **See direct impact** - My analyses help companies make better decisions
3. **Continuous learning** - Technology and tools constantly evolving
4. **Cross-functional** - Work with diverse teams (sales, marketing, product)

**Specific interest in this company:**
- [Company's industry] aligns with my interests
- Impressed by [specific project/tool/culture]
- Want to learn from [specific team/leader]
- Excited about [company's data challenges]"

---

## 7. Problem-Solving Questions {#problem-solving}

### Q31: Calculate the month-over-month growth rate

**Answer:**

```sql
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS total_sales
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT 
    month,
    total_sales,
    LAG(total_sales) OVER (ORDER BY month) AS prev_month_sales,
    total_sales - LAG(total_sales) OVER (ORDER BY month) AS sales_change,
    ROUND(
        100.0 * (total_sales - LAG(total_sales) OVER (ORDER BY month)) / 
        LAG(total_sales) OVER (ORDER BY month), 
        2
    ) AS growth_rate_percent
FROM monthly_sales
ORDER BY month;
```

---

### Q32: Find customers who purchased both Product A and Product B

**Answer:**

```sql
-- Method 1: Using INTERSECT
SELECT customer_id
FROM orders
WHERE product_id = 'A'

INTERSECT

SELECT customer_id
FROM orders
WHERE product_id = 'B';

-- Method 2: Using GROUP BY and HAVING
SELECT customer_id
FROM orders
WHERE product_id IN ('A', 'B')
GROUP BY customer_id
HAVING COUNT(DISTINCT product_id) = 2;

-- Method 3: Using self-join
SELECT DISTINCT a.customer_id
FROM orders a
JOIN orders b ON a.customer_id = b.customer_id
WHERE a.product_id = 'A' AND b.product_id = 'B';

-- Method 4: Find customers who bought A, B, and C (all three)
SELECT customer_id
FROM orders
WHERE product_id IN ('A', 'B', 'C')
GROUP BY customer_id
HAVING COUNT(DISTINCT product_id) = 3;
```

---

### Q33: Calculate cumulative sum of sales

**Answer:**

```sql
-- Basic running total
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS cumulative_sales
FROM orders
ORDER BY order_date;

-- Running total by category
SELECT 
    order_date,
    category,
    amount,
    SUM(amount) OVER (
        PARTITION BY category 
        ORDER BY order_date
    ) AS category_cumulative_sales
FROM orders
ORDER BY category, order_date;

-- Running total with reset each year
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY EXTRACT(YEAR FROM order_date)
        ORDER BY order_date
    ) AS ytd_sales
FROM orders
ORDER BY order_date;
```

---

### Q34: Find the top 3 products by sales in each category

**Answer:**

```sql
-- Using window functions
WITH product_sales AS (
    SELECT 
        category,
        product_id,
        product_name,
        SUM(sales_amount) AS total_sales,
        RANK() OVER (PARTITION BY category ORDER BY SUM(sales_amount) DESC) AS sales_rank
    FROM sales
    GROUP BY category, product_id, product_name
)
SELECT 
    category,
    product_name,
    total_sales,
    sales_rank
FROM product_sales
WHERE sales_rank <= 3
ORDER BY category, sales_rank;

-- Alternative: Using DENSE_RANK (handles ties differently)
WITH product_sales AS (
    SELECT 
        category,
        product_name,
        SUM(sales_amount) AS total_sales,
        DENSE_RANK() OVER (PARTITION BY category ORDER BY SUM(sales_amount) DESC) AS sales_rank
    FROM sales
    GROUP BY category, product_name
)
SELECT *
FROM product_sales
WHERE sales_rank <= 3;
```

---

### Q35: Calculate customer churn rate

**Answer:**

```sql
-- Define churn: No activity in last 90 days
WITH customer_last_activity AS (
    SELECT 
        customer_id,
        MAX(activity_date) AS last_activity_date,
        CURRENT_DATE - MAX(activity_date) AS days_since_last_activity
    FROM customer_activities
    GROUP BY customer_id
),
churn_status AS (
    SELECT 
        customer_id,
        last_activity_date,
        days_since_last_activity,
        CASE 
            WHEN days_since_last_activity > 90 THEN 'Churned'
            ELSE 'Active'
        END AS status
    FROM customer_last_activity
)
SELECT 
    status,
    COUNT(*) AS customer_count,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM churn_status
GROUP BY status;

-- Monthly churn rate
WITH monthly_active AS (
    SELECT 
        DATE_TRUNC('month', activity_date) AS month,
        COUNT(DISTINCT customer_id) AS active_customers
    FROM customer_activities
    GROUP BY DATE_TRUNC('month', activity_date)
)
SELECT 
    curr.month,
    curr.active_customers AS current_month_customers,
    prev.active_customers AS previous_month_customers,
    prev.active_customers - curr.active_customers AS churned_customers,
    ROUND(100.0 * (prev.active_customers - curr.active_customers) / prev.active_customers, 2) AS churn_rate
FROM monthly_active curr
LEFT JOIN monthly_active prev ON prev.month = curr.month - INTERVAL '1 month'
WHERE prev.month IS NOT NULL
ORDER BY curr.month;
```

---

## 8. Common Coding Challenges {#coding-challenges}

### Q36: Write a query to find employees earning more than their manager

**Answer:**

```sql
SELECT 
    e.employee_id,
    e.name AS employee_name,
    e.salary AS employee_salary,
    m.name AS manager_name,
    m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

---

### Q37: Find departments with no employees

**Answer:**

```sql
-- Method 1: Using LEFT JOIN
SELECT d.department_id, d.department_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
WHERE e.employee_id IS NULL;

-- Method 2: Using NOT EXISTS
SELECT d.department_id, d.department_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);

-- Method 3: Using NOT IN
SELECT department_id, department_name
FROM departments
WHERE department_id NOT IN (
    SELECT DISTINCT department_id
    FROM employees
    WHERE department_id IS NOT NULL
);
```

---

### Q38: Write a query to deduplicate a table

**Answer:**

```sql
-- Find duplicates first
SELECT 
    email,
    COUNT(*) AS duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Method 1: Delete duplicates keeping first occurrence
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id)
    FROM users
    GROUP BY email
);

-- Method 2: Using window function (safer - shows what will be deleted)
WITH duplicates AS (
    SELECT 
        id,
        email,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
    FROM users
)
SELECT * FROM duplicates WHERE rn > 1;  -- These will be deleted

-- Then delete:
DELETE FROM users
WHERE id IN (
    SELECT id
    FROM (
        SELECT 
            id,
            ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
        FROM users
    ) t
    WHERE rn > 1
);

-- Method 3: Create clean table
CREATE TABLE users_clean AS
SELECT DISTINCT ON (email) *
FROM users
ORDER BY email, id;
```

---

### Q39: Calculate the median value

**Answer:**

```sql
-- PostgreSQL: Using PERCENTILE_CONT
SELECT 
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees;

-- For median by group
SELECT 
    department_id,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees
GROUP BY department_id;

-- Alternative method (works in most databases)
WITH ordered_salaries AS (
    SELECT 
        salary,
        ROW_NUMBER() OVER (ORDER BY salary) AS rn,
        COUNT(*) OVER () AS total_count
    FROM employees
)
SELECT AVG(salary) AS median_salary
FROM ordered_salaries
WHERE rn IN (
    FLOOR((total_count + 1) / 2.0),
    CEIL((total_count + 1) / 2.0)
);
```

---

### Q40: Find consecutive dates

**Answer:**

```sql
-- Find users who logged in on consecutive days
WITH login_dates AS (
    SELECT DISTINCT
        user_id,
        DATE(login_timestamp) AS login_date
    FROM user_logins
),
date_groups AS (
    SELECT 
        user_id,
        login_date,
        login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date)::INTEGER AS grp
    FROM login_dates
)
SELECT 
    user_id,
    MIN(login_date) AS streak_start,
    MAX(login_date) AS streak_end,
    COUNT(*) AS consecutive_days
FROM date_groups
GROUP BY user_id, grp
HAVING COUNT(*) >= 3  -- At least 3 consecutive days
ORDER BY user_id, streak_start;

-- Simpler: Find users with activity yesterday and today
SELECT DISTINCT l1.user_id
FROM user_logins l1
JOIN user_logins l2 ON l1.user_id = l2.user_id
WHERE DATE(l1.login_timestamp) = CURRENT_DATE
    AND DATE(l2.login_timestamp) = CURRENT_DATE - 1;
```

---

## Quick Tips for Interview Success

### Technical Preparation:
1. ✅ Practice on LeetCode SQL section
2. ✅ Know window functions cold
3. ✅ Understand JOINs deeply
4. ✅ Practice explaining your thought process
5. ✅ Write clean, readable SQL

### During Interview:
1. 🎯 Ask clarifying questions
2. 🎯 Think out loud
3. 🎯 Start with simple solution, then optimize
4. 🎯 Test with sample data
5. 🎯 Consider edge cases

### Common Mistakes to Avoid:
1. ❌ Not asking about data structure
2. ❌ Jumping straight to code
3. ❌ Not considering NULL values
4. ❌ Forgetting about performance
5. ❌ Not explaining your approach

### Red Flags in Answers:
- "I don't know" without trying
- Overcomplicating simple problems
- Not considering edge cases
- Poor SQL formatting
- Not testing logic

---

## Sample Interview Flow

**Interviewer:** "How would you find the top 10 customers by revenue?"

**Good Answer:**
"Let me make sure I understand the requirements:
1. Do we want total revenue all-time or for a specific period?
2. Should we include refunds or only positive revenue?
3. Are there any customer types to exclude (e.g., internal test accounts)?

Assuming we want all-time revenue including returns, here's my approach:

```sql
SELECT 
    customer_id,
    customer_name,
    SUM(order_amount) AS total_revenue,
    COUNT(DISTINCT order_id) AS total_orders,
    AVG(order_amount) AS avg_order_value
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status IN ('completed', 'shipped')  -- Exclude cancelled
GROUP BY customer_id, customer_name
ORDER BY total_revenue DESC
LIMIT 10;
```

This query groups by customer, sums their revenue, and sorts to get the top 10. I'm also including order count and average order value for additional context."

---

## Resources for Further Practice

1. **SQL Practice:**
   - LeetCode SQL Problems
   - HackerRank SQL
   - Mode Analytics SQL Tutorial
   - SQLZoo

2. **BI Concepts:**
   - Kimball Group (data warehousing)
   - Tableau/Power BI tutorials
   - Google Analytics Academy

3. **Mock Interviews:**
   - Pramp
   - interviewing.io

4. **YouTube Channels:**
   - Alex The Analyst
   - DataCamp
   - Codebasics

---

## Final Checklist Before Interview

- [ ] Review SQL fundamentals
- [ ] Practice 10+ coding problems
- [ ] Prepare questions to ask interviewer
- [ ] Review company's products/data
- [ ] Prepare STAR stories
- [ ] Test your setup (for remote interviews)
- [ ] Have paper/whiteboard ready
- [ ] Get good sleep!

Good luck with your interview! 🚀
