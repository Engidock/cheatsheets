# SQL Cheatsheet

Complete detailed reference guide for SQL database development and administration.

## 🎯 SQL Basics & Syntax

### SELECT Statement

Basic SELECT:

```sql
SELECT * FROM employees;
SELECT name, email, salary FROM employees;
SELECT id, name AS employee_name FROM employees;
SELECT DISTINCT department FROM employees;
SELECT COUNT(*) FROM employees;
SELECT * FROM employees LIMIT 10;
SELECT * FROM employees LIMIT 10 OFFSET 5;
```

WHERE Clause:

```sql
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE department = 'Sales';
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;
SELECT * FROM employees WHERE name LIKE 'John%';
SELECT * FROM employees WHERE email IN ('a@ex.com', 'b@ex.com');
SELECT * FROM employees WHERE age IS NULL;
SELECT * FROM employees WHERE age IS NOT NULL;
```

Operators & Conditions:

```sql
-- Comparison operators
-- =  (equal), != or <>  (not equal)
-- <  (less than), >  (greater than)
-- <= (less or equal), >= (greater or equal)

-- Logical operators
-- AND: All conditions true
-- OR:  At least one true
-- NOT: Negate condition

-- String matching
-- LIKE 'pattern': Pattern matching
-- % (any chars), _ (single char)
```

### ORDER & GROUP

Sorting Results:

```sql
SELECT * FROM employees ORDER BY salary;
SELECT * FROM employees ORDER BY salary DESC;
SELECT * FROM employees ORDER BY department ASC, salary DESC;
SELECT * FROM employees ORDER BY RAND() LIMIT 5;
```

Grouping Data:

```sql
SELECT department, COUNT(*) FROM employees GROUP BY department;
SELECT department, AVG(salary) FROM employees GROUP BY department;
SELECT department, COUNT(*) FROM employees GROUP BY department HAVING COUNT(*) > 5;
SELECT department, SUM(salary) FROM employees GROUP BY department ORDER BY SUM(salary) DESC;
```

## 🔗 JOINs & Complex Queries

### Join Types

INNER JOIN:

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;
-- Only matching records from both tables
```

LEFT JOIN:

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
-- All from left table, matching from right
```

RIGHT JOIN:

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;
-- All from right table, matching from left
```

FULL OUTER JOIN:

```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;
-- All from both tables
```

CROSS JOIN:

```sql
SELECT e.name, p.product_name
FROM employees e
CROSS JOIN products p;
-- Cartesian product
```

### Subqueries

Subquery Examples:

```sql
SELECT name FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);

SELECT * FROM employees WHERE dept_id IN (
  SELECT id FROM departments WHERE location = 'NYC'
);

SELECT e.name, e.salary FROM employees e
WHERE (SELECT COUNT(*) FROM employees WHERE dept_id = e.dept_id) > 3;

SELECT * FROM (
  SELECT name, salary FROM employees WHERE salary > 50000
) AS high_earners;
```

## 📝 Data Modification

### INSERT, UPDATE, DELETE

INSERT Data:

```sql
INSERT INTO employees (name, email, salary)
VALUES ('John Doe', 'john@ex.com', 55000);

INSERT INTO employees (name, email, salary) VALUES
  ('John', 'john@ex.com', 55000),
  ('Jane', 'jane@ex.com', 60000),
  ('Bob', 'bob@ex.com', 52000);

INSERT INTO employees SELECT * FROM temp_employees;
```

UPDATE Data:

```sql
UPDATE employees SET salary = 60000 WHERE id = 1;
UPDATE employees SET salary = salary * 1.1 WHERE department = 'Sales';
UPDATE employees SET salary = 55000, department = 'HR' WHERE id = 5;
UPDATE employees SET last_review = NOW() WHERE manager_id = 3;
```

DELETE Data:

```sql
DELETE FROM employees WHERE id = 1;
DELETE FROM employees WHERE salary < 40000;
DELETE FROM employees WHERE hire_date < '2020-01-01';
TRUNCATE TABLE employees;  -- Delete all rows fast
```

## ⚡ Aggregate Functions

### Common Functions

Aggregation:

```sql
-- COUNT(*)        Count all rows
-- COUNT(column)   Count non-null values
-- SUM(column)     Sum values
-- AVG(column)     Average value
-- MIN(column)     Minimum value
-- MAX(column)     Maximum value
-- GROUP_CONCAT(column)  Concatenate values
```

Examples:

```sql
SELECT COUNT(*) AS total FROM employees;
SELECT MAX(salary) AS highest_salary FROM employees;
SELECT AVG(salary) AS avg_salary FROM employees;
```

### String Functions

String Operations:

```sql
-- CONCAT(str1, str2, ...)          Concatenate strings
-- CONCAT_WS(sep, str1, str2, ...)  With separator
-- LENGTH(str)                      String length
-- UPPER(str)                       Uppercase
-- LOWER(str)                       Lowercase
-- SUBSTRING(str, pos, len)         Extract substring
-- REPLACE(str, from, to)           Replace text
-- TRIM(str)                        Remove whitespace
-- LTRIM(str), RTRIM(str)           Trim sides
```

Examples:

```sql
SELECT CONCAT(first_name, ' ', last_name) FROM employees;
SELECT UPPER(name) FROM employees;
SELECT SUBSTRING(email, 1, POSITION('@' IN email) - 1) FROM employees;
```

### Date Functions

Date Operations:

```sql
-- NOW()                                  Current date/time
-- DATE(expr)                             Extract date
-- TIME(expr)                             Extract time
-- YEAR(date), MONTH(date), DAY(date)
-- DATE_ADD(date, INTERVAL n DAY/MONTH/YEAR)
-- DATE_SUB(date, INTERVAL n DAY/MONTH/YEAR)
-- DATEDIFF(date1, date2)                 Days between
-- DATE_FORMAT(date, format)              Format date
```

Examples:

```sql
SELECT * FROM employees WHERE hire_date > DATE_SUB(NOW(), INTERVAL 1 YEAR);
SELECT DATE_FORMAT(hire_date, '%Y-%m-%d') FROM employees;
SELECT DATEDIFF(NOW(), hire_date) AS days_employed FROM employees;
```

## 🏗️ Table & Database Operations

### CREATE & ALTER

Create Table:

```sql
CREATE TABLE employees (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE,
  salary DECIMAL(10, 2),
  hire_date DATE,
  department VARCHAR(50),
  manager_id INT,
  FOREIGN KEY (manager_id) REFERENCES employees(id)
);

CREATE TABLE IF NOT EXISTS employees (...);
CREATE TEMPORARY TABLE temp_data (...);
CREATE TABLE employees_backup AS SELECT * FROM employees;
```

Alter Table:

```sql
ALTER TABLE employees ADD COLUMN phone VARCHAR(15);
ALTER TABLE employees DROP COLUMN phone;
ALTER TABLE employees MODIFY COLUMN salary DECIMAL(12, 2);
ALTER TABLE employees RENAME COLUMN emp_id TO id;
ALTER TABLE employees ADD PRIMARY KEY (id);
ALTER TABLE employees ADD UNIQUE (email);
```

### Indexes & Keys

Index Management:

```sql
CREATE INDEX idx_email ON employees(email);
CREATE UNIQUE INDEX idx_ssn ON employees(ssn);
CREATE INDEX idx_name ON employees(name(10));  -- Prefix
DROP INDEX idx_email ON employees;
SHOW INDEXES FROM employees;
ANALYZE TABLE employees;
EXPLAIN SELECT * FROM employees WHERE email = 'john@ex.com';
```

## 🔑 Constraints & Validation

### Data Constraints

Constraints:

```sql
-- PRIMARY KEY   Unique identifier
-- FOREIGN KEY   Reference another table
-- UNIQUE        All values unique
-- NOT NULL      Required field
-- DEFAULT       Default value
-- CHECK         Condition validation
```

Examples:

```sql
CREATE TABLE employees (
  id INT PRIMARY KEY,
  email VARCHAR(100) UNIQUE NOT NULL,
  salary DECIMAL(10, 2) CHECK (salary > 0),
  status VARCHAR(20) DEFAULT 'active'
);

ALTER TABLE employees
  ADD CONSTRAINT fk_dept FOREIGN KEY (dept_id) REFERENCES departments(id);
```

## 🔍 Advanced Queries

### Window Functions

Window Functions:

```sql
SELECT name, salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank,
  RANK() OVER (ORDER BY salary DESC) AS salary_rank,
  LAG(salary) OVER (ORDER BY salary) AS prev_salary,
  LEAD(salary) OVER (ORDER BY salary) AS next_salary,
  AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

### Common Table Expressions

CTE & Recursive:

```sql
WITH high_earners AS (
  SELECT * FROM employees WHERE salary > 60000
)
SELECT * FROM high_earners ORDER BY salary DESC;

-- Recursive CTE
WITH RECURSIVE numbers AS (
  SELECT 1 AS num
  UNION ALL
  SELECT num + 1 FROM numbers WHERE num < 10
)
SELECT * FROM numbers;
```

### Set Operations

UNION, INTERSECT, EXCEPT:

```sql
-- UNION (combines, removes duplicates)
SELECT name FROM employees WHERE department = 'Sales'
UNION
SELECT name FROM employees WHERE salary > 70000;

-- UNION ALL (keeps duplicates)
SELECT name FROM current_employees
UNION ALL
SELECT name FROM former_employees;

-- INTERSECT (common rows)
SELECT emp_id FROM employees_2022
INTERSECT
SELECT emp_id FROM employees_2023;

-- EXCEPT (in first, not in second)
SELECT emp_id FROM current_employees
EXCEPT
SELECT emp_id FROM terminated_employees;
```

## 💾 Transactions & ACID

### Transaction Control

Transaction Basics:

```sql
START TRANSACTION;
BEGIN;

-- Multiple operations
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;    -- Save changes
ROLLBACK;  -- Undo changes

-- Savepoints
SAVEPOINT sp1;
DELETE FROM logs WHERE date < '2020-01-01';
ROLLBACK TO sp1;
```

Isolation Levels:

```sql
-- Set isolation level
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check current
SELECT @@GLOBAL.tx_isolation, @@SESSION.tx_isolation;
```

## 🔐 Security & Performance

### Best Practices

Security:

```sql
-- Use parameterized queries to prevent SQL injection
-- Prepared statement (varies by language)
PREPARE stmt FROM 'SELECT * FROM users WHERE id = ?';
SET @id = 1;
EXECUTE stmt USING @id;

-- Create views for data abstraction
CREATE VIEW employee_summary AS
SELECT id, name, department FROM employees WHERE status = 'active';

-- Use roles and permissions
CREATE ROLE analyst;
GRANT SELECT ON database.* TO analyst;
REVOKE INSERT, UPDATE, DELETE ON database.* FROM analyst;
```

Performance Tips:

```sql
-- Use appropriate indexes
EXPLAIN SELECT * FROM employees WHERE email = 'john@ex.com';

-- Avoid SELECT *
SELECT id, name, email FROM employees;

-- Use LIMIT for large results
SELECT * FROM logs LIMIT 1000;

-- Proper JOIN usage
SELECT e.name, d.dept_name FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- Denormalize carefully if needed
-- Archive old data
-- Use partitioning for large tables
```

## 🗄️ Views, Procedures & Triggers

### Database Objects

Views:

```sql
CREATE VIEW active_employees AS
SELECT id, name, email, salary FROM employees WHERE status = 'active';

ALTER VIEW active_employees AS
SELECT id, name, email FROM employees WHERE status = 'active';

DROP VIEW active_employees;

CREATE OR REPLACE VIEW employee_report AS
SELECT department, COUNT(*) AS total, AVG(salary)
FROM employees
GROUP BY department;
```

Stored Procedures:

```sql
DELIMITER $$
CREATE PROCEDURE GetEmployeesByDept(IN dept_name VARCHAR(50))
BEGIN
  SELECT * FROM employees WHERE department = dept_name;
END$$
DELIMITER ;

CALL GetEmployeesByDept('Sales');
DROP PROCEDURE GetEmployeesByDept;
```

Triggers:

```sql
CREATE TRIGGER salary_audit AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
  INSERT INTO salary_changes (emp_id, old_salary, new_salary, changed_at)
  VALUES (NEW.id, OLD.salary, NEW.salary, NOW());
END;

DROP TRIGGER salary_audit;
```

## 📊 Normalization & Design

### Database Design

Normalization Levels:

```text
1NF:  Atomic values, no repeating groups
2NF:  1NF + Remove partial dependencies
3NF:  2NF + Remove transitive dependencies
BCNF: 3NF + For complex relationships
4NF:  BCNF + Multivalued dependencies
5NF:  4NF + Join dependencies
```

Example: Normalize to 3NF:

```sql
-- Bad: employees (employee_id, name, projects)
-- Good:
-- employees   (id, name)
-- projects    (id, name)
-- assignments (emp_id, proj_id)
```

## 🐛 Troubleshooting & Debugging

### Common Issues

Debugging Queries:

```sql
-- Show execution plan
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
EXPLAIN FORMAT=JSON SELECT * FROM employees;

-- Check query statistics
SHOW STATUS LIKE 'Threads%';
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

-- Profile queries (MySQL)
SET profiling = 1;
SELECT * FROM employees WHERE id = 1;
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;
```

Common Errors:

```text
Syntax errors:          Check query formatting
Type mismatch:          Check column data types
Foreign key constraint: Verify relationships
Deadlock:                Check transaction isolation
Out of memory:           Optimize query, use LIMIT
Access denied:           Check user permissions
```

## 📋 Quick Reference Table

| Command | Purpose | Example |
|---|---|---|
| SELECT | Query data | `SELECT * FROM table` |
| INSERT | Add data | `INSERT INTO table VALUES (...)` |
| UPDATE | Modify data | `UPDATE table SET col = val` |
| DELETE | Remove data | `DELETE FROM table WHERE id = 1` |
| CREATE TABLE | Create table | `CREATE TABLE name (...)` |
| ALTER TABLE | Modify table | `ALTER TABLE name ADD col` |
| DROP TABLE | Delete table | `DROP TABLE name` |
| JOIN | Combine tables | `INNER JOIN, LEFT JOIN` |
| GROUP BY | Group results | `GROUP BY column` |
| ORDER BY | Sort results | `ORDER BY column ASC` |
| HAVING | Filter groups | `HAVING COUNT(*) > 5` |
| CREATE INDEX | Optimize query | `CREATE INDEX idx ON table(col)` |

## ✅ SQL Best Practices

**Query Optimization**
- Use indexes for frequently queried columns
- Avoid SELECT * when possible
- Use LIMIT for large datasets
- Join properly (INNER vs LEFT vs RIGHT)
- Use subqueries wisely

**Data Integrity**
- Define constraints properly
- Use transactions for related operations
- Validate data before insertion
- Implement referential integrity
- Regular backups and testing

**Security**
- Use parameterized queries
- Implement principle of least privilege
- Encrypt sensitive data
- Audit database access
- Validate all inputs

**Maintenance**
- Monitor query performance
- Archive old data
- Rebuild indexes periodically
- Document schema changes
- Test before production

**💡 Pro Tips:**
- Use meaningful column names
- Normalize properly (3NF typical)
- Document complex queries
- Use views for reusability
- Plan for scaling early

**⚠️ Never:**
- Skip backup strategy
- Ignore normalization
- Use SELECT * in production
- Skip input validation
- Hardcode credentials

## 🎓 SQL Data Types Reference

**Numeric Types**
- `INT` / `BIGINT`
- `DECIMAL(p,s)`
- `FLOAT` / `DOUBLE`
- `TINYINT`
- `SMALLINT`

**String Types**
- `VARCHAR(n)`
- `CHAR(n)`
- `TEXT`
- `BLOB`
- `ENUM`

**Date/Time**
- `DATE`
- `TIME`
- `DATETIME`
- `TIMESTAMP`
- `YEAR`

**Special Types**
- `JSON`
- `BOOLEAN`
- `UUID`
- `GEOMETRY`
- `SPATIAL`

**Aggregate Functions**
- `COUNT()`
- `SUM()`
- `AVG()`
- `MIN()` / `MAX()`
- `GROUP_CONCAT()`

**String Functions**
- `CONCAT()`
- `UPPER()` / `LOWER()`
- `LENGTH()`
- `SUBSTRING()`
- `REPLACE()`

---

*Source: adapted from the SQL cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
