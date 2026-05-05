# Essential SQL Guide for Data Analytics

This guide covers essential SQL concepts every data analyst should know, with sample input data and expected outputs. Whether you are learning SQL from scratch or looking for a quick reference, these practical examples will show you exactly how data transforms with each query.

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

---

If you found this repository helpful, please give it a star!

Follow me on:
- [LinkedIn](https://www.linkedin.com/in/pranil-k-45235858)
- [GitHub](https://github.com/pranilkochar)

Stay updated with my latest content and projects!
