# 🎓 Lesson 1.3: SQL Basic — DDL — Instructor Guide

## Session Overview

| Item | Detail |
|------|--------|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Hands-On SQL in DbGate |
| **Prerequisites** | Lesson 1.2 (database design concepts); DbGate installed; `unit-1-3.db` downloaded |
| **Tools** | DuckDB + DbGate |

### Agenda

| Time | Section | Focus |
|------|---------|-------|
| 0:00 – 0:55 | Part 1: The Foundation | Database hierarchy; CREATE TABLE; data types; INSERT & ALTER |
| 0:55 – 1:00 | Break | — |
| 1:00 – 1:55 | Part 2: The Blueprint | Constraints (PK, FK, NOT NULL, CHECK, DEFAULT) |
| 1:55 – 2:00 | Break | — |
| 2:00 – 2:55 | Part 3: The Pipeline | COPY for bulk import; Indexes; Views; Export |
| 2:55 – 3:00 | Wrap-Up | Key Takeaways & Post-Class Assignment Briefing |

---

## 🏃 Part 1: The Foundation (55 min)

### 🎯 Learning Objective
Identify the database hierarchy (Database → Schema → Table → Column → Row) and write `CREATE TABLE` statements with appropriate data types.

### 📖 Theory Recap (10 min)

**Analogy:** A database is a filing cabinet. A schema is a drawer. A table is a folder. Columns are the labelled fields on the form. Rows are the filled-in forms.

| Level | SQL Object | Real-World Equivalent |
|-------|-----------|----------------------|
| Database | `DATABASE` | The entire filing cabinet |
| Schema | `SCHEMA` | A drawer (organises related tables) |
| Table | `TABLE` | A folder of records |
| Column | Column definition | A labelled field on a form |
| Row | `INSERT` statement | One filled-in form |

**Common Data Types:**
| Type | Use for | Example |
|------|---------|---------|
| `INTEGER` | Whole numbers | Age, quantity, year |
| `VARCHAR(n)` | Variable-length text | Name, email (max n chars) |
| `TEXT` | Long text | Description, notes |
| `DATE` | Calendar date | Birthday, purchase date |
| `BOOLEAN` | True/False | Is active, has subscription |
| `DECIMAL(p,s)` | Precise decimals | Price, tax rate |

### 🛠️ Hands-On Activity: "Build the Foundation" (35 min)

**Code-along in DbGate:**

1. Create a schema and first table:
```sql
CREATE SCHEMA lesson;

CREATE TABLE lesson.users (
    id        INTEGER PRIMARY KEY,
    name      VARCHAR(100),
    email     VARCHAR(200),
    joined_at DATE
);
```

2. Insert a few rows:
```sql
INSERT INTO lesson.users VALUES (1, 'Alice', 'alice@example.com', '2024-01-15');
INSERT INTO lesson.users VALUES (2, 'Bob', 'bob@example.com', '2024-02-20');
```

3. Use ALTER TABLE to add a column:
```sql
ALTER TABLE lesson.users ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
```

4. Discuss: `TRUNCATE` (empty table, keep structure) vs. `DROP` (delete everything).

**Discussion Questions:**
- "What happens if you try to insert a second row with the same `id`?"
- "Why use `VARCHAR(100)` instead of `TEXT` for a name field?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "`DROP TABLE` and `TRUNCATE TABLE` do the same thing." → `TRUNCATE` removes all rows but keeps the table structure. `DROP` removes the table entirely — it's irreversible.
- **Business Case:** Every time a user signs up for a SaaS product, an `INSERT` statement like the one you just wrote runs in the background. Database schemas built badly at launch (wrong types, missing constraints) are extremely expensive to fix at scale.

---

## 🏃 Part 2: The Blueprint — Constraints (55 min)

### 🎯 Learning Objective
Apply NOT NULL, UNIQUE, CHECK, DEFAULT, Primary Key, and Foreign Key constraints to enforce data integrity.

### 📖 Theory Recap (10 min)

**Analogy:** Constraints are bouncers at the door of your database. They check every row trying to enter and reject anything that breaks the rules — before it causes chaos inside.

| Constraint | What it enforces | What it rejects |
|-----------|-----------------|----------------|
| `PRIMARY KEY` | Unique, non-null identity per row | Duplicates and nulls |
| `FOREIGN KEY` | References a valid PK in another table | Orphaned records |
| `NOT NULL` | Column must have a value | Empty submissions |
| `UNIQUE` | No two rows can have the same value | Duplicate emails, usernames |
| `CHECK` | Value must satisfy a condition | Negative prices, invalid ages |
| `DEFAULT` | Fallback value if none provided | — |

### 🛠️ Code-Along: "The School Database" (35 min)

Build a school database with proper constraints:

```sql
CREATE TABLE lesson.teachers (
    id       INTEGER PRIMARY KEY,
    name     VARCHAR(100) NOT NULL,
    subject  VARCHAR(50),
    salary   DECIMAL(10,2) CHECK (salary > 0)
);

CREATE TABLE lesson.classes (
    id         INTEGER PRIMARY KEY,
    name       VARCHAR(50) NOT NULL,
    teacher_id INTEGER REFERENCES lesson.teachers(id),
    room       VARCHAR(10) DEFAULT 'TBC'
);

CREATE TABLE lesson.students (
    id       INTEGER PRIMARY KEY,
    name     VARCHAR(100) NOT NULL,
    email    VARCHAR(200) UNIQUE,
    class_id INTEGER REFERENCES lesson.classes(id)
);
```

**Student challenge (15 min):** Complete the `students` table independently using the ERD as reference, then test by inserting valid and invalid rows.

**Discussion Questions:**
- "What happens if you try to `INSERT` a student with a `class_id` that doesn't exist?"
- "Why might you use `DEFAULT 'TBC'` for the room field?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "I can add constraints later with ALTER TABLE." → Yes, but it's harder and riskier if existing data already violates the constraint. Design constraints in from the start.
- **Business Case:** A European airline discovered that 12% of their passenger records had no email addresses because the field was nullable — causing failed check-in notifications. Adding `NOT NULL` at launch would have prevented this.

---

## 🏃 Part 3: The Pipeline (55 min)

### 🎯 Learning Objective
Execute `COPY` for bulk data import, create Indexes for performance, and build Views to simplify analyst access.

### 📖 Theory Recap (10 min)

**The COPY command is the industrial vacuum cleaner of SQL** — instead of inserting rows one by one, it inhales an entire CSV file in seconds.

**Indexes:** Like a book's index — instead of reading every page (full table scan), the database jumps directly to the relevant rows. Dramatically speeds up `WHERE` and `JOIN` queries on large tables.

**Views:** A saved query that behaves like a virtual table. The data isn't copied — the view re-runs the query every time you access it. Perfect for giving analysts a simplified, pre-filtered view of complex tables.

| Object | Purpose | Lives in DB? |
|--------|---------|-------------|
| Table | Stores actual data | ✅ Yes |
| View | Saved query, virtual table | ✅ Yes (but no data stored) |
| Index | Speed lookup structure | ✅ Yes |

### 🛠️ Hands-On Activity: "The Data Pipeline" (35 min)

1. **Bulk import from CSV:**
```sql
-- Note: Use the full path to your CSV file
COPY lesson.users FROM '/full/path/to/users.csv' (HEADER, DELIMITER ',');
```

2. **Update existing records:**
```sql
UPDATE lesson.users SET is_active = FALSE WHERE joined_at < '2023-01-01';
```

3. **Create an index for faster lookups:**
```sql
CREATE INDEX idx_users_email ON lesson.users(email);
```

4. **Create a view for analysts:**
```sql
CREATE VIEW lesson.active_users AS
SELECT id, name, email FROM lesson.users WHERE is_active = TRUE;

SELECT * FROM lesson.active_users;
```

5. **Export to CSV:**
```sql
COPY lesson.active_users TO '/full/path/to/active_users_export.csv' (HEADER, DELIMITER ',');
```

**Discussion Questions:**
- "Why use a View instead of just sharing the SELECT query?"
- "What happens to a View if you drop the underlying table?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "Views are faster than queries because they're pre-computed." → Standard views re-run the query every time. Only *materialised views* store the result. Performance depends on the underlying query complexity.
- **Business Case:** Data teams at companies like LinkedIn create Views to give different departments (HR, Finance, Marketing) access to pre-filtered, permission-appropriate subsets of the same underlying database — without giving them access to raw sensitive data.

---

## 🎯 Wrap-Up (5 min)

### Key Takeaways
1. **DDL is the skeleton of your database** — `CREATE`, `ALTER`, and `DROP` define the structure that everything else depends on.
2. **Constraints prevent bad data from entering** — it's always cheaper to reject invalid data at entry than to clean it up later.
3. **COPY, Indexes, and Views are your production tools** — bulk loading, performance, and access control are what separate a prototype from a real database.

### Next Steps
- **Post-Class:** Complete the [assignment.md](./assignment.md) challenges — The Tiny Library (schema design) and The Marketing Dump (messy data import) (45–60 min).
- **Next Lesson:** Lesson 1.4 builds on today's structure by teaching you to *query* data with DML — SELECT, WHERE, GROUP BY, and more.
