# Essential SQL Guide for Data Analytics

This guide covers essential SQL concepts every data analyst should know, with sample input data and expected outputs. Whether you are learning SQL from scratch or looking for a quick reference, these practical examples will show you exactly how data transforms with each query.

---

## 📚 Table of Contents: What You Will Learn

This guide is structured to cover everything from foundational querying to senior-level data manipulation. The questions span the following core analytics categories:

* **1. Window Functions & Advanced Ranking**
  * Moving Averages, Running Totals, and Year-To-Date (YTD) calculations.
  * `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE()`, and `CUME_DIST()`.
  * Finding the "Top N" per category and filtering with the `QUALIFY` clause.
* **2. Data Aggregation & Grouping**
  * Advanced grouping using `GROUP BY`, `HAVING`, `ROLLUP`, and `GROUPING SETS`.
  * Conditional Aggregation (Pivoting data using `CASE` inside `SUM()`).
  * Creating dynamic bins, histograms, and Pareto (80/20 rule) charts.
* **3. Complex Joins & Set Operations**
  * Self Joins, Full Outer Joins, and Cartesian Products (`CROSS JOIN`).
  * `LATERAL JOIN` / `CROSS APPLY` for row-by-row operations.
  * Combining datasets using `UNION ALL`, `INTERSECT`, and `EXCEPT`.
* **4. Date, Time & Cohort Analysis**
  * Timezone conversions, calculating working days, and extracting date parts.
  * Identifying overlapping date ranges and active subscriptions.
  * "Sessionizing" user clickstream data.
* **5. String Manipulation & JSON**
  * Regex pattern matching, string padding (`LPAD`), and concatenation.
  * Parsing JSON payloads natively in SQL.
  * Unnesting Arrays and flattening comma-separated strings.
* **6. Database Objects & Query Optimization**
  * Designing standard, unique, composite, and full-text Indexes.
  * Creating standard and Materialized Views.
  * Understanding `EXPLAIN` plans to optimize query performance.
* **7. Automation & Procedural SQL**
  * Building Triggers (`BEFORE INSERT`, `AFTER UPDATE`, `AFTER DELETE`).
  * Writing custom User-Defined Functions and Stored Procedures.
  * Handling ACID Transactions (`COMMIT` / `ROLLBACK`).

---

### 1. How do you calculate the moving average in SQL?
**Answer:**  
A moving average is used to smooth out short-term fluctuations and highlight longer-term trends or cycles in data. It can be calculated using the `AVG` function and the `OVER` clause with a `ROWS` or `RANGE` specification.

**Input Data (`sales` table):**
| date | value |
| :--- | :--- |
| 2026-05-01 | 10 |
| 2026-05-02 | 20 |
| 2026-05-03 | 30 |
| 2026-05-04 | 40 |

**SQL Query:**
```sql
-- Example of calculating a 3-day moving average
SELECT date, value,
       AVG(value) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM sales;
```

**Output:**
| date | value | moving_avg |
| :--- | :--- | :--- |
| 2026-05-01 | 10 | 10.00 |
| 2026-05-02 | 20 | 15.00 |
| 2026-05-03 | 30 | 20.00 |
| 2026-05-04 | 40 | 30.00 |

---

### 2. How do you rank data within groups in SQL?
**Answer:**  
You can rank data within groups using window functions such as `RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`. These functions assign a rank to each row within the partition of a result set.

**Input Data (`employees` table):**
| name | department | salary |
| :--- | :--- | :--- |
| Alice | Sales | 60000 |
| Bob | Sales | 80000 |
| Charlie| IT | 75000 |
| David | IT | 75000 |

**SQL Query:**
```sql
-- Example of ranking employees by salary within each department
SELECT name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
```

**Output:**
| name | department | salary | rank |
| :--- | :--- | :--- | :--- |
| Bob | Sales | 80000 | 1 |
| Alice | Sales | 60000 | 2 |
| Charlie| IT | 75000 | 1 |
| David | IT | 75000 | 1 |

---

### 3. How do you handle duplicate rows in SQL?
**Answer:**  
To handle duplicate rows, you can use the `DISTINCT` keyword to select only unique rows. Alternatively, you can use `ROW_NUMBER()` with a common table expression (CTE) to identify and remove duplicates.

**Input Data (`employees` table with duplicates):**
| name | department | salary |
| :--- | :--- | :--- |
| John | HR | 50000 |
| John | HR | 50000 |
| Mary | Finance | 60000 |

**SQL Query:**
```sql
-- Example of removing duplicates using ROW_NUMBER()
WITH CTE AS (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (PARTITION BY name, department ORDER BY salary DESC) AS row_num
    FROM employees
)
DELETE FROM CTE WHERE row_num > 1;

SELECT * FROM employees;
```

**Output:**
| name | department | salary |
| :--- | :--- | :--- |
| John | HR | 50000 |
| Mary | Finance | 60000 |

---

### 4. How do you pivot data in SQL?
**Answer:**  
Pivoting data means converting rows into columns. This can be achieved using the `PIVOT` function in databases that support it or using conditional aggregation with `CASE` statements.

**Input Data (`employees` table):**
| department | gender |
| :--- | :--- |
| IT | Male |
| IT | Female |
| IT | Male |
| HR | Female |

**SQL Query:**
```sql
-- Example of pivoting data using conditional aggregation
SELECT department,
       SUM(CASE WHEN gender = 'Male' THEN 1 ELSE 0 END) AS male_count,
       SUM(CASE WHEN gender = 'Female' THEN 1 ELSE 0 END) AS female_count
FROM employees
GROUP BY department;
```

**Output:**
| department | male_count | female_count |
| :--- | :--- | :--- |
| IT | 2 | 1 |
| HR | 0 | 1 |

---

### 5. How do you unpivot data in SQL?
**Answer:**  
Unpivoting data means converting columns into rows. This can be done using the `UNPIVOT` function in databases that support it or using a `UNION ALL` statement.

**Input Data (`departments` table):**
| department | male_count | female_count |
| :--- | :--- | :--- |
| IT | 2 | 1 |
| HR | 0 | 1 |

**SQL Query:**
```sql
-- Example of unpivoting data using UNION ALL
SELECT department, 'Male' AS gender, male_count AS count
FROM departments
UNION ALL
SELECT department, 'Female' AS gender, female_count AS count
FROM departments;
```

**Output:**
| department | gender | count |
| :--- | :--- | :--- |
| IT | Male | 2 |
| HR | Male | 0 |
| IT | Female | 1 |
| HR | Female | 1 |

---

### 6. How do you use the LEAD and LAG functions in SQL?
**Answer:**  
The `LEAD` and `LAG` functions are used to access data from subsequent or preceding rows in the result set, respectively. These functions are useful for calculating differences or comparisons between rows.

**Input Data (`sales` table):**
| date | value |
| :--- | :--- |
| 2026-05-01 | 100 |
| 2026-05-02 | 150 |
| 2026-05-03 | 120 |

**SQL Query:**
```sql
-- Example of using LEAD and LAG functions
SELECT date, value,
       LAG(value, 1) OVER (ORDER BY date) AS previous_value,
       LEAD(value, 1) OVER (ORDER BY date) AS next_value
FROM sales;
```

**Output:**
| date | value | previous_value | next_value |
| :--- | :--- | :--- | :--- |
| 2026-05-01 | 100 | NULL | 150 |
| 2026-05-02 | 150 | 100 | 120 |
| 2026-05-03 | 120 | 150 | NULL |

---

### 7. How do you handle time series data in SQL?
**Answer:**  
Handling time series data often involves calculating rolling averages, cumulative sums, and differences between time periods. Window functions like `SUM()`, `AVG()`, `ROW_NUMBER()`, and `LAG()` are commonly used.

**Input Data (`sales` table):**
| date | value |
| :--- | :--- |
| 2026-05-01 | 50 |
| 2026-05-02 | 100 |
| 2026-05-03 | 25 |

**SQL Query:**
```sql
-- Example of calculating a cumulative sum
SELECT date, value,
       SUM(value) OVER (ORDER BY date) AS cumulative_sum
FROM sales;
```

**Output:**
| date | value | cumulative_sum |
| :--- | :--- | :--- |
| 2026-05-01 | 50 | 50 |
| 2026-05-02 | 100 | 150 |
| 2026-05-03 | 25 | 175 |

---

### 8. How do you perform a recursive query in SQL?
**Answer:**  
A recursive query is used to retrieve hierarchical data, such as organizational structures. It is implemented using a common table expression (CTE) with the `WITH RECURSIVE` keyword.

**Input Data (`employees` table):**
| id | name | manager_id |
| :--- | :--- | :--- |
| 1 | CEO | NULL |
| 2 | VP | 1 |
| 3 | Analyst | 2 |

**SQL Query:**
```sql
-- Example of a recursive query to find all subordinates
WITH RECURSIVE EmployeeHierarchy AS (
    SELECT id, name, manager_id
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    INNER JOIN EmployeeHierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM EmployeeHierarchy;
```

**Output:**
| id | name | manager_id |
| :--- | :--- | :--- |
| 1 | CEO | NULL |
| 2 | VP | 1 |
| 3 | Analyst | 2 |

---

### 9. How do you optimize a SQL query for performance?
**Answer:**  
Query optimization involves techniques like indexing, using appropriate joins, and avoiding `SELECT *`. 

**Scenario:** Searching for an employee in a massive table is slow.
**Input Data:** A table with 1,000,000 rows. Filtering by `department = 'HR'` requires a full table scan.

**SQL Solution:**
```sql
-- Example of using an index to optimize a query
CREATE INDEX idx_employee_department ON employees(department);

-- Now this query will use the index and run significantly faster
SELECT name, department
FROM employees
WHERE department = 'HR';
```

**Output:**  
*Behind the scenes, the database reads the B-Tree index instead of scanning all 1,000,000 rows, returning the results in milliseconds instead of seconds.*

---

### 10. How do you handle missing data in SQL?
**Answer:**  
Missing data can be handled using functions like `COALESCE` and `NVL` to replace NULL values with default values. 

**Input Data (`employees` table):**
| name | salary |
| :--- | :--- |
| Sam | 50000 |
| Alex | NULL |

**SQL Query:**
```sql
-- Example of handling missing data using COALESCE
SELECT name, COALESCE(salary, 0) AS salary
FROM employees;
```

**Output:**
| name | salary |
| :--- | :--- |
| Sam | 50000 |
| Alex | 0 |

---

### 11. How do you calculate the median in SQL?
**Answer:**  
Calculating the median involves finding the middle value in a sorted list of numbers. This can be done using window functions like the `PERCENTILE_CONT` function in databases that support it.

**Input Data (`employees` table):**
| name | salary |
| :--- | :--- |
| Alice | 40000 |
| Bob | 50000 |
| Charlie| 60000 |
| David | 80000 |
| Eve | 100000 |

**SQL Query:**
```sql
-- Example of calculating the median using PERCENTILE_CONT
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees;
```

**Output:**
| median_salary |
| :--- |
| 60000 |

---

### 12. How do you perform a self-join in SQL?
**Answer:**  
A self-join is a regular join in which a table is joined with itself. It is extremely useful for querying hierarchical data, such as finding the manager for each employee within the same table.

**Input Data (`employees` table):**
| id | name | manager_id |
| :--- | :--- | :--- |
| 1 | Sarah (CEO) | NULL |
| 2 | John (Manager)| 1 |
| 3 | Mike (Analyst)| 2 |

**SQL Query:**
```sql
-- Example of a self-join to find an employee's manager
SELECT e1.name AS employee_name, e2.name AS manager_name
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;
```

**Output:**
| employee_name | manager_name |
| :--- | :--- |
| John (Manager)| Sarah (CEO) |
| Mike (Analyst)| John (Manager)|

---

### 13. How do you create a stored procedure in SQL?
**Answer:**  
A stored procedure is a precompiled collection of one or more SQL statements stored in the database. It allows you to reuse complex query logic simply by "calling" the procedure.

**Input Data (`employees` table):**
| id | name | department |
| :--- | :--- | :--- |
| 101 | Anna | Sales |
| 102 | Mark | HR |

**SQL Query:**
```sql
-- 1. Create the stored procedure
CREATE PROCEDURE GetEmployeeDetails
AS
BEGIN
    SELECT * FROM employees;
END;

-- 2. Execute the procedure
EXEC GetEmployeeDetails;
```

**Output:**
| id | name | department |
| :--- | :--- | :--- |
| 101 | Anna | Sales |
| 102 | Mark | HR |

---

### 14. How do you create a trigger in SQL?
**Answer:**  
A trigger is a special type of stored procedure that automatically executes (or "fires") in response to specific events on a particular table, such as an `INSERT`, `UPDATE`, or `DELETE`.

**Input Data:**
A new employee record is being inserted into the database.

**SQL Query:**
```sql
-- Example of creating a trigger
CREATE TRIGGER trgAfterInsert ON employees
FOR INSERT
AS
BEGIN
    PRINT 'New employee record inserted successfully.';
END;

-- Inserting a record to fire the trigger
INSERT INTO employees (id, name, department) VALUES (103, 'Luke', 'IT');
```

**Output:**
```text
New employee record inserted successfully.
(1 row affected)
```

---

### 15. How do you create a materialized view in SQL?
**Answer:**  
Unlike a standard view, a materialized view physically stores the result of a query. It is used to drastically improve query performance by precomputing complex joins or aggregations, though it needs to be refreshed to show new data.

**Input Data:**
`employees` table and `departments` table.

**SQL Query:**
```sql
-- Example of creating a materialized view
CREATE MATERIALIZED VIEW emp_dept_view AS
SELECT e.name, d.name AS department_name
FROM employees e
JOIN departments d ON e.department_id = d.id;

-- Querying the newly created physical view
SELECT * FROM emp_dept_view;
```

**Output:**
| name | department_name |
| :--- | :--- |
| Anna | Sales |
| Mark | Human Resources |

---

### 16. How do you refresh a materialized view in SQL?
**Answer:**  
Because a materialized view stores a physical snapshot of the data, it must be refreshed to reflect any recent changes made to the underlying base tables.

**Input Data (`employees` table):**
Anna is transferred from Sales to Marketing in the base tables, but the `emp_dept_view` still shows her in Sales.

**SQL Query:**
```sql
-- Example of refreshing a materialized view (PostgreSQL syntax)
REFRESH MATERIALIZED VIEW emp_dept_view;

-- Querying the view after refresh
SELECT * FROM emp_dept_view;
```

**Output:**
| name | department_name |
| :--- | :--- |
| Anna | Marketing |
| Mark | Human Resources |

---

### 17. How do you use the OVER() clause in SQL?
**Answer:**  
The `OVER()` clause dictates exactly how window functions should partition and order rows. It allows you to perform calculations across a specific set of table rows related to the current row, without collapsing the output like `GROUP BY` does.

**Input Data (`employees` table):**
| name | department | salary |
| :--- | :--- | :--- |
| Paul | IT | 60000 |
| Emma | IT | 70000 |
| Liam | HR | 55000 |

**SQL Query:**
```sql
-- Example of using OVER() to sequence rows within departments
SELECT name, department, salary,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num
FROM employees;
```

**Output:**
| name | department | salary | row_num |
| :--- | :--- | :--- | :--- |
| Emma | IT | 70000 | 1 |
| Paul | IT | 60000 | 2 |
| Liam | HR | 55000 | 1 |

---

### 18. How do you calculate a cumulative sum in SQL?
**Answer:**  
A cumulative sum calculates a running total by adding each value in a column to the sum of all preceding values. 

**Input Data (`daily_sales` table):**
| date | revenue |
| :--- | :--- |
| 2026-06-01 | 200 |
| 2026-06-02 | 300 |
| 2026-06-03 | 150 |

**SQL Query:**
```sql
-- Example of calculating a cumulative sum
SELECT date, revenue,
       SUM(revenue) OVER (ORDER BY date) AS cumulative_revenue
FROM daily_sales;
```

**Output:**
| date | revenue | cumulative_revenue |
| :--- | :--- | :--- |
| 2026-06-01 | 200 | 200 |
| 2026-06-02 | 300 | 500 |
| 2026-06-03 | 150 | 650 |

---

### 19. How do you calculate a running total partitioned by a category?
**Answer:**  
A partitioned running total is similar to a cumulative sum, but the total resets whenever the specified category changes. This is achieved using `PARTITION BY` alongside the `OVER()` clause.

**Input Data (`sales` table):**
| date | category | value |
| :--- | :--- | :--- |
| 2026-06-01 | Hardware | 100 |
| 2026-06-02 | Hardware | 200 |
| 2026-06-01 | Software | 50 |
| 2026-06-02 | Software | 50 |

**SQL Query:**
```sql
-- Example of calculating a running total partitioned by category
SELECT date, category, value,
       SUM(value) OVER (PARTITION BY category ORDER BY date) AS running_total
FROM sales;
```

**Output:**
| date | category | value | running_total |
| :--- | :--- | :--- | :--- |
| 2026-06-01 | Hardware | 100 | 100 |
| 2026-06-02 | Hardware | 200 | 300 |
| 2026-06-01 | Software | 50 | 50 |
| 2026-06-02 | Software | 50 | 100 |

---

### 20. How do you perform conditional aggregation in SQL?
**Answer:**  
Conditional aggregation uses aggregate functions (like `SUM` or `COUNT`) combined with `CASE` statements to perform logic on specific segments of data in a single pass.

**Input Data (`employees` table):**
| department | gender | salary |
| :--- | :--- | :--- |
| IT | Male | 80000 |
| IT | Female | 90000 |
| HR | Female | 60000 |
| HR | Male | 55000 |

**SQL Query:**
```sql
-- Example of summing salary based on a condition
SELECT department,
       SUM(CASE WHEN gender = 'Male' THEN salary ELSE 0 END) AS male_salary,
       SUM(CASE WHEN gender = 'Female' THEN salary ELSE 0 END) AS female_salary
FROM employees
GROUP BY department;
```

**Output:**
| department | male_salary | female_salary |
| :--- | :--- | :--- |
| IT | 80000 | 90000 |
| HR | 55000 | 60000 |

---
### 21. How do you perform a full outer join in SQL?
**Answer:**  
A full outer join returns all rows when there is a match in either the left or the right table. If there is no match on one side, it returns `NULL` for the columns from that side.

**Input Data (`employees` and `departments` tables):**
*Employees:*
| name | department_id |
| :--- | :--- |
| John | 1 |
| Jane | NULL |

*Departments:*
| id | name |
| :--- | :--- |
| 1 | HR |
| 2 | IT |

**SQL Query:**
```sql
-- Example of full outer join
SELECT e.name, d.name AS department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

**Output:**
| name | department_name |
| :--- | :--- |
| John | HR |
| Jane | NULL |
| NULL | IT |

---

### 22. How do you handle NULL values in conditional logic in SQL?
**Answer:**  
When working with conditional logic like `CASE` statements or `WHERE` clauses, it is crucial to remember that `NULL` is not equal to anything, not even itself. You must use operators like `IS NULL` or functions like `COALESCE` to handle them properly.

**Input Data (`employees` table):**
| name | commission |
| :--- | :--- |
| Bob | 500 |
| Sam | NULL |

**SQL Query:**
```sql
-- Example of handling NULL values in conditional logic
SELECT name,
       CASE 
           WHEN commission IS NULL THEN 'No Commission'
           ELSE 'Receives Commission'
       END AS commission_status
FROM employees;
```

**Output:**
| name | commission_status |
| :--- | :--- |
| Bob | Receives Commission |
| Sam | No Commission |

---

### 23. How do you remove duplicates based on specific columns in SQL?
**Answer:**  
To remove duplicates based on *specific* columns rather than the entire row, you can use the `ROW_NUMBER()` window function to partition the data by those specific columns, order them to keep the "best" record, and filter out the rest.

**Input Data (`customer_logins` table):**
| customer_id | login_time | ip_address |
| :--- | :--- | :--- |
| 101 | 08:00 AM | 192.168.1.1 |
| 101 | 09:00 AM | 192.168.1.5 |
| 102 | 10:00 AM | 192.168.1.1 |

**SQL Query:**
```sql
-- Example of keeping only the most recent login per customer
WITH RankedLogins AS (
    SELECT customer_id, login_time, ip_address,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY login_time DESC) as rn
    FROM customer_logins
)
SELECT customer_id, login_time, ip_address
FROM RankedLogins
WHERE rn = 1;
```

**Output:**
| customer_id | login_time | ip_address |
| :--- | :--- | :--- |
| 101 | 09:00 AM | 192.168.1.5 |
| 102 | 10:00 AM | 192.168.1.1 |

---

### 24. How do you create a temporary table in SQL?
**Answer:**  
A temporary table is a table that exists only for the duration of a database session or transaction. They are extremely useful for storing intermediate results during complex data transformations.

**Input Data (`employees` table):**
| id | name | department |
| :--- | :--- | :--- |
| 1 | Leo | Sales |
| 2 | Mia | IT |

**SQL Query:**
```sql
-- Example of creating a temporary table
CREATE TEMPORARY TABLE temp_sales_employees AS
SELECT * FROM employees WHERE department = 'Sales';

-- Querying the temporary table
SELECT * FROM temp_sales_employees;
```

**Output:**
| id | name | department |
| :--- | :--- | :--- |
| 1 | Leo | Sales |

---

### 25. How do you use the GROUP BY clause in SQL?
**Answer:**  
The `GROUP BY` clause groups rows that have identical values in specified columns into summary rows. It is essential when using aggregate functions (like `COUNT`, `MAX`, `MIN`, `SUM`, `AVG`) to perform calculations on each individual group.

**Input Data (`sales` table):**
| region | revenue |
| :--- | :--- |
| North | 100 |
| North | 150 |
| South | 200 |

**SQL Query:**
```sql
-- Example of using GROUP BY to find total revenue per region
SELECT region, SUM(revenue) AS total_revenue
FROM sales
GROUP BY region;
```

**Output:**
| region | total_revenue |
| :--- | :--- |
| North | 250 |
| South | 200 |

---

### 26. What is the difference between WHERE and HAVING clauses in SQL?
**Answer:**  
The `WHERE` clause filters individual rows *before* they are grouped. The `HAVING` clause filters summary groups *after* the `GROUP BY` operation has been performed.

**Input Data (`sales` table):**
| region | revenue |
| :--- | :--- |
| North | 100 |
| North | 150 |
| South | 50 |

**SQL Query:**
```sql
-- Example: We only want regions whose TOTAL revenue is greater than 100
SELECT region, SUM(revenue) AS total_revenue
FROM sales
GROUP BY region
HAVING SUM(revenue) > 100;
```

**Output:**
| region | total_revenue |
| :--- | :--- |
| North | 250 |

---

### 27. How do you find the second highest value in a column?
**Answer:**  
Finding the second highest value (like salary) is a classic interview question. It can be solved using window functions like `DENSE_RANK()`, or by using a subquery combined with the `MAX()` function.

**Input Data (`employees` table):**
| name | salary |
| :--- | :--- |
| Tom | 80000 |
| Dan | 90000 |
| Sue | 90000 |
| Ben | 70000 |

**SQL Query:**
```sql
-- Example using DENSE_RANK() to handle ties correctly
WITH RankedSalaries AS (
    SELECT name, salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
)
SELECT name, salary
FROM RankedSalaries
WHERE rank = 2;
```

**Output:**
| name | salary |
| :--- | :--- |
| Tom | 80000 |

---

### 28. How do you use the CASE statement in SQL?
**Answer:**  
The `CASE` statement creates conditional, "if-then-else" logic directly within your SQL queries. It allows you to transform or categorize data on the fly.

**Input Data (`employees` table):**
| name | salary |
| :--- | :--- |
| Ann | 60000 |
| Bob | 40000 |
| Cam | 25000 |

**SQL Query:**
```sql
-- Example of categorizing salary levels using CASE
SELECT name, salary,
       CASE
           WHEN salary > 50000 THEN 'High'
           WHEN salary BETWEEN 30000 AND 50000 THEN 'Medium'
           ELSE 'Low'
       END AS salary_level
FROM employees;
```

**Output:**
| name | salary | salary_level |
| :--- | :--- | :--- |
| Ann | 60000 | High |
| Bob | 40000 | Medium |
| Cam | 25000 | Low |

---

### 29. How do you calculate the exact difference between two dates in SQL?
**Answer:**  
Calculating date differences varies slightly by SQL dialect, but most modern databases use a variation of the `DATEDIFF` function or allow direct subtraction of date data types.

**Input Data:** 
Two dates: `2026-06-01` and `2026-06-15`.

**SQL Query:**
```sql
-- Example of calculating date difference in days (SQL Server syntax)
SELECT DATEDIFF(day, '2026-06-01', '2026-06-15') AS days_diff;

-- Example of calculating date difference in days (PostgreSQL syntax)
-- SELECT '2026-06-15'::date - '2026-06-01'::date AS days_diff;
```

**Output:**
| days_diff |
| :--- |
| 14 |

---

### 30. How do you use the UNION operator in SQL?
**Answer:**  
The `UNION` operator combines the result sets of two or more `SELECT` queries into a single column. Crucially, `UNION` automatically removes duplicate rows between the result sets. (To keep duplicates, you must use `UNION ALL`).

**Input Data (`employees` and `contractors` tables):**
*Employees:*
| name |
| :--- |
| John |
| Mary |

*Contractors:*
| name |
| :--- |
| John |
| Pete |

**SQL Query:**
```sql
-- Example of combining lists using UNION
SELECT name FROM employees
UNION
SELECT name FROM contractors;
```

**Output:**
| name |
| :--- |
| John |
| Mary |
| Pete |

---

### 31. How do you use the UNION ALL operator in SQL?
**Answer:**  
The `UNION ALL` operator is used to combine the result sets of two or more `SELECT` queries. Unlike `UNION`, `UNION ALL` does *not* remove duplicate rows, making it faster to execute when you know your data doesn't have duplicates or when you explicitly want to see every record.

**Input Data (`employees` and `contractors` tables):**
*Employees:*
| name |
| :--- |
| John |
| Mary |

*Contractors:*
| name |
| :--- |
| John |
| Pete |

**SQL Query:**
```sql
-- Example of combining lists using UNION ALL
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;
```

**Output:**
| name |
| :--- |
| John |
| Mary |
| John |
| Pete |

---

### 32. How do you use the INTERSECT operator in SQL?
**Answer:**  
The `INTERSECT` operator is used to return only the rows that appear in *both* result sets of two or more `SELECT` queries. 

**Input Data (`employees` and `managers` tables):**
*Employees:*
| name |
| :--- |
| Alice |
| Bob |
| Charlie |

*Managers:*
| name |
| :--- |
| Alice |
| Diana |

**SQL Query:**
```sql
-- Example of INTERSECT to find employees who are also managers
SELECT name FROM employees
INTERSECT
SELECT name FROM managers;
```

**Output:**
| name |
| :--- |
| Alice |

---

### 33. How do you use the EXCEPT operator in SQL?
**Answer:**  
The `EXCEPT` operator (known as `MINUS` in Oracle) returns the rows from the first `SELECT` query that are *not* present in the second `SELECT` query. It effectively subtracts the second dataset from the first.

**Input Data (`all_staff` and `managers` tables):**
*All Staff:*
| name |
| :--- |
| Alice |
| Bob |
| Charlie |

*Managers:*
| name |
| :--- |
| Alice |

**SQL Query:**
```sql
-- Example of EXCEPT to find staff who are NOT managers
SELECT name FROM all_staff
EXCEPT
SELECT name FROM managers;
```

**Output:**
| name |
| :--- |
| Bob |
| Charlie |

---

### 34. How do you use the EXISTS operator in SQL?
**Answer:**  
The `EXISTS` operator tests for the existence of any rows in a subquery. It returns `TRUE` the moment it finds at least one matching row, making it highly efficient for checking conditions without returning massive amounts of data.

**Input Data (`departments` and `employees` tables):**
*Departments:*
| id | name |
| :--- | :--- |
| 1 | HR |
| 2 | IT |
| 3 | Legal |

*Employees:*
| name | dept_id |
| :--- | :--- |
| John | 1 |
| Jane | 2 |

**SQL Query:**
```sql
-- Example of finding departments that actually have employees
SELECT name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.dept_id = d.id
);
```

**Output:**
| name |
| :--- |
| HR |
| IT |

---

### 35. How do you use the IN operator in SQL?
**Answer:**  
The `IN` operator is a shorthand for multiple `OR` conditions. It allows you to specify a list of specific values you want to filter for within a `WHERE` clause.

**Input Data (`employees` table):**
| name | department |
| :--- | :--- |
| Alice | HR |
| Bob | IT |
| Charlie | Sales |
| Dave | Marketing |

**SQL Query:**
```sql
-- Example of IN
SELECT name, department
FROM employees
WHERE department IN ('HR', 'Sales', 'IT');
```

**Output:**
| name | department |
| :--- | :--- |
| Alice | HR |
| Bob | IT |
| Charlie | Sales |

---

### 36. How do you use the BETWEEN operator in SQL?
**Answer:**  
The `BETWEEN` operator filters rows based on a specified continuous range of values (numbers, text, or dates). It is inclusive, meaning it includes both the start and end values.

**Input Data (`employees` table):**
| name | salary |
| :--- | :--- |
| Ann | 25000 |
| Bob | 40000 |
| Cam | 45000 |
| Dan | 60000 |

**SQL Query:**
```sql
-- Example of BETWEEN
SELECT name, salary
FROM employees
WHERE salary BETWEEN 30000 AND 50000;
```

**Output:**
| name | salary |
| :--- | :--- |
| Bob | 40000 |
| Cam | 45000 |

---

### 37. How do you use the LIKE operator in SQL?
**Answer:**  
The `LIKE` operator is used to search for a specified pattern within a text column. It is almost always used with wildcards: `%` (represents zero or more characters) and `_` (represents a single character).

**Input Data (`employees` table):**
| name |
| :--- |
| John |
| Jane |
| Jack |
| Bob |

**SQL Query:**
```sql
-- Example of finding names that start with 'J'
SELECT name
FROM employees
WHERE name LIKE 'J%';
```

**Output:**
| name |
| :--- |
| John |
| Jane |
| Jack |

---

### 38. How do you use the CONCAT function in SQL?
**Answer:**  
The `CONCAT` function takes two or more strings and merges them into a single string. This is heavily used for formatting names, addresses, or generating clean reports.

**Input Data (`employees` table):**
| first_name | last_name |
| :--- | :--- |
| John | Doe |
| Jane | Smith |

**SQL Query:**
```sql
-- Example of combining first and last names with a space in between
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;
```

**Output:**
| full_name |
| :--- |
| John Doe |
| Jane Smith |

---

### 39. How do you create an index in SQL?
**Answer:**  
An index acts like the table of contents in a book. It is created on a column (or columns) to drastically speed up data retrieval operations (`SELECT` queries), though it slightly slows down data modification (`INSERT`, `UPDATE`, `DELETE`) because the index must be maintained.

**Scenario:** Searching for an employee's name in a massive company database.

**SQL Query:**
```sql
-- Example of creating a standard index
CREATE INDEX idx_employee_name ON employees(name);

-- Subsequent searches will now use the index for faster lookups
SELECT * FROM employees WHERE name = 'John Doe';
```

**Output:**  
*Behind the scenes, the database reads the index pointer directly to the row's physical location instead of scanning the entire table top-to-bottom.*

---

### 40. How do you create a unique index in SQL?
**Answer:**  
A unique index serves two purposes: it speeds up data retrieval just like a regular index, but it also enforces a strict rule that duplicate values cannot be entered into that column. It is excellent for protecting data integrity.

**Scenario:** Ensuring no two employees can be registered with the exact same email address.

**SQL Query:**
```sql
-- Example of creating a unique index
CREATE UNIQUE INDEX idx_employee_email ON employees(email);

-- Attempting to insert a duplicate email will now throw a database error
```

**Output:**
*Any subsequent `INSERT` or `UPDATE` statement that tries to put an existing email address into the `email` column will be rejected by the database.*

---

### 41. How do you create a composite index in SQL?
**Answer:**  
A composite index is an index placed on two or more columns of a table. It is highly effective for optimizing queries that frequently filter, group, or sort data based on those specific multiple columns together.

**Scenario:** An HR system frequently searches for employees by both their name and their department simultaneously.

**SQL Query:**
```sql
-- Example of creating a composite index
CREATE INDEX idx_name_department ON employees(name, department);

-- This query will now use the composite index for lightning-fast retrieval
SELECT * FROM employees WHERE name = 'Alice' AND department = 'IT';
```

**Output:**  
*Behind the scenes, the database utilizes the multi-column index to instantly locate records matching both criteria, avoiding a full table scan.*

---

### 42. How do you create a full-text index in SQL?
**Answer:**  
A full-text index is a specialized index used to significantly speed up complex text searches (like looking for specific words or phrases) within large text fields, such as articles, descriptions, or resumes.

**Scenario:** A recruitment platform needs to search thousands of long-form text resumes for specific programming languages.

**SQL Query:**
```sql
-- Example of creating a full-text index (MySQL syntax)
CREATE FULLTEXT INDEX idx_employee_resume ON employees(resume);

-- Searching using the full-text index
SELECT name FROM employees WHERE MATCH(resume) AGAINST('Python SQL');
```

**Output:**  
*The database returns rows containing 'Python' or 'SQL' much faster than using a standard `LIKE '%Python%'` wildcard search, which forces a full table scan.*

---

### 43. How do you drop an index in SQL?
**Answer:**  
An index can be dropped (deleted) using the `DROP INDEX` statement. While indexes speed up read operations (`SELECT`), they slow down write operations (`INSERT`, `UPDATE`, `DELETE`). Dropping unused indexes reclaims storage space and improves write performance.

**Input Data:**
An existing index named `idx_employee_name` that is no longer being used by query plans.

**SQL Query:**
```sql
-- Example of dropping an index (Syntax varies slightly by database)
DROP INDEX idx_employee_name ON employees;
```

**Output:**  
*The index structure is removed from the database storage. Subsequent searches on the `name` column will revert to scanning the entire table.*

---

### 44. How do you use the TRANSLATE function in SQL?
**Answer:**  
The `TRANSLATE` function replaces a sequence of characters in a string with another set of corresponding characters. It performs a one-to-one character substitution, which is great for data cleaning or simple obfuscation.

**Input Data (`employees` table):**
| name |
| :--- |
| Jane Doe |
| Sam Smith |

**SQL Query:**
```sql
-- Example: Replacing all vowels with numbers
SELECT name, 
       TRANSLATE(name, 'aeiou', '12345') AS translated_name
FROM employees;
```

**Output:**
| name | translated_name |
| :--- | :--- |
| Jane Doe | J1n2 D42 |
| Sam Smith | S1m Sm3th |

---

### 45. How do you use the SUBSTRING function in SQL?
**Answer:**  
The `SUBSTRING` function extracts a specific portion of a string based on a starting position and a specified length. It is incredibly useful for parsing standardized data formats.

**Input Data (`employees` table):**
| phone_number |
| :--- |
| 555-123-4567 |
| 415-987-6543 |

**SQL Query:**
```sql
-- Example of extracting just the 3-digit area code (starts at position 1, length 3)
SELECT phone_number, 
       SUBSTRING(phone_number, 1, 3) AS area_code
FROM employees;
```

**Output:**
| phone_number | area_code |
| :--- | :--- |
| 555-123-4567 | 555 |
| 415-987-6543 | 415 |

---

### 46. How do you use the REPLACE function in SQL?
**Answer:**  
Unlike `TRANSLATE` (which replaces individual characters), the `REPLACE` function replaces all occurrences of an entire specified substring within a string with a new substring.

**Input Data (`departments` table):**
| dept_name |
| :--- |
| Human Resources - North |
| Human Resources - South |

**SQL Query:**
```sql
-- Example of replacing a long phrase with an abbreviation
SELECT dept_name, 
       REPLACE(dept_name, 'Human Resources', 'HR') AS clean_dept
FROM departments;
```

**Output:**
| dept_name | clean_dept |
| :--- | :--- |
| Human Resources - North | HR - North |
| Human Resources - South | HR - South |

---

### 47. How do you create a view in SQL?
**Answer:**  
A view is a "virtual table" generated by a saved SQL query. It does not store data itself; instead, it dynamically pulls data from underlying base tables whenever queried. Views simplify complex logic and can restrict user access to specific columns.

**Input Data:** 
A complex database with separate `employees` and `departments` tables.

**SQL Query:**
```sql
-- Example of creating a view that hides complex JOIN logic
CREATE VIEW employee_directory AS
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- Now users can simply query the view like a normal table
SELECT * FROM employee_directory;
```

**Output:**
| first_name | last_name | department_name |
| :--- | :--- | :--- |
| John | Doe | IT |
| Mary | Jane | HR |

---

### 48. How do you update data in a view in SQL?
**Answer:**  
You can update data directly through a view *only if* the view is "updatable." Generally, this means the view maps directly to a single base table without any aggregate functions (`SUM`, `AVG`), `GROUP BY` clauses, or complex joins.

**Input Data:**
An updatable view named `active_employees` pulling directly from the `employees` table.

**SQL Query:**
```sql
-- Example of updating a base table through a view
UPDATE active_employees
SET status = 'On Leave'
WHERE name = 'John Doe';
```

**Output:**  
*The base `employees` table is updated. John Doe's status is permanently changed to 'On Leave' in the underlying database architecture.*

---

### 49. How do you delete data from a view in SQL?
**Answer:**  
Similar to updating, you can delete records through a view if it is an updatable, single-table view. Deleting from the view permanently removes the corresponding row from the underlying base table.

**Input Data:**
An updatable view named `contractors_view`.

**SQL Query:**
```sql
-- Example of deleting a record through a view
DELETE FROM contractors_view
WHERE name = 'John Doe';
```

**Output:**  
*The row belonging to John Doe is permanently deleted from the underlying base table that powers `contractors_view`.*

---

### 50. How do you create a sequence in SQL?
**Answer:**  
A sequence is a database object that automatically generates an incrementing list of unique numeric values. Sequences are most commonly used to automatically generate primary key IDs when inserting new records.

**Input Data:**
An empty `employees` table that needs unique IDs for new hires.

**SQL Query:**
```sql
-- Example of creating a sequence (Oracle / PostgreSQL syntax)
CREATE SEQUENCE emp_sequence
START WITH 100
INCREMENT BY 1;

-- Using the sequence to automatically assign a new ID during an INSERT
INSERT INTO employees (id, name, department)
VALUES (NEXTVAL('emp_sequence'), 'John Doe', 'HR');

INSERT INTO employees (id, name, department)
VALUES (NEXTVAL('emp_sequence'), 'Jane Smith', 'IT');
```

**Output:**
| id | name | department |
| :--- | :--- | :--- |
| 100 | John Doe | HR |
| 101 | Jane Smith | IT |

---
### 51. How do you create a trigger that fires before an insert operation?
**Answer:**  
A `BEFORE INSERT` trigger automatically executes logic just before a new row is permanently saved to a table. This is incredibly useful for validating data or enforcing complex business rules that standard table constraints cannot handle.

**Input Data (`employees` table):**
An HR rep accidentally tries to insert a new employee with a negative salary.

**SQL Query:**
```sql
-- Example of a BEFORE INSERT trigger (MySQL syntax)
CREATE TRIGGER before_insert_employee
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < 0 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Error: Salary cannot be negative';
    END IF;
END;

-- Attempting to insert invalid data
INSERT INTO employees (name, salary) VALUES ('Dave', -5000);
```

**Output:**
```text
Error: Salary cannot be negative. The INSERT operation is aborted.
```

---

### 52. How do you create a trigger that fires after an update operation?
**Answer:**  
An `AFTER UPDATE` trigger runs immediately after a row has been modified. This is the standard method for building automated audit trails to track historical changes over time.

**Input Data:** 
An `employees` table where Alice's salary changes from 60000 to 75000, and an empty `audit_log` table.

**SQL Query:**
```sql
-- Example of an AFTER UPDATE trigger to track salary changes
CREATE TRIGGER after_update_employee
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Only log if the salary actually changed
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO audit_log (employee_id, old_salary, new_salary, change_date)
        VALUES (OLD.id, OLD.salary, NEW.salary, NOW());
    END IF;
END;
```

**Output (Inside `audit_log` table after Alice's update):**
| employee_id | old_salary | new_salary | change_date |
| :--- | :--- | :--- | :--- |
| 101 | 60000 | 75000 | 2026-06-01 10:00:00 |

---

### 53. How do you create a trigger that fires after a delete operation?
**Answer:**  
An `AFTER DELETE` trigger fires right after a row is removed. It is commonly used for "soft deletes" or archiving records into a backup table before they are permanently lost.

**Input Data:**
Bob leaves the company and is deleted from the `employees` table.

**SQL Query:**
```sql
-- Example of an AFTER DELETE trigger for archiving
CREATE TRIGGER after_delete_employee
AFTER DELETE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO deleted_employees_archive (employee_id, name, department, deleted_date)
    VALUES (OLD.id, OLD.name, OLD.department, NOW());
END;
```

**Output (Inside `deleted_employees_archive` table):**
| employee_id | name | department | deleted_date |
| :--- | :--- | :--- | :--- |
| 102 | Bob | IT | 2026-06-05 14:30:00 |

---

### 54. How do you create a custom function in SQL?
**Answer:**  
A custom function (or User-Defined Function) takes input parameters, performs an operation or calculation, and returns a single value. Functions can be used seamlessly inside `SELECT` or `WHERE` clauses just like built-in SQL functions.

**Input Data (`sales_reps` table):**
| name | total_sales |
| :--- | :--- |
| Jane | 100000 |
| Mark | 50000 |

**SQL Query:**
```sql
-- 1. Creating the function to calculate a 10% bonus
CREATE FUNCTION calculate_bonus (sales DECIMAL)
RETURNS DECIMAL
BEGIN
    RETURN sales * 0.10;
END;

-- 2. Using the function in a query
SELECT name, total_sales, calculate_bonus(total_sales) AS bonus
FROM sales_reps;
```

**Output:**
| name | total_sales | bonus |
| :--- | :--- | :--- |
| Jane | 100000 | 10000.00 |
| Mark | 50000 | 5000.00 |

---

### 55. How do you use IN and OUT parameters in a Stored Procedure?
**Answer:**  
While functions must return a single value, stored procedures can return multiple values using `OUT` parameters, or simply perform actions without returning anything.

**Input Data (`employees` table):**
An employee database where we want to quickly fetch an employee's name and salary using only their ID.

**SQL Query:**
```sql
-- Creating a procedure with an IN parameter (ID) and OUT parameters
CREATE PROCEDURE GetEmployeeStats (
    IN emp_id INT, 
    OUT emp_name VARCHAR(50), 
    OUT emp_salary DECIMAL
)
BEGIN
    SELECT name, salary INTO emp_name, emp_salary 
    FROM employees 
    WHERE id = emp_id;
END;
```

**Output:**
*When executed, this procedure will populate the `emp_name` and `emp_salary` variables in your database session or application backend, allowing you to use those specific values elsewhere.*

---

### 56. How do you use a Cursor in SQL?
**Answer:**  
SQL is designed for set-based operations (acting on all rows at once). A cursor breaks this rule by allowing you to iterate through a result set row-by-row. Cursors are generally slower and should only be used when row-by-row processing is strictly necessary.

**Scenario:** Applying complex, custom logic to each employee's record one at a time.

**SQL Query:**
```sql
-- Example of basic cursor logic (PL/SQL syntax)
DECLARE
    CURSOR emp_cursor IS SELECT name, salary FROM employees;
    emp_record emp_cursor%ROWTYPE;
BEGIN
    OPEN emp_cursor;
    LOOP
        FETCH emp_cursor INTO emp_record;
        EXIT WHEN emp_cursor%NOTFOUND;
        -- Process each row individually
        DBMS_OUTPUT.PUT_LINE(emp_record.name || ' makes ' || emp_record.salary);
    END LOOP;
    CLOSE emp_cursor;
END;
```

**Output:**
```text
Alice makes 80000
Bob makes 60000
Charlie makes 75000
```

---

### 57. How do you handle Transactions in SQL?
**Answer:**  
Transactions ensure data integrity using ACID properties. A transaction groups multiple SQL statements into a single unit of work. If all statements succeed, you `COMMIT` the changes. If any statement fails, you `ROLLBACK` the transaction, undoing everything so the database remains in a consistent state.

**Input Data (`bank_accounts` table):**
Transferring $500 from Account A to Account B.

**SQL Query:**
```sql
BEGIN TRANSACTION;

-- Deduct from Account A
UPDATE bank_accounts SET balance = balance - 500 WHERE account_id = 'A';

-- Add to Account B
UPDATE bank_accounts SET balance = balance + 500 WHERE account_id = 'B';

-- If both updates succeed without error:
COMMIT;

-- If an error occurs (e.g., Account A has insufficient funds):
-- ROLLBACK;
```

**Output:**
*Both balances are updated simultaneously. If the database crashes after the first update but before the second, the `ROLLBACK` ensures Account A doesn't lose $500 into thin air.*

---

### 58. How do you concatenate strings from multiple rows into one field?
**Answer:**  
Data analysts often need to collapse multiple rows into a single, comma-separated list. This is done using string aggregation functions like `STRING_AGG()` in PostgreSQL/SQL Server or `GROUP_CONCAT()` in MySQL.

**Input Data (`employees` table):**
| department | name |
| :--- | :--- |
| IT | Alice |
| IT | Bob |
| HR | Charlie |

**SQL Query:**
```sql
-- Example using STRING_AGG (PostgreSQL / SQL Server)
SELECT department, 
       STRING_AGG(name, ', ') AS employee_list
FROM employees
GROUP BY department;

-- Note: Use GROUP_CONCAT(name SEPARATOR ', ') in MySQL
```

**Output:**
| department | employee_list |
| :--- | :--- |
| IT | Alice, Bob |
| HR | Charlie |

---

### 59. How do you extract specific parts of a date in SQL?
**Answer:**  
Often, you only need the year, month, or day from a full timestamp to group data for reporting (e.g., finding total sales by year). This is achieved using the `EXTRACT()` function or `DATEPART()`.

**Input Data (`orders` table):**
| order_id | order_date | total |
| :--- | :--- | :--- |
| 1 | 2026-05-14 10:30:00 | 150 |
| 2 | 2026-08-21 14:45:00 | 200 |

**SQL Query:**
```sql
-- Example of extracting the Year and Month
SELECT order_id,
       EXTRACT(YEAR FROM order_date) AS order_year,
       EXTRACT(MONTH FROM order_date) AS order_month
FROM orders;
```

**Output:**
| order_id | order_year | order_month |
| :--- | :--- | :--- |
| 1 | 2026 | 5 |
| 2 | 2026 | 8 |

---

### 60. How do you convert data types in SQL?
**Answer:**  
Data type mismatch is a common issue for analysts. You can use the `CAST()` or `CONVERT()` functions to change a value from one data type to another (e.g., turning a string into a date, or a decimal into an integer).

**Input Data (`raw_data` table):**
| id | string_date | string_price |
| :--- | :--- | :--- |
| 1 | '2026-01-15' | '199.99' |

**SQL Query:**
```sql
-- Example of casting strings to usable dates and numbers
SELECT id,
       CAST(string_date AS DATE) AS formatted_date,
       CAST(string_price AS DECIMAL(10,2)) AS formatted_price
FROM raw_data;
```

**Output:**
| id | formatted_date | formatted_price |
| :--- | :--- | :--- |
| 1 | 2026-01-15 | 199.99 |

---
### 61. How do you format dates or extract the day of the week in SQL?
**Answer:**  
Data analysts frequently need to convert raw timestamps into readable formats (like 'YYYY-MM') or find the specific day of the week for grouping. This is typically done using `TO_CHAR()` or `DATE_FORMAT()`, depending on your SQL dialect.

**Input Data (`orders` table):**
| order_id | order_date |
| :--- | :--- |
| 1 | 2026-05-14 10:30:00 |
| 2 | 2026-05-15 14:45:00 |

**SQL Query:**
```sql
-- Example using PostgreSQL / Snowflake syntax
SELECT order_id,
       TO_CHAR(order_date, 'YYYY-MM-DD') AS formatted_date,
       TO_CHAR(order_date, 'Day') AS day_of_week
FROM orders;
```

**Output:**
| order_id | formatted_date | day_of_week |
| :--- | :--- | :--- |
| 1 | 2026-05-14 | Thursday |
| 2 | 2026-05-15 | Friday |

---

### 62. What is the exact difference between ROW_NUMBER(), RANK(), and DENSE_RANK()?
**Answer:**  
All three are window functions used to rank data, but they handle *ties* (identical values) differently:
*   `ROW_NUMBER()`: Gives a unique, sequential number to every row, ignoring ties.
*   `RANK()`: Gives the same rank to ties, but skips subsequent numbers (e.g., 1, 1, 3).
*   `DENSE_RANK()`: Gives the same rank to ties, and does *not* skip numbers (e.g., 1, 1, 2).

**Input Data (`scores` table):**
| student | score |
| :--- | :--- |
| Alice | 90 |
| Bob | 90 |
| Charlie | 85 |

**SQL Query:**
```sql
-- Example showing all three functions side-by-side
SELECT student, score,
       ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
       RANK() OVER (ORDER BY score DESC) AS rank_val,
       DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank_val
FROM scores;
```

**Output:**
| student | score | row_num | rank_val | dense_rank_val |
| :--- | :--- | :--- | :--- | :--- |
| Alice | 90 | 1 | 1 | 1 |
| Bob | 90 | 2 | 1 | 1 |
| Charlie | 85 | 3 | 3 | 2 |

---

### 63. How do you split data into equal groups or quartiles in SQL?
**Answer:**  
The `NTILE()` window function is used to distribute rows into a specified number of approximately equal groups (buckets). This is incredibly useful for finding quartiles, deciles, or percentiles.

**Input Data (`sales_reps` table):**
| name | revenue |
| :--- | :--- |
| Dan | 1000 |
| Eva | 2000 |
| Finn| 3000 |
| Gus | 4000 |

**SQL Query:**
```sql
-- Example of splitting sales reps into 4 groups (Quartiles)
SELECT name, revenue,
       NTILE(4) OVER (ORDER BY revenue DESC) AS quartile
FROM sales_reps;
```

**Output:**
| name | revenue | quartile |
| :--- | :--- | :--- |
| Gus | 4000 | 1 |
| Finn| 3000 | 2 |
| Eva | 2000 | 3 |
| Dan | 1000 | 4 |

---

### 64. How do you safely prevent "Divide by Zero" errors in SQL?
**Answer:**  
A classic trick to avoid your query crashing due to division by zero is to use the `NULLIF()` function. `NULLIF(A, B)` returns `NULL` if A equals B. Since any number divided by `NULL` results in `NULL` (instead of an error), the query runs safely.

**Input Data (`marketing_campaigns` table):**
| campaign | clicks | impressions |
| :--- | :--- | :--- |
| Spring Promo | 50 | 1000 |
| Dead Campaign| 0 | 0 |

**SQL Query:**
```sql
-- Example of calculating Click-Through Rate (CTR) safely
SELECT campaign, clicks, impressions,
       (clicks * 1.0 / NULLIF(impressions, 0)) AS ctr
FROM marketing_campaigns;
```

**Output:**
| campaign | clicks | impressions | ctr |
| :--- | :--- | :--- | :--- |
| Spring Promo | 50 | 1000 | 0.05 |
| Dead Campaign| 0 | 0 | NULL |

---

### 65. How do you extract values from JSON data in SQL?
**Answer:**  
Modern databases (like PostgreSQL, MySQL, and Snowflake) support parsing JSON natively. You can extract specific key-value pairs from a JSON string using specialized operators or functions without needing complex string manipulation.

**Input Data (`event_logs` table):**
| event_id | payload (JSON string) |
| :--- | :--- |
| 101 | {"user_id": 5, "device": "mobile", "status": "success"} |

**SQL Query:**
```sql
-- Example using standard JSON extraction (PostgreSQL syntax: ->> )
-- Note: MySQL uses JSON_EXTRACT(payload, '$.device')
SELECT event_id,
       payload->>'device' AS device_type,
       payload->>'status' AS event_status
FROM event_logs;
```

**Output:**
| event_id | device_type | event_status |
| :--- | :--- | :--- |
| 101 | mobile | success |

---

### 66. When would you use a CROSS JOIN in data analytics?
**Answer:**  
A `CROSS JOIN` creates a Cartesian product, combining every row from the first table with every row from the second table. Analysts use this to generate combinations—like creating a matrix of all store locations and all products to see which stores are missing inventory.

**Input Data (`sizes` and `colors` tables):**
*Sizes:* Small, Large.  
*Colors:* Red, Blue.

**SQL Query:**
```sql
-- Example of generating all possible product variations
SELECT s.size_name, c.color_name
FROM sizes s
CROSS JOIN colors c;
```

**Output:**
| size_name | color_name |
| :--- | :--- |
| Small | Red |
| Small | Blue |
| Large | Red |
| Large | Blue |

---

### 67. How do you calculate Month-over-Month (MoM) Growth?
**Answer:**  
Calculating growth requires comparing the current row's value to the previous row's value. You can combine the `LAG()` window function with standard percentage change math: `(Current - Previous) / Previous`.

**Input Data (`monthly_revenue` table):**
| month | revenue |
| :--- | :--- |
| 2026-01 | 10000 |
| 2026-02 | 12000 |

**SQL Query:**
```sql
-- Example of calculating MoM percentage growth
WITH RevenueWithLag AS (
    SELECT month, revenue,
           LAG(revenue, 1) OVER (ORDER BY month) AS prev_revenue
    FROM monthly_revenue
)
SELECT month, revenue, prev_revenue,
       ROUND(((revenue - prev_revenue) * 100.0 / prev_revenue), 2) AS growth_percent
FROM RevenueWithLag;
```

**Output:**
| month | revenue | prev_revenue | growth_percent |
| :--- | :--- | :--- | :--- |
| 2026-01 | 10000 | NULL | NULL |
| 2026-02 | 12000 | 10000 | 20.00 |

---

### 68. How do you find missing numbers or gaps in sequential data?
**Answer:**  
Finding gaps (like missing invoice numbers) is a classic problem. You can use the `LEAD()` function to look at the "next" row's value. If the difference between the next row and the current row is greater than 1, a gap exists.

**Input Data (`invoices` table):**
| invoice_id |
| :--- |
| 1 |
| 2 |
| 5 |

**SQL Query:**
```sql
-- Example of identifying the start and end of a sequence gap
WITH NextInvoices AS (
    SELECT invoice_id,
           LEAD(invoice_id, 1) OVER (ORDER BY invoice_id) AS next_id
    FROM invoices
)
SELECT invoice_id + 1 AS gap_start,
       next_id - 1 AS gap_end
FROM NextInvoices
WHERE next_id - invoice_id > 1;
```

**Output:**
| gap_start | gap_end |
| :--- | :--- |
| 3 | 4 |

---

### 69. How do you calculate a relative percentile using CUME_DIST()?
**Answer:**  
The `CUME_DIST()` (Cumulative Distribution) window function calculates the relative standing of a value within a group. It answers the question: "What percentage of values are less than or equal to this current value?"

**Input Data (`test_scores` table):**
| student | score |
| :--- | :--- |
| Alice | 50 |
| Bob | 75 |
| Charlie | 90 |
| Dan | 100 |

**SQL Query:**
```sql
-- Example of calculating cumulative distribution
SELECT student, score,
       CUME_DIST() OVER (ORDER BY score ASC) AS percentile_standing
FROM test_scores;
```

**Output:**
| student | score | percentile_standing |
| :--- | :--- | :--- |
| Alice | 50 | 0.25 (Bottom 25%) |
| Bob | 75 | 0.50 (Bottom 50%) |
| Charlie | 90 | 0.75 (Bottom 75%) |
| Dan | 100 | 1.00 (Top 100%) |

---

### 70. How do you control the sorting order of NULL values?
**Answer:**  
By default, some databases sort `NULL` values at the very top, while others put them at the very bottom. You can explicitly control this behavior in your queries by using the `NULLS FIRST` or `NULLS LAST` modifiers in your `ORDER BY` clause.

**Input Data (`tasks` table):**
| task_name | due_date |
| :--- | :--- |
| Task A | 2026-06-10 |
| Task B | NULL |
| Task C | 2026-06-05 |

**SQL Query:**
```sql
-- Example of pushing NULLs to the bottom of a sorted list
SELECT task_name, due_date
FROM tasks
ORDER BY due_date ASC NULLS LAST;
```

**Output:**
| task_name | due_date |
| :--- | :--- |
| Task C | 2026-06-05 |
| Task A | 2026-06-10 |
| Task B | NULL |

---
### 71. How do you find the "Top N" rows per group or category?
**Answer:**  
Finding the top performers (e.g., top 2 highest-paid employees per department) is a classic analytics task. You achieve this by wrapping the `ROW_NUMBER()` or `DENSE_RANK()` window function inside a Common Table Expression (CTE) and filtering the results.

**Input Data (`employees` table):**
| name | department | salary |
| :--- | :--- | :--- |
| Ann | IT | 90000 |
| Bob | IT | 85000 |
| Cam | IT | 80000 |
| Dan | HR | 60000 |
| Eva | HR | 55000 |

**SQL Query:**
```sql
-- Example of finding the Top 2 highest salaries per department
WITH RankedEmployees AS (
    SELECT name, department, salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
    FROM employees
)
SELECT name, department, salary
FROM RankedEmployees
WHERE rank <= 2;
```

**Output:**
| name | department | salary |
| :--- | :--- | :--- |
| Dan | HR | 60000 |
| Eva | HR | 55000 |
| Ann | IT | 90000 |
| Bob | IT | 85000 |

---

### 72. How do you automatically generate subtotals and grand totals in SQL?
**Answer:**  
Instead of writing multiple `GROUP BY` queries and `UNION`ing them together, you can use the `ROLLUP` extension. `ROLLUP` automatically calculates sub-totals for your hierarchical data, as well as a grand total at the very bottom.

**Input Data (`sales` table):**
| region | country | revenue |
| :--- | :--- | :--- |
| Europe | France | 100 |
| Europe | Germany| 200 |
| Asia | Japan | 150 |

**SQL Query:**
```sql
-- Example using ROLLUP to get country totals, region totals, and a grand total
SELECT region, country, SUM(revenue) AS total_revenue
FROM sales
GROUP BY ROLLUP (region, country);
```

**Output:**
| region | country | total_revenue |
| :--- | :--- | :--- |
| Europe | France | 100 |
| Europe | Germany| 200 |
| Europe | NULL | 300 *(Europe Subtotal)* |
| Asia | Japan | 150 |
| Asia | NULL | 150 *(Asia Subtotal)* |
| NULL | NULL | 450 *(Grand Total)* |

---

### 73. What is an UPSERT, and how do you do it in SQL?
**Answer:**  
An "Upsert" is a combination of Update and Insert. If a record already exists, it updates it; if it doesn't exist, it inserts it. This is typically handled using the `MERGE` statement or `INSERT ... ON CONFLICT` / `ON DUPLICATE KEY UPDATE` depending on your database.

**Input Data (`inventory` table):**
| product_id | stock |
| :--- | :--- |
| 101 | 50 |

**SQL Query:**
```sql
-- Example of an UPSERT (PostgreSQL syntax)
-- If product 101 exists, add 20 to stock. If 102 doesn't exist, insert it.
INSERT INTO inventory (product_id, stock)
VALUES (101, 20), (102, 30)
ON CONFLICT (product_id) 
DO UPDATE SET stock = inventory.stock + EXCLUDED.stock;
```

**Output:**
| product_id | stock |
| :--- | :--- |
| 101 | 70 *(Updated)* |
| 102 | 30 *(Inserted)* |

---

### 74. How do you use Regular Expressions (Regex) in SQL?
**Answer:**  
Regular expressions allow for highly complex pattern matching beyond the standard `LIKE` operator. You can use regex to validate formats, extract specific text strings (like emails or phone numbers), or replace complex patterns.

**Input Data (`users` table):**
| user_id | text_entry |
| :--- | :--- |
| 1 | Contact me at john@email.com today. |
| 2 | No email provided here. |

**SQL Query:**
```sql
-- Example of extracting an email address using Regex (PostgreSQL syntax)
SELECT user_id, 
       SUBSTRING(text_entry FROM '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}') AS extracted_email
FROM users
WHERE text_entry ~ '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}';
```

**Output:**
| user_id | extracted_email |
| :--- | :--- |
| 1 | john@email.com |

---

### 75. How do you handle Timezone conversions in SQL?
**Answer:**  
Data is almost always stored in UTC format in data warehouses to maintain consistency. Analysts must frequently convert these UTC timestamps into the local timezone of their users for accurate daily reporting.

**Input Data (`logins` table):**
| user_id | login_time_utc |
| :--- | :--- |
| 1 | 2026-06-01 18:00:00 |

**SQL Query:**
```sql
-- Example of converting UTC to Eastern Standard Time (PostgreSQL/Snowflake syntax)
SELECT user_id,
       login_time_utc,
       login_time_utc AT TIME ZONE 'UTC' AT TIME ZONE 'America/New_York' AS login_time_est
FROM logins;
```

**Output:**
| user_id | login_time_utc | login_time_est |
| :--- | :--- | :--- |
| 1 | 2026-06-01 18:00:00 | 2026-06-01 14:00:00 |

---

### 76. How do you calculate Year-To-Date (YTD) totals in SQL?
**Answer:**  
Year-To-Date calculations are just running totals that reset on January 1st of every year. You can accomplish this by partitioning your window function by the extracted year.

**Input Data (`daily_sales` table):**
| date | revenue |
| :--- | :--- |
| 2025-12-31 | 500 |
| 2026-01-01 | 100 |
| 2026-01-02 | 150 |

**SQL Query:**
```sql
-- Example of calculating YTD revenue
SELECT date, revenue,
       SUM(revenue) OVER (
           PARTITION BY EXTRACT(YEAR FROM date) 
           ORDER BY date
       ) AS ytd_revenue
FROM daily_sales;
```

**Output:**
| date | revenue | ytd_revenue |
| :--- | :--- | :--- |
| 2025-12-31 | 500 | 500 |
| 2026-01-01 | 100 | 100 *(Resets for new year)* |
| 2026-01-02 | 150 | 250 |

---

### 77. How do you unnest or flatten Array data in SQL?
**Answer:**  
Modern databases often store lists of items (like tags or multiple categories) inside a single cell as an Array. To analyze these properly, analysts must "flatten" or "unnest" the array so that each item gets its own row.

**Input Data (`articles` table):**
| article_id | tags (Array) |
| :--- | :--- |
| 101 | ['SQL', 'Data', 'Tech'] |

**SQL Query:**
```sql
-- Example of unnesting an array (PostgreSQL / Snowflake syntax)
SELECT article_id, 
       UNNEST(tags) AS individual_tag
FROM articles;
```

**Output:**
| article_id | individual_tag |
| :--- | :--- |
| 101 | SQL |
| 101 | Data |
| 101 | Tech |

---

### 78. How do you find the maximum value across multiple columns in a single row?
**Answer:**  
The `MAX()` function finds the highest value in a single column vertically across many rows. If you need to look horizontally across multiple columns in the *same* row, you use the `GREATEST()` function.

**Input Data (`student_grades` table):**
| student | math_score | science_score | english_score |
| :--- | :--- | :--- | :--- |
| Alice | 85 | 92 | 88 |

**SQL Query:**
```sql
-- Example of finding the highest score for each student
SELECT student,
       GREATEST(math_score, science_score, english_score) AS best_subject_score
FROM student_grades;
```

**Output:**
| student | best_subject_score |
| :--- | :--- |
| Alice | 92 |

---

### 79. What are the ANY and ALL operators?
**Answer:**  
`ANY` and `ALL` are logical operators used to compare a single value against a range of values returned by a subquery. 
*   `> ANY` means "greater than at least one value in the list".
*   `> ALL` means "greater than every single value in the list".

**Input Data (`sales` and `targets` tables):**
Sales rep Bob made `150` sales. The target list is `[100, 140, 200]`.

**SQL Query:**
```sql
-- Did Bob beat ALL the targets? (150 > 200 is False)
-- Did Bob beat ANY of the targets? (150 > 100 is True)
SELECT name
FROM sales
WHERE total_sales > ALL (SELECT target_amount FROM targets);
```

**Output:**
*Returns an empty set, because Bob (150) is not greater than the highest target (200).*

---

### 80. How do you see how a query is being executed behind the scenes?
**Answer:**  
If a query is running too slowly, analysts use the `EXPLAIN` or `EXPLAIN PLAN` command. By placing this keyword in front of your query, the database will output the "execution plan"—showing whether it is using indexes, performing full table scans, or struggling with inefficient joins.

**Input Data:**
A slow-running query on an `employees` table.

**SQL Query:**
```sql
-- Example of analyzing query performance
EXPLAIN
SELECT * FROM employees WHERE department = 'Sales';
```

**Output:**
*Instead of returning the employees, the database returns a text readout of the execution path, such as:*
`Seq Scan on employees (cost=0.00..15.00 rows=5 width=40) Filter: (department = 'Sales'::text)`

---
### 81. How do you fall back on multiple columns if data is missing?
**Answer:**  
While `COALESCE` is often used to replace a `NULL` with a zero, its true power lies in evaluating a list of columns and returning the first non-null value it finds. This is perfect for prioritizing contact methods or addresses.

**Input Data (`customer_contacts` table):**
| customer_id | work_phone | mobile_phone | home_phone |
| :--- | :--- | :--- | :--- |
| 1 | NULL | 555-0002 | 555-0003 |
| 2 | NULL | NULL | 555-0003 |
| 3 | 555-0001 | 555-0002 | NULL |

**SQL Query:**
```sql
-- Example of prioritizing contact numbers
SELECT customer_id,
       COALESCE(work_phone, mobile_phone, home_phone, 'No Contact Info') AS primary_contact
FROM customer_contacts;
```

**Output:**
| customer_id | primary_contact |
| :--- | :--- |
| 1 | 555-0002 |
| 2 | 555-0003 |
| 3 | 555-0001 |

---

### 82. How do you generate a continuous series of dates (Calendar Table) in SQL?
**Answer:**  
Data analysts often need a continuous list of dates to join against sales data so that days with "zero sales" still show up in reports instead of disappearing. This is done using `GENERATE_SERIES` (PostgreSQL) or recursive CTEs.

**Input Data:** 
No input table needed. We are generating data out of thin air!

**SQL Query:**
```sql
-- Example of generating a date series for the first week of 2026 (PostgreSQL syntax)
SELECT CAST(generate_series(
           '2026-01-01'::DATE, 
           '2026-01-03'::DATE, 
           '1 day'::INTERVAL
       ) AS DATE) AS calendar_date;
```

**Output:**
| calendar_date |
| :--- |
| 2026-01-01 |
| 2026-01-02 |
| 2026-01-03 |

---

### 83. How do you analyze active subscriptions on a specific date?
**Answer:**  
To find out how many users were active on a historical date, you don't look for an exact date match. Instead, you check if your target date falls *between* the user's start date and end date (or if the end date is still NULL).

**Input Data (`subscriptions` table):**
| user | start_date | end_date |
| :--- | :--- | :--- |
| Alice | 2026-01-01 | 2026-01-15 |
| Bob | 2026-01-10 | NULL *(Still active)* |
| Dan | 2026-02-01 | 2026-03-01 |

**SQL Query:**
```sql
-- Example: Who was actively subscribed on January 12th, 2026?
SELECT user
FROM subscriptions
WHERE '2026-01-12' >= start_date 
  AND ('2026-01-12' <= end_date OR end_date IS NULL);
```

**Output:**
| user |
| :--- |
| Alice |
| Bob |

---

### 84. How do you filter Window Functions without using a CTE?
**Answer:**  
In modern cloud data warehouses like Snowflake, BigQuery, and Teradata, you can use the `QUALIFY` clause. It acts exactly like a `HAVING` clause, but specifically for filtering the results of Window Functions without needing to write a bulky subquery or CTE.

**Input Data (`sales` table):**
| employee | region | revenue |
| :--- | :--- | :--- |
| Ann | East | 1000 |
| Bob | East | 800 |
| Cam | West | 1500 |

**SQL Query:**
```sql
-- Example of getting the #1 salesperson per region instantly (Snowflake/BigQuery syntax)
SELECT employee, region, revenue
FROM sales
QUALIFY ROW_NUMBER() OVER (PARTITION BY region ORDER BY revenue DESC) = 1;
```

**Output:**
| employee | region | revenue |
| :--- | :--- | :--- |
| Ann | East | 1000 |
| Cam | West | 1500 |

---

### 85. How do you pad strings with leading zeros?
**Answer:**  
When joining tables, an ID formatted as `123` might fail to join with `00123`. Analysts use `LPAD()` (Left Pad) or `RPAD()` (Right Pad) to standardize string lengths by filling the remaining space with a specific character, usually a zero.

**Input Data (`raw_users` table):**
| user_id |
| :--- |
| 45 |
| 1024 |

**SQL Query:**
```sql
-- Example of ensuring all user IDs are exactly 5 digits long
SELECT user_id,
       LPAD(CAST(user_id AS VARCHAR), 5, '0') AS standardized_id
FROM raw_users;
```

**Output:**
| user_id | standardized_id |
| :--- | :--- |
| 45 | 00045 |
| 1024 | 01024 |

---

### 86. How do you find exact duplicates in a table?
**Answer:**  
To find out which records appear more than once, you group by the columns that *should* be unique, and use the `HAVING` clause to filter for groups that have a `COUNT` greater than 1.

**Input Data (`email_list` table):**
| email | signup_date |
| :--- | :--- |
| a@test.com | 2026-01-01 |
| b@test.com | 2026-01-02 |
| a@test.com | 2026-01-03 |

**SQL Query:**
```sql
-- Example of isolating duplicate email addresses
SELECT email, COUNT(*) as occurrence_count
FROM email_list
GROUP BY email
HAVING COUNT(*) > 1;
```

**Output:**
| email | occurrence_count |
| :--- | :--- |
| a@test.com | 2 |

---

### 87. How do you calculate Daily Active Users (DAU) accurately?
**Answer:**  
To calculate DAU (or MAU for monthly), you cannot just count the rows in a login table, because one user might log in 10 times a day. You must use `COUNT(DISTINCT column_name)` to ensure each user is only counted once per day.

**Input Data (`login_events` table):**
| event_date | user_id |
| :--- | :--- |
| 2026-05-01 | User_1 |
| 2026-05-01 | User_1 |
| 2026-05-01 | User_2 |

**SQL Query:**
```sql
-- Example of calculating true Daily Active Users
SELECT event_date,
       COUNT(DISTINCT user_id) AS daily_active_users
FROM login_events
GROUP BY event_date;
```

**Output:**
| event_date | daily_active_users |
| :--- | :--- |
| 2026-05-01 | 2 |

---

### 88. How do you customize a Window Function's frame size?
**Answer:**  
By default, a window function calculates from the first row to the current row. You can customize this "frame" using `ROWS BETWEEN`. This is useful for analyzing highly specific rolling windows, like the last 30 days or the next 3 rows.

**Input Data (`revenue` table):**
| day | amount |
| :--- | :--- |
| 1 | 10 |
| 2 | 20 |
| 3 | 30 |
| 4 | 40 |

**SQL Query:**
```sql
-- Example of summing ONLY the current row and the row immediately preceding it
SELECT day, amount,
       SUM(amount) OVER (ORDER BY day ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) as rolling_2_day_sum
FROM revenue;
```

**Output:**
| day | amount | rolling_2_day_sum |
| :--- | :--- | :--- |
| 1 | 10 | 10 |
| 2 | 20 | 30 *(10+20)* |
| 3 | 30 | 50 *(20+30)* |
| 4 | 40 | 70 *(30+40)* |

---

### 89. What is a LATERAL JOIN (or CROSS APPLY)?
**Answer:**  
A `LATERAL JOIN` (PostgreSQL/Snowflake) or `CROSS APPLY` (SQL Server) acts like a `FOR EACH` loop in SQL. It allows a subquery or function on the right side of the join to reference columns from the table on the left side of the join, executing row-by-row.

**Input Data (`departments` table):**
| dept_id | dept_name |
| :--- | :--- |
| 1 | Sales |
| 2 | IT |

*(Assume an `employees` table exists with many employees per dept).*

**SQL Query:**
```sql
-- Example: For each department, fetch their single highest paid employee
SELECT d.dept_name, top_emp.name, top_emp.salary
FROM departments d
CROSS JOIN LATERAL (
    SELECT name, salary 
    FROM employees e 
    WHERE e.dept_id = d.dept_id 
    ORDER BY salary DESC 
    LIMIT 1
) AS top_emp;
```

**Output:**
| dept_name | name | salary |
| :--- | :--- | :--- |
| Sales | Ann | 120000 |
| IT | Bob | 115000 |

---

### 90. How do you group data by custom logical buckets without adding columns?
**Answer:**  
You can use a `CASE` statement directly inside a `GROUP BY` clause. This allows you to aggregate data into custom buckets (like age ranges or revenue tiers) on the fly without needing to alter the underlying table or write a CTE.

**Input Data (`users` table):**
| user | age |
| :--- | :--- |
| Ann | 19 |
| Bob | 24 |
| Cam | 35 |

**SQL Query:**
```sql
-- Example of aggregating users by custom age buckets
SELECT 
    CASE 
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 30 THEN '20-30'
        ELSE 'Over 30' 
    END AS age_bucket,
    COUNT(*) AS user_count
FROM users
GROUP BY 1; -- "1" refers to the first column in the SELECT statement
```

**Output:**
| age_bucket | user_count |
| :--- | :--- |
| Under 20 | 1 |
| 20-30 | 1 |
| Over 30 | 1 |

---
### 91. How do you find the FIRST and LAST values in a grouped dataset?
**Answer:**  
While `MIN` and `MAX` find the lowest and highest numerical values, the `FIRST_VALUE()` and `LAST_VALUE()` window functions retrieve the actual first or last record based on a specific chronological sorting order.

**Input Data (`user_purchases` table):**
| user | purchase_date | item |
| :--- | :--- | :--- |
| Ann | 2026-01-01 | Laptop |
| Ann | 2026-05-15 | Mouse |
| Bob | 2026-02-10 | Keyboard |

**SQL Query:**
```sql
-- Example of finding the very first item a user ever bought
SELECT DISTINCT user,
       FIRST_VALUE(item) OVER (PARTITION BY user ORDER BY purchase_date) AS first_purchase
FROM user_purchases;
```

**Output:**
| user | first_purchase |
| :--- | :--- |
| Ann | Laptop |
| Bob | Keyboard |

---

### 92. How do you identify Overlapping Date Ranges?
**Answer:**  
A common problem in analytics (especially in hospitality or HR) is checking if two date ranges overlap. The mathematical rule for overlapping ranges is simple: `Start A <= End B` AND `End A >= Start B`.

**Input Data (`hotel_bookings` table):**
| booking_id | start_date | end_date |
| :--- | :--- | :--- |
| 1 | 2026-06-01 | 2026-06-10 |
| 2 | 2026-06-08 | 2026-06-15 |

**SQL Query:**
```sql
-- Example of finding bookings that overlap with each other
SELECT a.booking_id AS booking_a, b.booking_id AS booking_b
FROM hotel_bookings a
JOIN hotel_bookings b ON a.booking_id != b.booking_id
WHERE a.start_date <= b.end_date AND a.end_date >= b.start_date;
```

**Output:**
| booking_a | booking_b |
| :--- | :--- |
| 1 | 2 |
| 2 | 1 |

---

### 93. What are GROUPING SETS and when do you use them?
**Answer:**  
`GROUPING SETS` allow you to define multiple, highly specific groupings in a single query. Unlike `ROLLUP` (which assumes a hierarchy) or `CUBE` (which generates all possible combinations), `GROUPING SETS` lets you pick and choose exactly which sub-totals you want.

**Input Data (`sales` table):**
| region | segment | revenue |
| :--- | :--- | :--- |
| East | B2B | 100 |
| East | B2C | 50 |

**SQL Query:**
```sql
-- Example of getting the total by region, AND the total by segment, but NO grand total
SELECT region, segment, SUM(revenue) AS total
FROM sales
GROUP BY GROUPING SETS (
    (region),
    (segment)
);
```

**Output:**
| region | segment | total |
| :--- | :--- | :--- |
| East | NULL | 150 |
| NULL | B2B | 100 |
| NULL | B2C | 50 |

---

### 94. How do you "Sessionize" user activity data?
**Answer:**  
Sessionizing means grouping consecutive events (like clicks) into a single "session" if they occur within a specific timeframe (e.g., 30 minutes). This is done by using `LAG()` to find the time difference between events, flagging gaps > 30 mins as a "new session", and taking a running sum of that flag.

**Input Data (`page_views` table):**
| user | click_time |
| :--- | :--- |
| Ann | 10:00:00 |
| Ann | 10:15:00 *(Same session)* |
| Ann | 11:30:00 *(New session)* |

**SQL Query:**
```sql
-- Step 1 & 2: Calculate time diff and flag new sessions
WITH TimeDiffs AS (
    SELECT user, click_time,
           CASE 
             WHEN EXTRACT(EPOCH FROM (click_time - LAG(click_time) OVER (PARTITION BY user ORDER BY click_time))) / 60 > 30 
             THEN 1 ELSE 0 
           END AS is_new_session
    FROM page_views
)
-- Step 3: Cumulative sum to create a session ID
SELECT user, click_time,
       SUM(is_new_session) OVER (PARTITION BY user ORDER BY click_time) AS session_id
FROM TimeDiffs;
```

**Output:**
| user | click_time | session_id |
| :--- | :--- | :--- |
| Ann | 10:00:00 | 0 |
| Ann | 10:15:00 | 0 |
| Ann | 11:30:00 | 1 |

---

### 95. How do you create a Pareto Chart (80/20 Rule) in SQL?
**Answer:**  
Pareto analysis identifies the top factors driving a metric (e.g., "which 20% of products drive 80% of sales?"). You do this by calculating a running total and dividing it by the grand total to get a cumulative percentage.

**Input Data (`products` table):**
| product | sales |
| :--- | :--- |
| Prod A | 800 |
| Prod B | 150 |
| Prod C | 50 |

**SQL Query:**
```sql
-- Example of generating a running percentage
WITH RunningTotals AS (
    SELECT product, sales,
           SUM(sales) OVER (ORDER BY sales DESC) AS running_total,
           SUM(sales) OVER () AS grand_total
    FROM products
)
SELECT product, sales,
       (running_total * 100.0 / grand_total) AS cumulative_percent
FROM RunningTotals;
```

**Output:**
| product | sales | cumulative_percent |
| :--- | :--- | :--- |
| Prod A | 800 | 80.00% |
| Prod B | 150 | 95.00% |
| Prod C | 50 | 100.00% |

---

### 96. How does the ON clause differ from the WHERE clause in a LEFT JOIN?
**Answer:**  
In a `LEFT JOIN`, conditions placed in the `ON` clause determine *how* the tables are joined, but keep all rows from the left table. Conditions placed in the `WHERE` clause filter the *final* result set, effectively turning your `LEFT JOIN` into an `INNER JOIN`.

**Input Data (`users` and `orders` tables):**
*Users:* Ann, Bob.  
*Orders:* Ann made an order on '2026-05-01'. Bob made no orders.

**SQL Query:**
```sql
-- Query 1: Condition in ON clause (Keeps Bob!)
SELECT u.name, o.order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.order_date = '2026-05-01';

-- Query 2: Condition in WHERE clause (Removes Bob!)
SELECT u.name, o.order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.order_date = '2026-05-01';
```

---

### 97. How do you calculate the number of Working Days between two dates?
**Answer:**  
To calculate working days (excluding weekends), you can extract the day of the week for your date ranges. Many data warehouses simplify this with built-in functions, or you can use date math.

**Input Data:**
Start Date: `2026-05-01` (Friday), End Date: `2026-05-05` (Tuesday).

**SQL Query:**
```sql
-- Example logic for working days (BigQuery / Snowflake conceptual approach)
-- Subtract weekends from the total day difference
SELECT 
    start_date, end_date,
    (DATEDIFF(day, start_date, end_date) + 1) 
    - (DATEDIFF(week, start_date, end_date) * 2) 
    AS working_days
FROM projects;
```

**Output:**
| start_date | end_date | working_days |
| :--- | :--- | :--- |
| 2026-05-01 | 2026-05-05 | 3 *(Fri, Mon, Tue)* |

---

### 98. How do you safely compare two columns that might both be NULL?
**Answer:**  
In standard SQL, `NULL = NULL` evaluates to `UNKNOWN` (False), not True. If you need to check if two columns are exactly identical, even if they are both NULL, you use the `IS NOT DISTINCT FROM` operator (or `<=>` in MySQL).

**Input Data (`records` table):**
| id | old_status | new_status |
| :--- | :--- | :--- |
| 1 | NULL | NULL |
| 2 | Active | Active |

**SQL Query:**
```sql
-- Example of checking for true equality, accounting for NULLs
SELECT id, old_status, new_status
FROM records
WHERE old_status IS NOT DISTINCT FROM new_status;
```

**Output:**
| id | old_status | new_status |
| :--- | :--- | :--- |
| 1 | NULL | NULL |
| 2 | Active | Active |

---

### 99. How do you create dynamic bins (a Histogram) in SQL?
**Answer:**  
To create a frequency distribution or histogram, you can mathematically force continuous numbers into specific buckets (e.g., intervals of 10 or 100) using the `FLOOR()` or `ROUND()` functions.

**Input Data (`transactions` table):**
| user | amount |
| :--- | :--- |
| Ann | 25 |
| Bob | 29 |
| Cam | 42 |

**SQL Query:**
```sql
-- Example of bucketing transaction amounts into intervals of $10
SELECT 
    FLOOR(amount / 10) * 10 AS bin_start,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY bin_start
ORDER BY bin_start;
```

**Output:**
| bin_start | transaction_count |
| :--- | :--- |
| 20 | 2 *(Ann, Bob)* |
| 40 | 1 *(Cam)* |

---

### 100. How do you un-pivot an array into rows without an UNNEST function?
**Answer:**  
In legacy databases that lack native JSON or Array unnesting functions, analysts must use a "Tally Table" or "Numbers Table" (a table containing numbers 1 through 100) combined with string splitting logic to manually un-pivot delimited strings into rows.

**Input Data (`tags` table):**
| id | csv_tags |
| :--- | :--- |
| 1 | "SQL,Python,Tableau" |

**SQL Query:**
```sql
-- Example using a modern built-in alternative (SQL Server syntax)
-- Modern equivalent of the old Tally Table trick
SELECT id, value AS individual_tag
FROM tags
CROSS APPLY STRING_SPLIT(csv_tags, ',');
```

**Output:**
| id | individual_tag |
| :--- | :--- |
| 1 | SQL |
| 1 | Python |
| 1 | Tableau |

---

If you found this repository helpful, please give it a star! Stay updated with my latest content and projects!
---

If you found this repository helpful, please give it a star!

Follow me on:
- [LinkedIn](https://www.linkedin.com/in/pranil-k-45235858)
- [GitHub](https://github.com/pranilkochar)

Stay updated with my latest content and projects!
