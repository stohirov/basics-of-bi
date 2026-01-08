# PostgreSQL Guide for BI Interns
**Complete Reference with Practical Examples**

---

## Table of Contents
1. [Introduction to PostgreSQL](#introduction)
2. [Basic SQL Queries](#basic-queries)
3. [Filtering and Sorting](#filtering-sorting)
4. [Aggregate Functions](#aggregate-functions)
5. [GROUP BY and HAVING](#group-by-having)
6. [JOINs](#joins)
7. [Subqueries](#subqueries)
8. [Common Table Expressions (CTEs)](#ctes)
9. [Window Functions](#window-functions)
10. [CASE Statements](#case-statements)
11. [String Functions](#string-functions)
12. [Date and Time Functions](#date-time-functions)
13. [Data Types](#data-types)
14. [Creating Tables and Views](#creating-tables-views)
15. [Indexes](#indexes)
16. [Best Practices for BI](#best-practices)

---

## 1. Introduction to PostgreSQL {#introduction}

PostgreSQL is a powerful, open-source relational database system. As a BI Intern, you'll use it to extract, transform, and analyze data.

### Connecting to PostgreSQL
```sql
-- Command line
psql -U username -d database_name -h hostname

-- Or use GUI tools like:
-- - pgAdmin
-- - DBeaver
-- - DataGrip
```

---

## 2. Basic SQL Queries {#basic-queries}

### SELECT - Retrieve Data
```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary 
FROM employees;

-- Select with alias
SELECT 
    first_name AS "First Name",
    last_name AS "Last Name",
    salary AS "Annual Salary"
FROM employees;

-- Select distinct values
SELECT DISTINCT department_id 
FROM employees;

-- LIMIT - Get first N rows
SELECT * FROM employees 
LIMIT 10;

-- OFFSET - Skip first N rows
SELECT * FROM employees 
LIMIT 10 OFFSET 20;  -- Rows 21-30
```

---

## 3. Filtering and Sorting {#filtering-sorting}

### WHERE Clause - Filter Rows
```sql
-- Basic conditions
SELECT * FROM employees 
WHERE salary > 50000;

-- Multiple conditions with AND
SELECT * FROM employees 
WHERE salary > 50000 AND department_id = 10;

-- Multiple conditions with OR
SELECT * FROM employees 
WHERE department_id = 10 OR department_id = 20;

-- IN operator
SELECT * FROM employees 
WHERE department_id IN (10, 20, 30);

-- BETWEEN operator
SELECT * FROM employees 
WHERE salary BETWEEN 40000 AND 60000;

-- LIKE operator (pattern matching)
SELECT * FROM employees 
WHERE first_name LIKE 'John%';  -- Starts with John

SELECT * FROM employees 
WHERE email LIKE '%@gmail.com';  -- Ends with @gmail.com

SELECT * FROM employees 
WHERE first_name LIKE '%oh%';  -- Contains 'oh'

-- IS NULL / IS NOT NULL
SELECT * FROM employees 
WHERE manager_id IS NULL;

SELECT * FROM employees 
WHERE phone_number IS NOT NULL;

-- NOT operator
SELECT * FROM employees 
WHERE NOT department_id = 10;
```

### ORDER BY - Sort Results
```sql
-- Sort ascending (default)
SELECT * FROM employees 
ORDER BY salary;

-- Sort descending
SELECT * FROM employees 
ORDER BY salary DESC;

-- Multiple columns
SELECT * FROM employees 
ORDER BY department_id ASC, salary DESC;

-- Sort by column position
SELECT first_name, last_name, salary 
FROM employees 
ORDER BY 3 DESC;  -- Sort by 3rd column (salary)
```

---

## 4. Aggregate Functions {#aggregate-functions}

### Basic Aggregations
```sql
-- COUNT - Count rows
SELECT COUNT(*) FROM employees;

SELECT COUNT(DISTINCT department_id) 
FROM employees;

-- SUM - Total
SELECT SUM(salary) AS total_salary 
FROM employees;

-- AVG - Average
SELECT AVG(salary) AS average_salary 
FROM employees;

-- MIN and MAX
SELECT 
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees;

-- Multiple aggregations
SELECT 
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    SUM(salary) AS total_payroll
FROM employees;
```

---

## 5. GROUP BY and HAVING {#group-by-having}

### GROUP BY - Aggregate by Groups
```sql
-- Count employees per department
SELECT 
    department_id,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;

-- Average salary per department
SELECT 
    department_id,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
ORDER BY avg_salary DESC;

-- Multiple columns in GROUP BY
SELECT 
    department_id,
    job_title,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id, job_title;
```

### HAVING - Filter Groups
```sql
-- Departments with more than 5 employees
SELECT 
    department_id,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;

-- Departments with average salary > 60000
SELECT 
    department_id,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 60000
ORDER BY avg_salary DESC;

-- Combination of WHERE and HAVING
SELECT 
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= '2020-01-01'  -- Filter before grouping
GROUP BY department_id
HAVING COUNT(*) > 3  -- Filter after grouping
ORDER BY avg_salary DESC;
```

---

## 6. JOINs {#joins}

### INNER JOIN
```sql
-- Join employees with departments
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Multiple joins
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    l.city,
    l.country
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id;
```

### LEFT JOIN (LEFT OUTER JOIN)
```sql
-- All employees, even those without departments
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- Find employees without departments
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

### RIGHT JOIN (RIGHT OUTER JOIN)
```sql
-- All departments, even those without employees
SELECT 
    d.department_name,
    e.first_name,
    e.last_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;
```

### FULL OUTER JOIN
```sql
-- All employees and all departments
SELECT 
    e.employee_id,
    e.first_name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;
```

### CROSS JOIN
```sql
-- Cartesian product (every combination)
SELECT 
    e.first_name,
    d.department_name
FROM employees e
CROSS JOIN departments d;
```

### SELF JOIN
```sql
-- Find employees and their managers
SELECT 
    e.first_name AS employee_name,
    m.first_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

---

## 7. Subqueries {#subqueries}

### Subquery in WHERE Clause
```sql
-- Employees with salary above average
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Employees in departments located in USA
SELECT first_name, last_name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location_id IN (
        SELECT location_id
        FROM locations
        WHERE country = 'USA'
    )
);
```

### Subquery in FROM Clause
```sql
-- Average salary by department, then filter
SELECT *
FROM (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) dept_avg
WHERE avg_salary > 60000;
```

### Subquery in SELECT Clause
```sql
-- Employee with their department's average salary
SELECT 
    first_name,
    last_name,
    salary,
    (SELECT AVG(salary) 
     FROM employees e2 
     WHERE e2.department_id = e1.department_id) AS dept_avg_salary
FROM employees e1;
```

### EXISTS
```sql
-- Departments that have employees
SELECT department_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);
```

---

## 8. Common Table Expressions (CTEs) {#ctes}

### Basic CTE
```sql
-- More readable than subqueries
WITH dept_avg AS (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT 
    e.first_name,
    e.last_name,
    e.salary,
    da.avg_salary
FROM employees e
JOIN dept_avg da ON e.department_id = da.department_id
WHERE e.salary > da.avg_salary;
```

### Multiple CTEs
```sql
WITH 
dept_stats AS (
    SELECT 
        department_id,
        COUNT(*) AS emp_count,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
high_paying_depts AS (
    SELECT department_id
    FROM dept_stats
    WHERE avg_salary > 60000
)
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.department_id IN (SELECT department_id FROM high_paying_depts);
```

### Recursive CTE
```sql
-- Employee hierarchy (organizational chart)
WITH RECURSIVE emp_hierarchy AS (
    -- Base case: top-level managers
    SELECT 
        employee_id,
        first_name,
        manager_id,
        1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees reporting to previous level
    SELECT 
        e.employee_id,
        e.first_name,
        e.manager_id,
        eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM emp_hierarchy
ORDER BY level, employee_id;
```

---

## 9. Window Functions {#window-functions}

**Window functions are crucial for BI work!**

### ROW_NUMBER()
```sql
-- Assign row numbers
SELECT 
    first_name,
    last_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;

-- Row number within each department
SELECT 
    first_name,
    last_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
    ) AS dept_salary_rank
FROM employees;
```

### RANK() and DENSE_RANK()
```sql
SELECT 
    first_name,
    last_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- RANK: 1, 2, 2, 4, 5 (skips 3)
-- DENSE_RANK: 1, 2, 2, 3, 4 (no skipping)
```

### NTILE()
```sql
-- Divide into quartiles
SELECT 
    first_name,
    last_name,
    salary,
    NTILE(4) OVER (ORDER BY salary) AS salary_quartile
FROM employees;
```

### LAG() and LEAD()
```sql
-- Compare with previous/next row
SELECT 
    employee_id,
    first_name,
    hire_date,
    salary,
    LAG(salary) OVER (ORDER BY hire_date) AS prev_hire_salary,
    LEAD(salary) OVER (ORDER BY hire_date) AS next_hire_salary
FROM employees;

-- Salary change from previous year
SELECT 
    employee_id,
    year,
    salary,
    LAG(salary) OVER (PARTITION BY employee_id ORDER BY year) AS prev_year_salary,
    salary - LAG(salary) OVER (PARTITION BY employee_id ORDER BY year) AS salary_change
FROM employee_salaries;
```

### FIRST_VALUE() and LAST_VALUE()
```sql
SELECT 
    employee_id,
    first_name,
    department_id,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
    ) AS highest_dept_salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_dept_salary
FROM employees;
```

### SUM() as Window Function
```sql
-- Running total
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Running total by customer
SELECT 
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS customer_running_total
FROM orders;
```

### Moving Average
```sql
-- 7-day moving average
SELECT 
    date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM daily_revenue;
```

---

## 10. CASE Statements {#case-statements}

### Simple CASE
```sql
SELECT 
    first_name,
    last_name,
    salary,
    CASE 
        WHEN salary < 40000 THEN 'Low'
        WHEN salary BETWEEN 40000 AND 70000 THEN 'Medium'
        WHEN salary > 70000 THEN 'High'
        ELSE 'Unknown'
    END AS salary_category
FROM employees;
```

### CASE with Aggregation
```sql
-- Count employees by salary range
SELECT 
    COUNT(CASE WHEN salary < 40000 THEN 1 END) AS low_salary_count,
    COUNT(CASE WHEN salary BETWEEN 40000 AND 70000 THEN 1 END) AS mid_salary_count,
    COUNT(CASE WHEN salary > 70000 THEN 1 END) AS high_salary_count
FROM employees;

-- Conditional sum
SELECT 
    department_id,
    SUM(CASE WHEN job_title = 'Manager' THEN salary ELSE 0 END) AS manager_payroll,
    SUM(CASE WHEN job_title != 'Manager' THEN salary ELSE 0 END) AS non_manager_payroll
FROM employees
GROUP BY department_id;
```

### Pivot Table with CASE
```sql
-- Sales by product and quarter
SELECT 
    product_id,
    SUM(CASE WHEN quarter = 'Q1' THEN sales ELSE 0 END) AS q1_sales,
    SUM(CASE WHEN quarter = 'Q2' THEN sales ELSE 0 END) AS q2_sales,
    SUM(CASE WHEN quarter = 'Q3' THEN sales ELSE 0 END) AS q3_sales,
    SUM(CASE WHEN quarter = 'Q4' THEN sales ELSE 0 END) AS q4_sales
FROM quarterly_sales
GROUP BY product_id;
```

---

## 11. String Functions {#string-functions}

```sql
-- CONCAT - Concatenate strings
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;

-- Alternative: || operator
SELECT first_name || ' ' || last_name AS full_name
FROM employees;

-- UPPER and LOWER
SELECT 
    UPPER(first_name) AS uppercase_name,
    LOWER(last_name) AS lowercase_name
FROM employees;

-- LENGTH
SELECT 
    first_name,
    LENGTH(first_name) AS name_length
FROM employees;

-- SUBSTRING
SELECT 
    email,
    SUBSTRING(email FROM 1 FOR POSITION('@' IN email) - 1) AS username
FROM employees;

-- LEFT and RIGHT
SELECT 
    LEFT(first_name, 3) AS first_three,
    RIGHT(last_name, 3) AS last_three
FROM employees;

-- TRIM, LTRIM, RTRIM
SELECT 
    TRIM('  hello  ') AS trimmed,
    LTRIM('  hello  ') AS left_trimmed,
    RTRIM('  hello  ') AS right_trimmed;

-- REPLACE
SELECT 
    phone_number,
    REPLACE(phone_number, '-', '') AS phone_no_dashes
FROM employees;

-- POSITION / STRPOS
SELECT 
    email,
    POSITION('@' IN email) AS at_position
FROM employees;

-- SPLIT_PART
SELECT 
    email,
    SPLIT_PART(email, '@', 1) AS username,
    SPLIT_PART(email, '@', 2) AS domain
FROM employees;

-- INITCAP - Capitalize first letter
SELECT INITCAP('john doe') AS formatted_name;
```

---

## 12. Date and Time Functions {#date-time-functions}

### Current Date/Time
```sql
-- Current timestamp
SELECT NOW();
SELECT CURRENT_TIMESTAMP;

-- Current date
SELECT CURRENT_DATE;

-- Current time
SELECT CURRENT_TIME;
```

### Date Arithmetic
```sql
-- Add/subtract intervals
SELECT 
    CURRENT_DATE,
    CURRENT_DATE + INTERVAL '7 days' AS one_week_later,
    CURRENT_DATE - INTERVAL '1 month' AS one_month_ago,
    CURRENT_DATE + INTERVAL '2 years' AS two_years_later;

-- Age calculation
SELECT 
    first_name,
    birth_date,
    AGE(birth_date) AS age,
    AGE(CURRENT_DATE, birth_date) AS age_precise
FROM employees;
```

### Extract Parts
```sql
SELECT 
    hire_date,
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    EXTRACT(MONTH FROM hire_date) AS hire_month,
    EXTRACT(DAY FROM hire_date) AS hire_day,
    EXTRACT(DOW FROM hire_date) AS day_of_week,  -- 0=Sunday
    EXTRACT(QUARTER FROM hire_date) AS quarter
FROM employees;

-- Alternative: DATE_PART
SELECT DATE_PART('year', hire_date) AS hire_year
FROM employees;
```

### Date Formatting
```sql
SELECT 
    hire_date,
    TO_CHAR(hire_date, 'YYYY-MM-DD') AS iso_format,
    TO_CHAR(hire_date, 'Mon DD, YYYY') AS readable_format,
    TO_CHAR(hire_date, 'Day, Month DD, YYYY') AS full_format,
    TO_CHAR(hire_date, 'YYYY-Q') AS year_quarter
FROM employees;
```

### Date Truncation
```sql
SELECT 
    order_date,
    DATE_TRUNC('day', order_date) AS day,
    DATE_TRUNC('week', order_date) AS week,
    DATE_TRUNC('month', order_date) AS month,
    DATE_TRUNC('quarter', order_date) AS quarter,
    DATE_TRUNC('year', order_date) AS year
FROM orders;
```

### Date Difference
```sql
-- Days between dates
SELECT 
    hire_date,
    CURRENT_DATE - hire_date AS days_employed;

-- Months between dates
SELECT 
    hire_date,
    EXTRACT(YEAR FROM AGE(CURRENT_DATE, hire_date)) * 12 +
    EXTRACT(MONTH FROM AGE(CURRENT_DATE, hire_date)) AS months_employed
FROM employees;
```

### Common BI Date Patterns
```sql
-- Sales by month
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS monthly_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;

-- Year-over-year comparison
SELECT 
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(amount) AS sales
FROM orders
GROUP BY 
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY year, month;

-- Last 30 days
SELECT *
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days';
```

---

## 13. Data Types {#data-types}

### Numeric Types
```sql
-- INTEGER (or INT)
age INT

-- BIGINT (large integers)
user_id BIGINT

-- DECIMAL / NUMERIC (exact precision)
price DECIMAL(10, 2)  -- 10 digits, 2 after decimal

-- REAL / FLOAT (approximate)
temperature REAL

-- SERIAL (auto-increment)
id SERIAL PRIMARY KEY
```

### Character Types
```sql
-- CHAR(n) - fixed length
state_code CHAR(2)

-- VARCHAR(n) - variable length
email VARCHAR(255)

-- TEXT - unlimited length
description TEXT
```

### Date/Time Types
```sql
-- DATE
birth_date DATE

-- TIME
start_time TIME

-- TIMESTAMP
created_at TIMESTAMP

-- TIMESTAMP WITH TIME ZONE
updated_at TIMESTAMPTZ

-- INTERVAL
duration INTERVAL
```

### Boolean
```sql
is_active BOOLEAN
```

### JSON
```sql
-- JSON
metadata JSON

-- JSONB (binary, faster)
settings JSONB

-- Example queries
SELECT data->>'name' AS name
FROM users
WHERE data->>'status' = 'active';
```

### Arrays
```sql
-- Array column
tags TEXT[]

-- Query arrays
SELECT *
FROM articles
WHERE 'postgres' = ANY(tags);
```

---

## 14. Creating Tables and Views {#creating-tables-views}

### CREATE TABLE
```sql
-- Basic table
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    hire_date DATE NOT NULL,
    salary DECIMAL(10, 2),
    department_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- With foreign key
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- With check constraint
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) CHECK (price >= 0),
    stock_quantity INT CHECK (stock_quantity >= 0)
);
```

### ALTER TABLE
```sql
-- Add column
ALTER TABLE employees
ADD COLUMN phone_number VARCHAR(20);

-- Drop column
ALTER TABLE employees
DROP COLUMN phone_number;

-- Rename column
ALTER TABLE employees
RENAME COLUMN phone_number TO contact_number;

-- Change data type
ALTER TABLE employees
ALTER COLUMN salary TYPE DECIMAL(12, 2);

-- Add constraint
ALTER TABLE employees
ADD CONSTRAINT unique_email UNIQUE (email);
```

### CREATE VIEW
```sql
-- Simple view
CREATE VIEW active_employees AS
SELECT 
    employee_id,
    first_name,
    last_name,
    email,
    department_id
FROM employees
WHERE status = 'active';

-- Complex view for reporting
CREATE VIEW sales_summary AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    p.category,
    COUNT(DISTINCT o.order_id) AS order_count,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.price) AS revenue
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
GROUP BY DATE_TRUNC('month', order_date), p.category;

-- Use the view
SELECT * FROM sales_summary
WHERE month >= '2024-01-01'
ORDER BY revenue DESC;
```

### Materialized View
```sql
-- Create materialized view (stores results)
CREATE MATERIALIZED VIEW monthly_sales_mv AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS total_sales,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- Refresh materialized view
REFRESH MATERIALIZED VIEW monthly_sales_mv;

-- Query materialized view
SELECT * FROM monthly_sales_mv;
```

---

## 15. Indexes {#indexes}

### Create Index
```sql
-- Basic index
CREATE INDEX idx_employees_last_name 
ON employees(last_name);

-- Multi-column index
CREATE INDEX idx_employees_dept_salary 
ON employees(department_id, salary);

-- Unique index
CREATE UNIQUE INDEX idx_employees_email 
ON employees(email);

-- Partial index
CREATE INDEX idx_active_employees 
ON employees(last_name)
WHERE status = 'active';
```

### Drop Index
```sql
DROP INDEX idx_employees_last_name;
```

### When to Use Indexes
- Columns frequently used in WHERE clauses
- Columns used in JOIN conditions
- Columns used in ORDER BY
- Foreign key columns

### When NOT to Use Indexes
- Small tables
- Columns with frequent INSERT/UPDATE/DELETE
- Columns with low cardinality (few distinct values)

---

## 16. Best Practices for BI {#best-practices}

### 1. Write Readable Queries
```sql
-- Good: Formatted and clear
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    e.salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.hire_date >= '2020-01-01'
ORDER BY e.salary DESC;

-- Bad: Hard to read
SELECT e.employee_id,e.first_name,e.last_name,d.department_name,e.salary FROM employees e JOIN departments d ON e.department_id=d.department_id WHERE e.hire_date>='2020-01-01' ORDER BY e.salary DESC;
```

### 2. Use CTEs for Complex Queries
```sql
-- More maintainable
WITH recent_orders AS (
    SELECT *
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
),
customer_totals AS (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(amount) AS total_spent
    FROM recent_orders
    GROUP BY customer_id
)
SELECT 
    c.customer_name,
    ct.order_count,
    ct.total_spent
FROM customers c
JOIN customer_totals ct ON c.customer_id = ct.customer_id;
```

### 3. Optimize Query Performance
```sql
-- Use EXPLAIN to analyze queries
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE customer_id = 1000;

-- Avoid SELECT *
-- Good
SELECT order_id, customer_id, total
FROM orders;

-- Bad (retrieves unnecessary data)
SELECT * FROM orders;

-- Use WHERE before JOIN when possible
-- Good
SELECT e.*, d.*
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 50000;  -- Filter early
```

### 4. Handle NULLs Properly
```sql
-- Use COALESCE for default values
SELECT 
    first_name,
    COALESCE(middle_name, '') AS middle_name,
    last_name
FROM employees;

-- Use NULLIF
SELECT 
    revenue,
    NULLIF(expenses, 0) AS expenses,  -- Avoid division by zero
    revenue / NULLIF(expenses, 0) AS profit_margin
FROM financials;
```

### 5. Use Window Functions for Rankings
```sql
-- Instead of multiple subqueries
SELECT 
    employee_id,
    first_name,
    department_id,
    salary,
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dept_rank,
    PERCENT_RANK() OVER (ORDER BY salary) AS salary_percentile
FROM employees;
```

### 6. Date Filtering Tips
```sql
-- Use DATE_TRUNC for period comparisons
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS monthly_sales
FROM orders
WHERE order_date >= DATE_TRUNC('year', CURRENT_DATE)  -- This year
GROUP BY DATE_TRUNC('month', order_date);

-- Use intervals for relative dates
SELECT *
FROM orders
WHERE order_date BETWEEN 
    CURRENT_DATE - INTERVAL '7 days' AND 
    CURRENT_DATE;
```

### 7. Create Reusable Views
```sql
-- Base metrics view
CREATE VIEW daily_metrics AS
SELECT 
    DATE_TRUNC('day', created_at) AS date,
    COUNT(DISTINCT user_id) AS daily_active_users,
    COUNT(*) AS events,
    COUNT(DISTINCT session_id) AS sessions
FROM events
GROUP BY DATE_TRUNC('day', created_at);

-- Use in reports
SELECT * FROM daily_metrics
WHERE date >= CURRENT_DATE - INTERVAL '30 days';
```

### 8. Comment Your Code
```sql
-- Calculate customer lifetime value
-- Including all purchases and returns over entire history
SELECT 
    customer_id,
    -- Total purchases
    SUM(CASE WHEN type = 'purchase' THEN amount ELSE 0 END) AS total_purchases,
    -- Total returns
    SUM(CASE WHEN type = 'return' THEN amount ELSE 0 END) AS total_returns,
    -- Net LTV
    SUM(CASE 
        WHEN type = 'purchase' THEN amount 
        WHEN type = 'return' THEN -amount 
        ELSE 0 
    END) AS lifetime_value
FROM transactions
GROUP BY customer_id;
```

### 9. Test with Sample Data First
```sql
-- Test logic on small dataset
SELECT *
FROM orders
LIMIT 100;

-- Then run full query
SELECT *
FROM orders;
```

### 10. Common BI Query Patterns
```sql
-- Cohort analysis
WITH user_cohorts AS (
    SELECT 
        user_id,
        DATE_TRUNC('month', first_order_date) AS cohort_month
    FROM (
        SELECT 
            user_id,
            MIN(order_date) AS first_order_date
        FROM orders
        GROUP BY user_id
    ) first_orders
)
SELECT 
    uc.cohort_month,
    DATE_TRUNC('month', o.order_date) AS order_month,
    COUNT(DISTINCT o.user_id) AS active_users,
    SUM(o.amount) AS revenue
FROM user_cohorts uc
JOIN orders o ON uc.user_id = o.user_id
GROUP BY uc.cohort_month, DATE_TRUNC('month', o.order_date);

-- Funnel analysis
SELECT 
    COUNT(DISTINCT CASE WHEN step = 'view' THEN user_id END) AS views,
    COUNT(DISTINCT CASE WHEN step = 'add_to_cart' THEN user_id END) AS add_to_cart,
    COUNT(DISTINCT CASE WHEN step = 'checkout' THEN user_id END) AS checkout,
    COUNT(DISTINCT CASE WHEN step = 'purchase' THEN user_id END) AS purchase
FROM user_events
WHERE event_date = CURRENT_DATE;

-- Retention rate
WITH active_users AS (
    SELECT DISTINCT user_id, DATE_TRUNC('month', activity_date) AS month
    FROM user_activity
)
SELECT 
    curr.month AS current_month,
    COUNT(DISTINCT curr.user_id) AS active_users,
    COUNT(DISTINCT prev.user_id) AS retained_users,
    ROUND(100.0 * COUNT(DISTINCT prev.user_id) / COUNT(DISTINCT curr.user_id), 2) AS retention_rate
FROM active_users curr
LEFT JOIN active_users prev 
    ON curr.user_id = prev.user_id 
    AND prev.month = curr.month - INTERVAL '1 month'
GROUP BY curr.month
ORDER BY curr.month;
```

---

## Quick Reference Commands

```sql
-- Database operations
\l                          -- List databases
\c database_name           -- Connect to database
\dt                        -- List tables
\d table_name              -- Describe table
\dv                        -- List views
\q                         -- Quit

-- Help
\?                         -- psql help
\h SQL_COMMAND            -- SQL command help

-- Timing
\timing on                 -- Show query execution time
```

---

## Practice Exercises

### Exercise 1: Basic Queries
Write queries to:
1. Find all employees with salary > 60000
2. Count employees per department
3. Find the top 5 highest paid employees

### Exercise 2: Joins
1. List all employees with their department names
2. Find departments with no employees
3. Show employee hierarchy (employee and their manager)

### Exercise 3: Window Functions
1. Rank employees by salary within each department
2. Calculate running total of sales by date
3. Find the salary difference between each employee and the department average

### Exercise 4: Date Analysis
1. Calculate monthly sales for the last 12 months
2. Find customers who made purchases in consecutive months
3. Calculate year-over-year growth rate

### Exercise 5: Advanced
1. Build a customer segmentation query (RFM analysis)
2. Create a sales funnel with conversion rates
3. Identify trending products (sales increasing month-over-month)

---

## Additional Resources

- **Official Documentation**: https://www.postgresql.org/docs/
- **Practice**: Mode Analytics SQL Tutorial, LeetCode SQL problems
- **BI Tools**: Tableau, Power BI, Metabase (for visualizing query results)
- **GUI Tools**: pgAdmin, DBeaver, TablePlus, DataGrip

---

## Tips for BI Intern Success

1. **Understand the business context** - Know what metrics matter
2. **Ask questions** - Clarify requirements before writing complex queries
3. **Document your work** - Comment queries and create documentation
4. **Test thoroughly** - Verify results with sample data
5. **Optimize** - Monitor query performance as data grows
6. **Learn visualization tools** - Queries are just the start; presenting insights is key
7. **Stay curious** - Explore the data to find interesting patterns
8. **Version control** - Use Git for your SQL scripts

Good luck with your BI internship! 🚀