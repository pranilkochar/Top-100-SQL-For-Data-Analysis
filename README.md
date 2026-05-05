# 100 SQL Interview Questions for Data Analytics and Answers

Welcome to my Analytics World! This repository contains 100 essential SQL interview questions. To make learning easier, I have included sample **Input Data** and the expected **Output** for the queries so you can see exactly how the data transforms.

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

*(Questions 11-100 to follow in subsequent updates)*

---

If you found this repository helpful, please give it a star!

Follow me on:
- [LinkedIn](https://www.linkedin.com/in/pranilkochar)
- [GitHub](https://github.com/pranilkochar)

Stay updated with my latest content and projects!
