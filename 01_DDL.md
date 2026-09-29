# SQL Command Reference (DDL + DML)

Notes:
- Syntax shown is generally ANSI SQL / Snowflake. Where other databases (MySQL, Postgres, SQL Server) differ meaningfully, it's called out.
- DDL = Data Definition Language (structure: CREATE, ALTER, DROP, TRUNCATE, RENAME, COMMENT).
- DML = Data Manipulation Language (data: INSERT, UPDATE, DELETE, MERGE).
- DCL = Data Control Language (GRANT, REVOKE) — included briefly at the end for completeness.

---

## 1. CREATE

### 1.1 Create a database / schema
```sql
CREATE DATABASE mydb;
CREATE SCHEMA mydb.public;

-- avoid error if already exists
CREATE DATABASE IF NOT EXISTS mydb;
```

### 1.2 Create a table (basic)
```sql
CREATE TABLE employees (
    emp_id      INT PRIMARY KEY,
    first_name  VARCHAR(50) NOT NULL,
    last_name   VARCHAR(50) NOT NULL,
    email       VARCHAR(100) UNIQUE,
    salary      NUMBER(10,2) DEFAULT 0,  10 digits in total and 2 after decimal are allowed
    dept_id     INT,
    hire_date   DATE DEFAULT CURRENT_DATE()
);
```
- `PRIMARY KEY` — unique + not null identifier for the row.
- `NOT NULL` — column must always have a value.
- `UNIQUE` — no duplicate values allowed. multiple nulls can be present !!
- `DEFAULT` — value used when none is supplied on INSERT.

### 1.3 Create table with a foreign key
```sql
CREATE TABLE departments (
    dept_id     INT PRIMARY KEY,
    dept_name   VARCHAR(50) NOT NULL
);

CREATE TABLE employees (
    emp_id      INT PRIMARY KEY,
    dept_id     INT,
    CONSTRAINT fk_dept FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
Foreign keys in a table can be duplicates ! 
```
Note: Snowflake accepts FK/PK/UNIQUE syntax but does **not enforce** them (metadata only, used by optimizer/tools). MySQL/Postgres/SQL Server enforce them.

for normal snowflake tables :
constraints enforced by snowflake : not null , default, autoincrement
not enforced by snowflake : pk , fk, unique , check

for hybrid tables :


### 1.4 Create table from another table (copy structure + data)
```sql
-- structure + data
CREATE TABLE employees_backup AS
SELECT * FROM employees;

-- structure + data, filtered
CREATE TABLE high_earners AS
SELECT * FROM employees WHERE salary > 100000;

-- structure only, no data
CREATE TABLE employees_empty LIKE employees;
```

### 1.5 Create table if not exists / or replace
```sql
CREATE TABLE IF NOT EXISTS employees (...);   -- skip if it already exists
CREATE OR REPLACE TABLE employees (...);      -- drop + recreate (Snowflake-friendly, destructive!)
```

### 1.6 Create temporary / transient table (Snowflake)
```sql
CREATE TEMPORARY TABLE tmp_calc (id INT, val NUMBER);   -- session-scoped, auto-dropped
CREATE TRANSIENT TABLE staging_data (id INT, val NUMBER); -- no fail-safe, cheaper storage
```

### 1.7 Create view
```sql
CREATE VIEW active_employees AS
SELECT * FROM employees WHERE status = 'ACTIVE';

CREATE OR REPLACE VIEW active_employees AS ...;

-- materialized view (Snowflake): physically stores results, auto-refreshed
CREATE MATERIALIZED VIEW mv_dept_totals AS
SELECT dept_id, SUM(salary) AS total_salary
FROM employees
GROUP BY dept_id;
```

### 1.8 Create index
```sql
CREATE INDEX idx_emp_dept ON employees(dept_id);
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

Indexing helps READ performance but makes INSERT , UPDATE, DELETE ( write ) slower
Apply Index to those columns where u need filtering, join, aggregation,etc
Seperate Indexes can be used else columns can be clubbed to form a index, 
there can be n no. of indexes although not recommended

```
Note: Snowflake doesn't use traditional indexes (it auto-manages micro-partitions); this applies to MySQL/Postgres/SQL Server. Snowflake instead uses `CLUSTER BY`:
```sql
ALTER TABLE employees CLUSTER BY (dept_id);
```

### 1.9 Create sequence (auto-incrementing numbers)
```sql
CREATE SEQUENCE emp_seq START = 1 INCREMENT = 1;

-- usage
INSERT INTO employees (emp_id, first_name) VALUES (emp_seq.NEXTVAL, 'Asha');

-- or use AUTOINCREMENT / IDENTITY directly on a column
CREATE TABLE employees (
    emp_id INT AUTOINCREMENT START 1 INCREMENT 1,
    first_name VARCHAR(50)
);
```

---

## 2. ALTER TABLE

### 2.1 Add a column
```sql
ALTER TABLE employees ADD COLUMN middle_name VARCHAR(50);

-- add with default value
ALTER TABLE employees ADD COLUMN is_active BOOLEAN DEFAULT TRUE;

-- add multiple columns at once
ALTER TABLE employees ADD COLUMN phone VARCHAR(20), ADD COLUMN city VARCHAR(50);
```

### 2.2 Drop a column
```sql
ALTER TABLE employees DROP COLUMN middle_name;
ALTER TABLE employees DROP COLUMN phone, DROP COLUMN city; -- multiple
```

### 2.3 Rename a column
```sql
ALTER TABLE employees RENAME COLUMN first_name TO given_name;
```

### 2.4 Change a column's data type
```sql
ALTER TABLE employees ALTER COLUMN salary SET DATA TYPE NUMBER(12,2);

-- Snowflake shorthand
ALTER TABLE employees MODIFY COLUMN salary NUMBER(12,2);

-- MySQL syntax (differs)
-- ALTER TABLE employees MODIFY salary DECIMAL(12,2);
```
Caveat: type changes only succeed if existing data is compatible (e.g., widening VARCHAR is fine; narrowing or incompatible casts may fail).

### 2.5 Set / drop NOT NULL, DEFAULT
```sql
ALTER TABLE employees ALTER COLUMN email SET NOT NULL;
ALTER TABLE employees ALTER COLUMN email DROP NOT NULL;

ALTER TABLE employees ALTER COLUMN salary SET DEFAULT 0;
ALTER TABLE employees ALTER COLUMN salary DROP DEFAULT;
```

### 2.6 Rename a table
```sql
ALTER TABLE employees RENAME TO staff;
```

### 2.7 Add / drop constraints
```sql
ALTER TABLE employees ADD CONSTRAINT uq_email UNIQUE (email);
ALTER TABLE employees DROP CONSTRAINT uq_email;

ALTER TABLE employees ADD CONSTRAINT fk_dept FOREIGN KEY (dept_id) REFERENCES departments(dept_id);
```

### 2.8 Swap table (Snowflake trick — zero-downtime table replace)
```sql
CREATE TABLE employees_new AS SELECT * FROM employees; -- build the new version
ALTER TABLE employees SWAP WITH employees_new;         -- atomic swap
DROP TABLE employees_new;                              -- old data now here, drop if unneeded
```

---

## 3. DROP / TRUNCATE

### 3.1 Drop table
```sql
DROP TABLE employees;
DROP TABLE IF EXISTS employees;
```
Removes the table entirely, including structure.

### 3.2 Truncate table
```sql
TRUNCATE TABLE employees;
```
Deletes all rows but **keeps the table structure**. Faster than `DELETE` (no row-by-row logging in most engines); cannot be filtered with `WHERE`.

### 3.3 Drop database / schema / view / index
```sql
DROP DATABASE mydb;
DROP SCHEMA mydb.public;
DROP VIEW active_employees;
DROP INDEX idx_emp_dept;          -- syntax varies by engine
DROP SEQUENCE emp_seq;
```

### 3.4 Drop column (see 2.2) vs Drop table
- `DROP COLUMN` — removes one column, table stays.
- `DROP TABLE` — removes the whole table.

---

## 4. RENAME / COMMENT

```sql
ALTER TABLE employees RENAME TO staff;
ALTER TABLE staff RENAME COLUMN given_name TO first_name;

COMMENT ON TABLE employees IS 'Master employee record';
COMMENT ON COLUMN employees.salary IS 'Annual salary in USD';
```

---

## 5. INSERT (DML)

### 5.1 Insert a single row
```sql
INSERT INTO employees (emp_id, first_name, last_name, email, salary, dept_id)
VALUES (1, 'Asha', 'Rao', 'asha@corp.com', 75000, 10);
```

### 5.2 Insert without listing columns (must match table's column order exactly — risky, avoid in production)
```sql
INSERT INTO employees
VALUES (2, 'Ravi', 'Kumar', 'ravi@corp.com', 68000, 10, DEFAULT);
```

### 5.3 Insert multiple rows at once
```sql
INSERT INTO employees (emp_id, first_name, last_name, email, salary, dept_id)
VALUES
    (3, 'Meera', 'Nair', 'meera@corp.com', 82000, 20),
    (4, 'Sam', 'Iyer', 'sam@corp.com', 91000, 20),
    (5, 'Priya', 'Shah', 'priya@corp.com', 77000, 30);
```

### 5.4 Insert from a SELECT (copy from another table/query)
```sql
INSERT INTO employees_backup (emp_id, first_name, last_name, salary)
SELECT emp_id, first_name, last_name, salary
FROM employees
WHERE dept_id = 20;
```

### 5.5 Insert with default values
```sql
INSERT INTO employees (emp_id, first_name, last_name)
VALUES (6, 'Neha', 'Joshi');   -- other columns fall back to DEFAULT or NULL
```

### 5.6 Insert only if not already present (avoid duplicates) — use MERGE (section 8) or:
```sql
INSERT INTO employees (emp_id, first_name)
SELECT 7, 'Karan'
WHERE NOT EXISTS (SELECT 1 FROM employees WHERE emp_id = 7);
```

---

## 6. UPDATE (DML)

### 6.1 Update all rows
```sql
UPDATE employees SET salary = salary * 1.10;   -- 10% raise for everyone
```

### 6.2 Update with a condition
```sql
UPDATE employees
SET salary = salary * 1.15
WHERE dept_id = 20;
```

### 6.3 Update multiple columns
```sql
UPDATE employees
SET salary = 95000, dept_id = 40
WHERE emp_id = 3;
```

### 6.4 Update using a subquery
```sql
UPDATE employees e
SET dept_id = (SELECT dept_id FROM departments WHERE dept_name = 'Engineering')
WHERE e.emp_id = 5;
```

### 6.5 Update joining another table (Snowflake / Postgres style)
```sql
UPDATE employees e
SET salary = e.salary + b.bonus
FROM bonuses b
WHERE e.emp_id = b.emp_id;
```
(MySQL uses `UPDATE employees e JOIN bonuses b ON ... SET ...` instead of `FROM`.)

---

## 7. DELETE (DML)

### 7.1 Delete specific rows
```sql
DELETE FROM employees WHERE dept_id = 30;
```

### 7.2 Delete all rows (keeps structure, logs each row — slower than TRUNCATE)
```sql
DELETE FROM employees;
```

### 7.3 Delete using a subquery
```sql
DELETE FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE dept_name = 'Closed Division');
```

### 7.4 Delete with join reference (Snowflake / Postgres style USING)
```sql
DELETE FROM employees e
USING departments d
WHERE e.dept_id = d.dept_id AND d.dept_name = 'Closed Division';
```

---

## 8. MERGE (a.k.a. "UPSERT" — insert if new, update if exists)
```sql
MERGE INTO employees AS tgt
USING staging_employees AS src
ON tgt.emp_id = src.emp_id
WHEN MATCHED THEN
    UPDATE SET tgt.salary = src.salary, tgt.dept_id = src.dept_id
WHEN NOT MATCHED THEN
    INSERT (emp_id, first_name, last_name, salary, dept_id)
    VALUES (src.emp_id, src.first_name, src.last_name, src.salary, src.dept_id);
```
Very common in ETL: syncs a target table with a source/staging table in one statement.

---

## 9. Quick DDL vs DML vs DCL cheat sheet

| Category | Purpose                          | Commands |
|----------|-----------------------------------|----------|
| DDL      | Define/alter structure            | CREATE, ALTER, DROP, TRUNCATE, RENAME, COMMENT |
| DML      | Manipulate data                   | INSERT, UPDATE, DELETE, MERGE |
| DQL      | Query data                        | SELECT |
| DCL      | Control access                    | GRANT, REVOKE |
| TCL      | Manage transactions               | COMMIT, ROLLBACK, SAVEPOINT, BEGIN/START TRANSACTION |

### DCL quick reference
```sql
GRANT SELECT, INSERT ON TABLE employees TO ROLE analyst;
REVOKE INSERT ON TABLE employees FROM ROLE analyst;
```

### TCL quick reference
```sql
BEGIN;                       -- start transaction (or: START TRANSACTION;)
UPDATE employees SET salary = salary * 1.05 WHERE dept_id = 10;
DELETE FROM employees WHERE emp_id = 99;
ROLLBACK;                    -- undo everything since BEGIN
-- or:
COMMIT;                      -- make all changes permanent
```
Note: DDL statements (CREATE/ALTER/DROP) auto-commit in most databases including Snowflake — they cannot be rolled back like DML.

---

## 10. Common gotchas / things worth remembering
- `TRUNCATE` is DDL-ish (fast, no WHERE, resets identity/auto-increment in most engines); `DELETE` is DML (row-by-row, supports WHERE, can be rolled back).
- `DROP` removes structure + data; `DELETE`/`TRUNCATE` remove only data.
- Snowflake `CREATE OR REPLACE` is destructive — it fully drops and recreates the object (loses grants/history unless cloned first).
- Use `CREATE TABLE ... CLONE ...` in Snowflake for a zero-copy instant snapshot instead of `CREATE TABLE AS SELECT` when you just want a full copy:
  ```sql
  CREATE TABLE employees_clone CLONE employees;
  ```
- Always test destructive DDL (`DROP`, `TRUNCATE`, `ALTER ... DROP COLUMN`) on a clone or in a transaction-safe environment first.
