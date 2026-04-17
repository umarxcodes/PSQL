# 🐘 PostgreSQL Full Notes (Basic → Senior Level)

---

# 📌 1. Introduction

PostgreSQL is an advanced open-source relational database management system (RDBMS) used to store, manage, and query structured data using SQL.

---

# 🧠 2. Basic Concepts

## Database

A database is a collection of organized data.

## Table

A table stores data in rows and columns.

Example:

| id  | name | age |
| --- | ---- | --- |
| 1   | Umar | 22  |

---

# ⚙️ 3. Installation (Ubuntu)

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

Start service:

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

Login:

```bash
sudo -i -u postgres
psql
```

---

# 🗄️ 4. Database Commands

Create database:

```sql
CREATE DATABASE umar_db;
```

Connect database:

```sql
\c umar_db
```

List databases:

```sql
\l
```

---

# 📊 5. Table Creation

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    age INT,
    course TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Describe table:

```sql
\d students
```

---

# ➕ 6. INSERT (Add Data)

Single insert:

```sql
INSERT INTO students (name, age, course)
VALUES ('Umar', 22, 'CS');
```

Multiple insert:

```sql
INSERT INTO students (name, age, course)
VALUES
('Ali', 23, 'SE'),
('Ahmed', 21, 'AI');
```

---

# 🔍 7. SELECT (Read Data)

All data:

```sql
SELECT * FROM students;
```

Specific columns:

```sql
SELECT name, age FROM students;
```

---

# 🎯 8. WHERE (Filtering)

```sql
SELECT * FROM students WHERE age > 20;
SELECT * FROM students WHERE course = 'CS';
```

---

# 🔀 9. ORDER BY (Sorting)

```sql
SELECT * FROM students ORDER BY age ASC;
SELECT * FROM students ORDER BY age DESC;
```

---

# 📉 10. LIMIT

```sql
SELECT * FROM students LIMIT 5;
```

---

# 🔎 11. LIKE (Search)

```sql
SELECT * FROM students WHERE name LIKE 'A%';
SELECT * FROM students WHERE course LIKE '%Science%';
```

---

# 🔢 12. IN Operator

```sql
SELECT * FROM students WHERE age IN (20, 21, 22);
```

---

# 📏 13. BETWEEN

```sql
SELECT * FROM students WHERE age BETWEEN 18 AND 25;
```

---

# 📊 14. Aggregation

Count:

```sql
SELECT COUNT(*) FROM students;
```

Max:

```sql
SELECT MAX(age) FROM students;
```

Min:

```sql
SELECT MIN(age) FROM students;
```

Average:

```sql
SELECT AVG(age) FROM students;
```

---

# 📦 15. GROUP BY

```sql
SELECT course, COUNT(*)
FROM students
GROUP BY course;
```

---

# 🚨 16. HAVING

```sql
SELECT course, COUNT(*)
FROM students
GROUP BY course
HAVING COUNT(*) > 1;
```

---

# ✏️ 17. UPDATE

```sql
UPDATE students
SET age = 25
WHERE name = 'Umar';
```

---

# 🗑️ 18. DELETE

```sql
DELETE FROM students
WHERE name = 'Ali';
```

---

# 🔐 19. Constraints

```sql
PRIMARY KEY
NOT NULL
UNIQUE
CHECK (age > 0)
FOREIGN KEY
```

Example:

```sql
age INT CHECK (age > 0)
```

---

# 🔗 20. JOINs (VERY IMPORTANT)

## INNER JOIN

```sql
SELECT *
FROM students
INNER JOIN courses
ON students.course_id = courses.id;
```

## LEFT JOIN

```sql
SELECT *
FROM students
LEFT JOIN courses
ON students.course_id = courses.id;
```

---

# ⚡ 21. Indexing

Create index:

```sql
CREATE INDEX idx_students_name ON students(name);
```

---

# 🧠 22. Transactions (ACID)

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

Rollback:

```sql
ROLLBACK;
```

---

# 🔥 23. JSON Support

```sql
SELECT data->>'name' FROM users;
```

---

# ⚙️ 24. Useful psql Commands

```sql
\l   -- list databases
\c   -- connect database
\dt  -- list tables
\d table_name -- describe table
\q   -- quit
```

---

# 🚀 25. Senior-Level Thinking

A senior engineer thinks about:

- Performance
- Index usage
- Query optimization
- Locking
- Scalability
- Data modeling

---

# 🧪 26. Practice Tasks

1. Create students table
2. Insert 5 records
3. Use WHERE filter
4. Use ORDER BY
5. Use GROUP BY

---

# 🏁 END

You now have a complete PostgreSQL learning reference from beginner → senior level.
