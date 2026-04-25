# 🐘 PostgreSQL Mastery — From Zero to Senior Engineer

> _"Give me six hours to chop down a tree and I will spend the first four sharpening the axe."_ — Abraham Lincoln
>
> Give me six months with databases and I'll spend the first two mastering PostgreSQL.

---

## 📖 Table of Contents

1. [What is PostgreSQL?](#1-what-is-postgresql)
2. [Installation & Setup](#2-installation--setup)
3. [Core Database Concepts](#3-core-database-concepts)
4. [Data Types Deep Dive](#4-data-types-deep-dive)
5. [Table Design & Constraints](#5-table-design--constraints)
6. [CRUD Operations](#6-crud-operations)
7. [Filtering, Sorting & Pagination](#7-filtering-sorting--pagination)
8. [Aggregate Functions & Grouping](#8-aggregate-functions--grouping)
9. [JOINs — The Heart of Relational DBs](#9-joins--the-heart-of-relational-dbs)
10. [Subqueries & CTEs](#10-subqueries--ctes)
11. [Indexes — Make It Blazing Fast](#11-indexes--make-it-blazing-fast)
12. [Transactions & ACID](#12-transactions--acid)
13. [Views](#13-views)
14. [Stored Procedures & Functions](#14-stored-procedures--functions)
15. [Triggers](#15-triggers)
16. [JSON & JSONB Support](#16-json--jsonb-support)
17. [Window Functions](#17-window-functions)
18. [Performance & Query Optimization](#18-performance--query-optimization)
19. [Roles, Users & Security](#19-roles-users--security)
20. [Backup & Restore](#20-backup--restore)
21. [psql Cheat Sheet](#21-psql-cheat-sheet)
22. [Senior-Level Tips & Patterns](#22-senior-level-tips--patterns)

---

## 1. What is PostgreSQL?

PostgreSQL (a.k.a. **Postgres**) is an advanced, open-source **Object-Relational Database Management System (ORDBMS)**. It has been actively developed for over **35 years** and is trusted by companies like Apple, Spotify, Instagram, and Reddit.

### Why PostgreSQL over MySQL?

| Feature              | PostgreSQL      | MySQL                |
| -------------------- | --------------- | -------------------- |
| Full ACID compliance | ✅ Always       | ⚠️ Depends on engine |
| JSON/JSONB support   | ✅ Excellent    | ⚠️ Basic             |
| Window functions     | ✅ Full support | ✅ MySQL 8+          |
| CTEs                 | ✅ Full support | ✅ MySQL 8+          |
| Custom data types    | ✅ Yes          | ❌ No                |
| Concurrency model    | MVCC (superior) | Locking (older)      |
| Full text search     | ✅ Built-in     | ⚠️ Limited           |

> **Bottom line:** PostgreSQL is what serious engineers use when they care about data integrity, performance, and flexibility.

---

## 2. Installation & Setup

### Ubuntu / Debian

```bash
# Update & install
sudo apt update
sudo apt install postgresql postgresql-contrib -y

# Start and enable service on boot
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Check status
sudo systemctl status postgresql
```

### macOS (Homebrew)

```bash
brew install postgresql@16
brew services start postgresql@16
```

### Connect to PostgreSQL

```bash
# Switch to postgres superuser
sudo -i -u postgres

# Enter the psql shell
psql

# OR connect directly (shorthand)
sudo -u postgres psql
```

### Create Your Own Superuser (Recommended)

```sql
-- Inside psql as postgres user
CREATE USER umar WITH SUPERUSER PASSWORD 'securepassword';
CREATE DATABASE umar_dev OWNER umar;

-- Exit and reconnect as your user
\q
psql -U umar -d umar_dev
```

---

## 3. Core Database Concepts

### The Big Picture

```
PostgreSQL Server
│
├── Database: umar_dev
│   ├── Schema: public         ← default schema
│   │   ├── Table: users
│   │   ├── Table: orders
│   │   └── View: active_users
│   └── Schema: analytics
│       └── Table: events
│
└── Database: umar_test
```

### Key Terms You Must Know

| Term               | What it means                                    |
| ------------------ | ------------------------------------------------ |
| **Schema**         | A namespace inside a database (like a folder)    |
| **Table**          | Rows and columns of data                         |
| **Row / Record**   | A single entry in a table                        |
| **Column / Field** | A named attribute of the table                   |
| **Primary Key**    | Uniquely identifies each row                     |
| **Foreign Key**    | Links one table to another                       |
| **Index**          | Speeds up data lookup                            |
| **Transaction**    | A group of queries that succeed or fail together |
| **Constraint**     | A rule that enforces data validity               |

---

## 4. Data Types Deep Dive

Choosing the right data type is the first sign of a senior engineer.

### Numeric Types

```sql
SMALLINT        -- -32,768 to 32,767          (2 bytes)
INTEGER / INT   -- -2.1B to 2.1B              (4 bytes)  ← use for most IDs
BIGINT          -- ±9.2 quintillion           (8 bytes)  ← use for large-scale IDs
DECIMAL(10, 2)  -- Exact: 10 digits, 2 after decimal    ← use for MONEY
NUMERIC(p, s)   -- Same as DECIMAL
REAL            -- Float 6 decimal digits     (4 bytes)
DOUBLE PRECISION-- Float 15 decimal digits    (8 bytes)
SERIAL          -- Auto-increment INTEGER (legacy)
BIGSERIAL       -- Auto-increment BIGINT (legacy)
```

> ⚠️ **Never store money as FLOAT.** Floating-point math is imprecise. Always use `DECIMAL(19, 4)` or `NUMERIC` for financial data.

### Text Types

```sql
CHAR(n)         -- Fixed length, padded with spaces (rarely useful)
VARCHAR(n)      -- Variable length, up to n characters
TEXT            -- Unlimited length string ← prefer this in PostgreSQL
```

> ✅ In PostgreSQL, `TEXT` and `VARCHAR` have **identical performance**. Just use `TEXT` unless you need to enforce a length limit.

### Date & Time Types

```sql
DATE            -- '2024-04-17'
TIME            -- '14:30:00'
TIMESTAMP       -- '2024-04-17 14:30:00'           (no timezone)
TIMESTAMPTZ     -- '2024-04-17 14:30:00+05:00'    ← ALWAYS use this
INTERVAL        -- '3 days', '2 hours 30 minutes'
```

> 🕐 **Always use `TIMESTAMPTZ`** (timestamp with time zone). It stores UTC internally and converts to local timezone on display — essential for any app with users across time zones.

### Boolean

```sql
BOOLEAN         -- TRUE / FALSE / NULL
```

### Other Important Types

```sql
UUID            -- 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11'
JSON            -- Stored as text, validated as JSON
JSONB           -- Stored as binary ← FASTER, use this
ARRAY           -- INTEGER[], TEXT[], etc.
ENUM            -- Custom type with fixed values
INET            -- IP address '192.168.1.1'
CIDR            -- IP network '192.168.1.0/24'
```

---

## 5. Table Design & Constraints

### Create a Production-Ready Table

```sql
CREATE TABLE users (
    id          BIGSERIAL       PRIMARY KEY,
    email       TEXT            NOT NULL UNIQUE,
    username    VARCHAR(50)     NOT NULL UNIQUE,
    full_name   TEXT,
    age         SMALLINT        CHECK (age >= 0 AND age <= 150),
    role        TEXT            NOT NULL DEFAULT 'user',
    is_active   BOOLEAN         NOT NULL DEFAULT TRUE,
    created_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);
```

### Constraints Reference

```sql
-- PRIMARY KEY: unique + not null, one per table
id BIGSERIAL PRIMARY KEY

-- NOT NULL: value must be provided
email TEXT NOT NULL

-- UNIQUE: no duplicates allowed
email TEXT UNIQUE

-- CHECK: custom validation rule
age INT CHECK (age BETWEEN 0 AND 150)
price DECIMAL CHECK (price > 0)

-- DEFAULT: fallback value
is_active BOOLEAN DEFAULT TRUE

-- FOREIGN KEY: links to another table
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

-- Named constraints (best practice for large schemas)
CONSTRAINT chk_age CHECK (age >= 0),
CONSTRAINT uq_email UNIQUE (email)
```

### Foreign Key with Cascade Options

```sql
CREATE TABLE orders (
    id          BIGSERIAL PRIMARY KEY,
    user_id     BIGINT NOT NULL,
    total       DECIMAL(10, 2) NOT NULL,
    created_at  TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE     -- delete orders when user is deleted
        ON UPDATE CASCADE     -- update if user id changes
);
```

**ON DELETE options:**

| Option        | Behavior                           |
| ------------- | ---------------------------------- |
| `CASCADE`     | Delete child rows automatically    |
| `SET NULL`    | Set foreign key to NULL            |
| `SET DEFAULT` | Set to default value               |
| `RESTRICT`    | Prevent deletion if children exist |
| `NO ACTION`   | Same as RESTRICT (default)         |

### Alter Table (Modify After Creation)

```sql
-- Add a column
ALTER TABLE users ADD COLUMN phone TEXT;

-- Drop a column
ALTER TABLE users DROP COLUMN phone;

-- Rename a column
ALTER TABLE users RENAME COLUMN full_name TO name;

-- Change data type
ALTER TABLE users ALTER COLUMN age TYPE INTEGER;

-- Add a constraint
ALTER TABLE users ADD CONSTRAINT chk_role CHECK (role IN ('user', 'admin', 'moderator'));

-- Drop a constraint
ALTER TABLE users DROP CONSTRAINT chk_role;

-- Rename table
ALTER TABLE users RENAME TO app_users;
```

---

## 6. CRUD Operations

### INSERT

```sql
-- Single row
INSERT INTO users (email, username, full_name, age)
VALUES ('umar@gmail.com', 'umar_dev', 'Umar Khan', 22);


INSERT INTO users (name, email, role)
VALUES ('Zain', 'zain@gmail.com', 'user')
RETURNING *;

-- Multiple rows (much faster than individual inserts)
INSERT INTO users (email, username, age)
VALUES
    ('ali@gmail.com',   'ali_codes',  23),
    ('sara@gmail.com',  'sara_ux',    25),
    ('ahmed@gmail.com', 'ahmed_ai',   21);

-- INSERT and return the created row (super useful in apps)
INSERT INTO users (email, username)
VALUES ('new@gmail.com', 'new_user')
RETURNING id, email, created_at;

-- Upsert: insert or update on conflict
INSERT INTO users (email, username)
VALUES ('umar@gmail.com', 'umar_v2')
ON CONFLICT (email)
DO UPDATE SET username = EXCLUDED.username, updated_at = NOW();

-- Insert and do nothing on conflict
INSERT INTO users (email, username)
VALUES ('umar@gmail.com', 'umar_v2')
ON CONFLICT (email) DO NOTHING;
```

### SELECT

```sql
-- All rows and columns
SELECT * FROM users;

-- Specific columns (always prefer this in production — never use * in app code)
SELECT id, email, username, created_at FROM users;

-- Alias columns
SELECT
    id,
    email,
    full_name   AS name,
    created_at  AS "Member Since"
FROM users;

-- Computed columns
SELECT
    id,
    full_name,
    age,
    age * 365 AS age_in_days
FROM users;

-- Remove duplicates
SELECT DISTINCT role FROM users;
```

### UPDATE

```sql
-- Update one field
UPDATE users
SET age = 23
WHERE id = 1;

-- Update multiple fields
UPDATE users
SET
    full_name  = 'Umar Khan Updated',
    age        = 24,
    updated_at = NOW()
WHERE email = 'umar@gmail.com';

-- Update with RETURNING
UPDATE users
SET is_active = FALSE
WHERE id = 5
RETURNING id, email, is_active;

-- Update based on another table (UPDATE...FROM)
UPDATE orders o
SET total = o.total * 0.9  -- 10% discount
FROM users u
WHERE o.user_id = u.id
  AND u.role = 'premium';
```

### DELETE

```sql
-- Delete specific rows
DELETE FROM users WHERE id = 1;

-- Delete with condition
DELETE FROM users WHERE is_active = FALSE;

-- Delete and return deleted rows
DELETE FROM users WHERE id = 99 RETURNING *;

-- Delete all rows (but keep table structure)
DELETE FROM users;

-- TRUNCATE: much faster than DELETE for wiping a whole table
TRUNCATE TABLE users;                        -- fast, no row-by-row logging
TRUNCATE TABLE users RESTART IDENTITY;      -- also resets SERIAL counter
TRUNCATE TABLE users, orders CASCADE;       -- also truncates related tables
```

> ⚠️ `TRUNCATE` cannot be rolled back in some configurations. Use with care.

---

## 7. Filtering, Sorting & Pagination

### WHERE Clause

```sql
-- Comparison operators
WHERE age = 22
WHERE age != 22     -- or <>
WHERE age > 18
WHERE age >= 18
WHERE age < 60
WHERE age <= 60

-- Multiple conditions
WHERE age > 18 AND role = 'admin'
WHERE age < 18 OR role = 'admin'
WHERE NOT is_active

-- NULL checks (never use = NULL)
WHERE phone IS NULL
WHERE phone IS NOT NULL

-- Range
WHERE age BETWEEN 18 AND 30   -- inclusive on both ends

-- List membership
WHERE role IN ('admin', 'moderator')
WHERE role NOT IN ('banned', 'deleted')

-- Pattern matching
WHERE email LIKE '%@gmail.com'    -- case-sensitive
WHERE email ILIKE '%@GMAIL.COM'   -- case-insensitive (PostgreSQL-specific, very useful)
WHERE username LIKE 'um%'         -- starts with 'um'
WHERE username LIKE '%dev'        -- ends with 'dev'
WHERE username LIKE '%_ar%'       -- has any char then 'ar'
```

### ORDER BY & LIMIT

```sql
-- Single column sort
SELECT * FROM users ORDER BY age ASC;     -- ascending (default)
SELECT * FROM users ORDER BY age DESC;    -- descending

-- Multiple columns
SELECT * FROM users ORDER BY role ASC, created_at DESC;

-- NULLs go last (default is NULLS LAST for DESC, NULLS FIRST for ASC)
SELECT * FROM users ORDER BY age ASC NULLS LAST;

-- Pagination (page 1 = OFFSET 0, page 2 = OFFSET 10, etc.)
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 10
OFFSET 0;    -- page 1

SELECT * FROM users
ORDER BY created_at DESC
LIMIT 10
OFFSET 10;   -- page 2



SELECT * FROM users LIMIT 5 OFFSET 5;
Meaning:

👉 Skip first 5 users
👉 Then show next 5 users

```

> 🔥 **Senior Tip:** `OFFSET` pagination becomes slow on large tables. For high-performance pagination, use **keyset pagination** (cursor-based):

```sql
-- Cursor-based pagination: much faster for large datasets
-- First page
SELECT * FROM users ORDER BY id ASC LIMIT 10;

-- Next page: use last seen id as cursor
SELECT * FROM users
WHERE id > 10      -- last id from previous page
ORDER BY id ASC
LIMIT 10;
```

---

## 8. Aggregate Functions & Grouping

### Built-in Aggregate Functions

```sql
SELECT COUNT(*)             FROM users;                  -- total rows
SELECT COUNT(phone)         FROM users;                  -- non-null phones only
SELECT COUNT(DISTINCT role) FROM users;                  -- unique roles

SELECT SUM(total)           FROM orders;                 -- sum
SELECT AVG(age)             FROM users;                  -- average
SELECT ROUND(AVG(age), 2)   FROM users;                  -- rounded average

SELECT MAX(age)             FROM users;                  -- highest value
SELECT MIN(age)             FROM users;                  -- lowest value

SELECT MAX(created_at)      FROM orders;                 -- most recent order
```

### GROUP BY

```sql
-- Count users per role
SELECT role, COUNT(*) AS user_count
FROM users
GROUP BY role
ORDER BY user_count DESC;

-- Revenue per user
SELECT
    user_id,
    COUNT(*)          AS order_count,
    SUM(total)        AS total_revenue,
    ROUND(AVG(total), 2) AS avg_order_value
FROM orders
GROUP BY user_id
ORDER BY total_revenue DESC;

-- Group by multiple columns
SELECT role, is_active, COUNT(*) AS count
FROM users
GROUP BY role, is_active;
```

### HAVING (filter after GROUP BY)

```sql
-- Only show roles with more than 5 users
SELECT role, COUNT(*) AS user_count
FROM users
GROUP BY role
HAVING COUNT(*) > 5;

-- Users who spent more than $1000 total
SELECT user_id, SUM(total) AS total_spent
FROM orders
GROUP BY user_id
HAVING SUM(total) > 1000
ORDER BY total_spent DESC;
```

> 💡 **Rule:** `WHERE` filters **before** grouping. `HAVING` filters **after** grouping.

---

## 9. JOINs — The Heart of Relational DBs

```
Table A         Table B
┌─────┐         ┌─────┐
│  1  │         │  1  │ ← INNER JOIN matches only these
│  2  │ ────────│  2  │
│  3  │         │  4  │
└─────┘         └─────┘
```

### Sample Tables

```sql
CREATE TABLE departments (
    id   SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE employees (
    id            SERIAL PRIMARY KEY,
    name          TEXT NOT NULL,
    department_id INT REFERENCES departments(id),
    salary        DECIMAL(10,2)
);
```

### INNER JOIN

Returns rows that have a **match in both tables**.

```sql
SELECT e.name, e.salary, d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

### LEFT JOIN (LEFT OUTER JOIN)

Returns **all rows from the left** table, plus matched rows from the right. Unmatched right rows are NULL.

```sql
-- Shows ALL employees, even those without a department
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

Returns **all rows from the right** table. Rarely used — you can always rewrite as LEFT JOIN.

```sql
-- Shows ALL departments, even those with no employees
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

### FULL OUTER JOIN

Returns **all rows from both tables**, with NULLs where there's no match.

```sql
SELECT e.name, d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

### CROSS JOIN

Returns the **Cartesian product** — every row combined with every other row.

```sql
SELECT e.name, d.name
FROM employees e
CROSS JOIN departments d;
-- If 5 employees × 3 departments = 15 rows
```

### SELF JOIN

Joining a table **to itself** — great for hierarchical data.

```sql
-- Employees and their managers (manager_id references the same table)
SELECT
    e.name          AS employee,
    m.name          AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### JOIN Best Practices

```sql
-- ✅ Always use table aliases for clarity
SELECT u.email, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;

-- ✅ Join on indexed columns (PKs and FKs are indexed by default)

-- ✅ Multiple JOINs
SELECT
    u.email,
    o.total,
    p.name AS product_name
FROM users u
JOIN orders o          ON u.id = o.user_id
JOIN order_items oi    ON o.id = oi.order_id
JOIN products p        ON oi.product_id = p.id
WHERE u.is_active = TRUE
ORDER BY o.created_at DESC;
```

---

## 10. Subqueries & CTEs

### Subqueries

A query nested inside another query.

```sql
-- Subquery in WHERE
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM (derived table)
SELECT dept_avg.department_id, dept_avg.avg_salary
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_avg
WHERE dept_avg.avg_salary > 50000;

-- EXISTS (often faster than IN for large datasets)
SELECT u.email
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id
    AND o.total > 500
);

-- NOT EXISTS
SELECT u.email
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

### CTEs (Common Table Expressions) — WITH clause

CTEs make complex queries readable and maintainable. This is a sign of a senior engineer.

```sql
-- Basic CTE
WITH active_users AS (
    SELECT id, email, full_name
    FROM users
    WHERE is_active = TRUE
)
SELECT * FROM active_users WHERE full_name LIKE 'A%';

-- Multiple CTEs
WITH
    active_users AS (
        SELECT id, email FROM users WHERE is_active = TRUE
    ),
    recent_orders AS (
        SELECT user_id, SUM(total) AS total_spent
        FROM orders
        WHERE created_at > NOW() - INTERVAL '30 days'
        GROUP BY user_id
    )
SELECT
    u.email,
    COALESCE(o.total_spent, 0) AS spent_last_30_days
FROM active_users u
LEFT JOIN recent_orders o ON u.id = o.user_id
ORDER BY spent_last_30_days DESC;

-- Recursive CTE (for hierarchical data like org charts, categories)
WITH RECURSIVE org_chart AS (
    -- Base case: top-level employees (no manager)
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: employees who report to someone in org_chart
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT level, name FROM org_chart ORDER BY level, name;
```

---

## 11. Indexes — Make It Blazing Fast

An index is a separate data structure that allows PostgreSQL to find rows without scanning the entire table.

### Types of Indexes

```sql
-- B-Tree (default, handles =, <, >, BETWEEN, LIKE 'prefix%')
CREATE INDEX idx_users_email       ON users(email);
CREATE INDEX idx_orders_user_id    ON orders(user_id);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);

-- Unique Index (enforces uniqueness + speeds up lookups)
CREATE UNIQUE INDEX uq_users_email ON users(email);

-- Composite Index (multi-column — order matters!)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);
-- ↑ Efficient for: WHERE user_id = ? ORDER BY created_at DESC

-- Partial Index (index only a subset of rows — very powerful)
CREATE INDEX idx_active_users ON users(email) WHERE is_active = TRUE;
-- ↑ Much smaller and faster if most queries filter by is_active

-- Expression Index (index a function result)
CREATE INDEX idx_lower_email ON users(LOWER(email));
-- ↑ Needed for: WHERE LOWER(email) = 'umar@gmail.com'

-- GIN Index (for JSONB, arrays, full-text search)
CREATE INDEX idx_user_data ON users USING GIN(metadata);

-- BRIN Index (for large tables with naturally ordered data like timestamps)
CREATE INDEX idx_events_time ON events USING BRIN(created_at);
```

### Check if Index is Being Used

```sql
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'umar@gmail.com';

-- Look for:
-- "Index Scan" ← index is being used ✅
-- "Seq Scan"   ← full table scan, index not used ⚠️
```

### When NOT to Add an Index

- Small tables (< 1000 rows) — full scan is faster
- Columns with very low cardinality (e.g., boolean, gender) — index won't help much
- Columns that are rarely queried
- Tables with very frequent writes (indexes slow down INSERT/UPDATE/DELETE)

### List & Drop Indexes

```sql
-- List all indexes on a table
\d users

-- Or query the catalog
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'users';

-- Drop an index
DROP INDEX idx_users_email;

-- Drop concurrently (no table lock — use on production)
DROP INDEX CONCURRENTLY idx_users_email;
```

---

## 12. Transactions & ACID

### ACID Properties

| Property        | Meaning                                              |
| --------------- | ---------------------------------------------------- |
| **Atomicity**   | All operations succeed, or none do                   |
| **Consistency** | Database always goes from one valid state to another |
| **Isolation**   | Concurrent transactions don't interfere              |
| **Durability**  | Committed data survives crashes                      |

### Basic Transaction

```sql
BEGIN;

    UPDATE accounts SET balance = balance - 500 WHERE id = 1;
    UPDATE accounts SET balance = balance + 500 WHERE id = 2;

    -- Check for errors here in application code
    -- If all good:
COMMIT;

    -- If something went wrong:
ROLLBACK;
```

### Savepoints (Partial Rollback)

```sql
BEGIN;

    INSERT INTO orders (user_id, total) VALUES (1, 100);

    SAVEPOINT after_order;   -- create a checkpoint

    INSERT INTO order_items (order_id, product_id) VALUES (99, 5);
    -- Oops, product 5 doesn't exist!

    ROLLBACK TO SAVEPOINT after_order;   -- undo only to checkpoint

    INSERT INTO order_items (order_id, product_id) VALUES (99, 3);

COMMIT;
```

### Transaction Isolation Levels

```sql
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;    -- default
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;   -- consistent snapshot
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;      -- strongest, slowest
```

---

## 13. Views

A view is a saved query that acts like a virtual table.

```sql
-- Create a view
CREATE VIEW active_users_view AS
SELECT id, email, full_name, created_at
FROM users
WHERE is_active = TRUE;

-- Use it like a table
SELECT * FROM active_users_view;

-- Updatable view (simple views can be updated)
UPDATE active_users_view SET full_name = 'New Name' WHERE id = 1;

-- Replace a view (update its definition)
CREATE OR REPLACE VIEW active_users_view AS
SELECT id, email, full_name, role, created_at
FROM users
WHERE is_active = TRUE AND role != 'banned';

-- Drop a view
DROP VIEW active_users_view;

-- Materialized View (stores results physically — great for expensive queries)
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT
    DATE_TRUNC('month', created_at) AS month,
    SUM(total) AS revenue,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_TRUNC('month', created_at);

-- Manually refresh materialized view
REFRESH MATERIALIZED VIEW monthly_sales;

-- Refresh without locking (concurrent reads still work)
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales;
```

---

## 14. Stored Procedures & Functions

### Functions

```sql
-- Simple function: calculate age from birthday
CREATE OR REPLACE FUNCTION calculate_age(birthday DATE)
RETURNS INTEGER AS $$
BEGIN
    RETURN DATE_PART('year', AGE(birthday));
END;
$$ LANGUAGE plpgsql;

-- Use it
SELECT calculate_age('2002-03-15');
SELECT name, calculate_age(birthday) AS age FROM users;

-- Function that returns a table
CREATE OR REPLACE FUNCTION get_user_orders(p_user_id BIGINT)
RETURNS TABLE(order_id BIGINT, total DECIMAL, created_at TIMESTAMPTZ) AS $$
BEGIN
    RETURN QUERY
    SELECT id, total, created_at
    FROM orders
    WHERE user_id = p_user_id
    ORDER BY created_at DESC;
END;
$$ LANGUAGE plpgsql;

-- Use it
SELECT * FROM get_user_orders(1);
```

### Stored Procedures (PostgreSQL 11+)

```sql
-- Procedures can use transactions; functions cannot
CREATE OR REPLACE PROCEDURE transfer_money(
    sender_id   BIGINT,
    receiver_id BIGINT,
    amount      DECIMAL
)
LANGUAGE plpgsql AS $$
BEGIN
    UPDATE accounts SET balance = balance - amount WHERE id = sender_id;
    UPDATE accounts SET balance = balance + amount WHERE id = receiver_id;
    COMMIT;
END;
$$;

-- Call it
CALL transfer_money(1, 2, 500.00);
```

---

## 15. Triggers

A trigger automatically fires when a specified event occurs on a table.

```sql
-- Step 1: Create the trigger function
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Step 2: Attach trigger to a table
CREATE TRIGGER trigger_users_updated_at
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION set_updated_at();

-- Now every UPDATE on users automatically sets updated_at!

-- Audit trigger: log all changes
CREATE TABLE audit_log (
    id          BIGSERIAL PRIMARY KEY,
    table_name  TEXT,
    operation   TEXT,
    old_data    JSONB,
    new_data    JSONB,
    changed_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_data, new_data)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP = 'DELETE' THEN row_to_json(OLD)::JSONB ELSE NULL END,
        CASE WHEN TG_OP != 'DELETE' THEN row_to_json(NEW)::JSONB ELSE NULL END
    );
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_users
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();
```

---

## 16. JSON & JSONB Support

PostgreSQL's JSONB support is one of its killer features — it lets you store flexible schema data while still querying it efficiently.

```sql
-- Create table with JSONB column
CREATE TABLE products (
    id       BIGSERIAL PRIMARY KEY,
    name     TEXT NOT NULL,
    metadata JSONB          -- flexible attributes
);

-- Insert JSON data
INSERT INTO products (name, metadata)
VALUES (
    'Gaming Laptop',
    '{"brand": "ASUS", "specs": {"ram": 32, "cpu": "i9"}, "tags": ["gaming", "portable"]}'
);

-- Query JSON fields
SELECT metadata->>'brand'           AS brand FROM products;  -- returns TEXT
SELECT metadata->'specs'            AS specs FROM products;  -- returns JSONB
SELECT metadata->'specs'->>'ram'    AS ram   FROM products;  -- returns TEXT
SELECT (metadata->'specs'->>'ram')::INT AS ram_gb FROM products; -- cast to INT

-- Filter by JSON field
SELECT * FROM products WHERE metadata->>'brand' = 'ASUS';
SELECT * FROM products WHERE (metadata->'specs'->>'ram')::INT >= 16;

-- Check if key exists
SELECT * FROM products WHERE metadata ? 'brand';

-- Check if array contains value
SELECT * FROM products WHERE metadata->'tags' ? 'gaming';

-- Update a JSON field
UPDATE products
SET metadata = jsonb_set(metadata, '{specs, ram}', '64')
WHERE id = 1;

-- Add a new JSON key
UPDATE products
SET metadata = metadata || '{"color": "black"}'
WHERE id = 1;

-- Remove a JSON key
UPDATE products
SET metadata = metadata - 'color'
WHERE id = 1;

-- Index JSONB for fast querying (GIN index)
CREATE INDEX idx_products_metadata ON products USING GIN(metadata);

-- Index a specific JSON path
CREATE INDEX idx_products_brand ON products((metadata->>'brand'));
```

---

## 17. Window Functions

Window functions perform calculations **across a set of rows related to the current row** — without collapsing rows like GROUP BY does.

```sql
-- Syntax
function_name() OVER (
    PARTITION BY column    -- group rows (optional)
    ORDER BY column        -- define order within window (optional)
    ROWS/RANGE frame       -- define frame (optional)
)
```

### ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT
    name,
    department_id,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS row_num,
    RANK()       OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dense_rank
FROM employees;

-- ROW_NUMBER: 1,2,3,4 (no ties, always unique)
-- RANK:       1,2,2,4 (ties get same rank, then skips)
-- DENSE_RANK: 1,2,2,3 (ties get same rank, no skips)
```

### LAG & LEAD (Access previous/next row)

```sql
SELECT
    created_at,
    total,
    LAG(total)  OVER (ORDER BY created_at) AS previous_order_total,
    LEAD(total) OVER (ORDER BY created_at) AS next_order_total,
    total - LAG(total) OVER (ORDER BY created_at) AS change_from_previous
FROM orders
WHERE user_id = 1;
```

### Running Totals & Moving Averages

```sql
SELECT
    created_at::DATE AS date,
    total,
    SUM(total)  OVER (ORDER BY created_at) AS running_total,
    AVG(total)  OVER (ORDER BY created_at ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7
FROM orders;
```

### NTILE (Percentile buckets)

```sql
-- Divide users into 4 salary quartiles
SELECT
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;
```

### FIRST_VALUE & LAST_VALUE

```sql
SELECT
    name,
    salary,
    FIRST_VALUE(salary) OVER (PARTITION BY department_id ORDER BY salary DESC) AS highest_in_dept,
    LAST_VALUE(salary)  OVER (
        PARTITION BY department_id ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_in_dept
FROM employees;
```

---

## 18. Performance & Query Optimization

### EXPLAIN & EXPLAIN ANALYZE

```sql
-- Show the query plan (no execution)
EXPLAIN SELECT * FROM users WHERE email = 'umar@gmail.com';

-- Execute AND show timing stats (use this in development)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'umar@gmail.com';

-- More detailed output
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders WHERE user_id = 1;
```

**Key things to look for in EXPLAIN output:**

| Node type          | Meaning                                                |
| ------------------ | ------------------------------------------------------ |
| `Seq Scan`         | Reading entire table — no index used                   |
| `Index Scan`       | Using an index (good)                                  |
| `Index Only Scan`  | Reading from index alone, no heap access (best)        |
| `Bitmap Heap Scan` | Batch index lookup                                     |
| `Hash Join`        | Hashing one table then probing (good for large tables) |
| `Nested Loop`      | For each row in A, look up matching rows in B          |
| `Sort`             | Sorting in memory                                      |
| `cost=X..Y`        | X = startup cost, Y = total cost                       |
| `rows=N`           | Estimated row count                                    |
| `actual time=X..Y` | Actual execution time in ms                            |

### pg_stat_statements (find slow queries)

```sql
-- Enable the extension (in postgresql.conf or superuser)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Find the slowest queries
SELECT
    query,
    calls,
    ROUND(total_exec_time::NUMERIC, 2) AS total_ms,
    ROUND(mean_exec_time::NUMERIC, 2)  AS avg_ms,
    rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;
```

### Useful Optimization Tips

```sql
-- ✅ Use EXISTS instead of COUNT when you just need to check if rows exist
-- Bad:
SELECT * FROM users WHERE (SELECT COUNT(*) FROM orders WHERE user_id = users.id) > 0;
-- Good:
SELECT * FROM users WHERE EXISTS (SELECT 1 FROM orders WHERE user_id = users.id);

-- ✅ Avoid SELECT * in production code — fetch only needed columns
-- Bad:  SELECT * FROM users WHERE id = 1
-- Good: SELECT id, email, full_name FROM users WHERE id = 1

-- ✅ Use LIMIT early in CTEs to reduce rows flowing through the pipeline

-- ✅ Avoid functions on indexed columns in WHERE (breaks index usage)
-- Bad:  WHERE UPPER(email) = 'UMAR@GMAIL.COM'
-- Good: Create index on expression, then: WHERE email = LOWER('UMAR@GMAIL.COM')

-- ✅ VACUUM and ANALYZE keep statistics fresh (auto-vacuum handles this usually)
VACUUM ANALYZE users;
ANALYZE orders;

-- ✅ Use connection pooling (PgBouncer) in production — never open raw connections per request
```

---

## 19. Roles, Users & Security

```sql
-- Create a role (group of permissions)
CREATE ROLE readonly;

-- Grant permissions to role
GRANT CONNECT ON DATABASE umar_dev TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

-- Create a user (login role)
CREATE USER analyst_user WITH PASSWORD 'securepass123';

-- Assign role to user
GRANT readonly TO analyst_user;

-- Create an app user with limited permissions
CREATE USER app_user WITH PASSWORD 'apppass';
GRANT CONNECT ON DATABASE umar_dev TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- Revoke permissions
REVOKE INSERT ON users FROM app_user;

-- List all roles
\du

-- Drop a user
DROP USER analyst_user;

-- Row Level Security (RLS) — users only see their own rows
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY orders_isolation ON orders
    USING (user_id = current_user_id());  -- custom function returning current user's id
```

---

## 20. Backup & Restore

```bash
# Backup a single database (custom format, compressed)
pg_dump -U postgres -d umar_dev -F c -f umar_dev_backup.dump

# Backup as plain SQL
pg_dump -U postgres -d umar_dev -F p -f umar_dev_backup.sql

# Backup all databases
pg_dumpall -U postgres -f all_databases.sql

# Restore from custom format
pg_restore -U postgres -d umar_dev_new -F c umar_dev_backup.dump

# Restore from SQL file
psql -U postgres -d umar_dev_new < umar_dev_backup.sql

# Restore with verbose output
pg_restore -U postgres -d umar_dev -F c --verbose umar_dev_backup.dump
```

---

## 21. psql Cheat Sheet

### Connection & Navigation

```sql
\l                   -- list all databases
\c dbname            -- connect to a database
\dn                  -- list schemas
\dt                  -- list tables in current schema
\dt schema.*         -- list tables in a specific schema
\dv                  -- list views
\df                  -- list functions
\di                  -- list indexes
\du                  -- list users/roles
\q                   -- quit

\d  table_name       -- describe table (columns, constraints, indexes)
\d+ table_name       -- verbose describe (includes storage details)
```

### Productivity Features

```sql
\timing              -- toggle query execution time display
\e                   -- open last query in $EDITOR (vim/nano)
\i file.sql          -- run a SQL file
\o output.txt        -- redirect output to a file
\! command           -- run a shell command
\set var value       -- set a psql variable
\echo :var           -- print a variable

-- Turn on expanded output (great for wide tables)
\x

-- Set pager
\pset pager off      -- turn off pager
```

### Search & History

```bash
# In psql:
Ctrl+R        # reverse search command history
\s            # show command history
\s file.txt   # save history to file
```

---

## 22. Senior-Level Tips & Patterns

### 1. Always Use TIMESTAMPTZ, Never TIMESTAMP

```sql
-- ❌ Timestamp without timezone — ambiguous!
created_at TIMESTAMP DEFAULT NOW()

-- ✅ Always timezone-aware
created_at TIMESTAMPTZ DEFAULT NOW()
```

### 2. UUID as Primary Keys for Distributed Systems

```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT NOT NULL UNIQUE
);

-- Or use the newer built-in (PostgreSQL 13+)
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
```

### 3. Soft Deletes Over Hard Deletes

```sql
-- Instead of DELETE, set deleted_at
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;

-- "Delete" a user
UPDATE users SET deleted_at = NOW() WHERE id = 1;

-- Query only active users
SELECT * FROM users WHERE deleted_at IS NULL;

-- Create a partial index for performance
CREATE INDEX idx_users_not_deleted ON users(id) WHERE deleted_at IS NULL;
```

### 4. Use generate_series for Test Data

```sql
-- Generate 10,000 test users
INSERT INTO users (email, username)
SELECT
    'user' || i || '@test.com',
    'user_' || i
FROM generate_series(1, 10000) AS s(i);

-- Generate date ranges
SELECT generate_series(
    '2024-01-01'::DATE,
    '2024-12-31'::DATE,
    '1 month'::INTERVAL
) AS month;
```

### 5. Common Table Statistics

```sql
-- Table sizes
SELECT
    relname AS table,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
    pg_size_pretty(pg_relation_size(relid))        AS table_size,
    pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) AS indexes_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Row counts (approximate, very fast)
SELECT relname, n_live_tup AS row_count
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;

-- Index usage stats
SELECT
    indexrelname AS index,
    idx_scan AS times_used,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Unused indexes (expensive to maintain, never used!)
SELECT indexrelname FROM pg_stat_user_indexes WHERE idx_scan = 0;
```

### 6. Connection Info

```sql
-- Show current connections
SELECT pid, usename, application_name, client_addr, state, query
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_start;

-- Kill a connection
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;
```

### 7. Schema Migration Pattern (Production-Safe)

```sql
-- ✅ Adding a nullable column is safe (no lock)
ALTER TABLE users ADD COLUMN last_login TIMESTAMPTZ;

-- ✅ Adding a column with default (PostgreSQL 11+ is safe)
ALTER TABLE users ADD COLUMN theme TEXT DEFAULT 'light';

-- ⚠️ Adding NOT NULL constraint requires careful approach on large tables:
-- Step 1: Add column as nullable
ALTER TABLE users ADD COLUMN verified BOOLEAN;
-- Step 2: Backfill data
UPDATE users SET verified = FALSE WHERE verified IS NULL;
-- Step 3: Add constraint with NOT VALID (no full table scan)
ALTER TABLE users ADD CONSTRAINT chk_verified_not_null CHECK (verified IS NOT NULL) NOT VALID;
-- Step 4: Validate (background, doesn't block reads/writes)
ALTER TABLE users VALIDATE CONSTRAINT chk_verified_not_null;
-- Step 5: Add actual NOT NULL (now instant because constraint already validated)
ALTER TABLE users ALTER COLUMN verified SET NOT NULL;
ALTER TABLE users DROP CONSTRAINT chk_verified_not_null;
```

---

## 🎓 Final Words

You've now got everything from zero to production-grade PostgreSQL in your arsenal. Here's what separates juniors from seniors:

| Junior                       | Senior                         |
| ---------------------------- | ------------------------------ |
| Uses `SELECT *`              | Selects only needed columns    |
| Adds indexes randomly        | Profiles first, then indexes   |
| Stores money as FLOAT        | Uses `DECIMAL(19, 4)`          |
| Uses TIMESTAMP               | Uses TIMESTAMPTZ               |
| Runs migrations without care | Plans zero-downtime migrations |
| Writes complex subqueries    | Uses CTEs for readability      |
| Hard-deletes records         | Soft-deletes with `deleted_at` |
| Ignores EXPLAIN              | Lives in EXPLAIN ANALYZE       |

> 🐘 **PostgreSQL rewards the patient and punishes the lazy.** Learn it deeply once, and it pays you back for years.

---

_Prepared for senior full-stack engineers who refuse to settle for "good enough."_
