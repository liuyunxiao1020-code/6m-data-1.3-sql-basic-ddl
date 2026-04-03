# 📚 Lesson 1.3: SQL Basic — DDL with DuckDB

**In data science, data is the new oil. DDL is how we build the refinery.**

## Session Overview

| | |
|---|---|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Code-Along ("I do, We do, You do") |
| **Tools** | [DbGate](https://www.dbgate.io/download-community/), [DuckDB](https://duckdb.org/) |
| **Database file** | `db/unit-1-3.db` |

## Agenda

| Time | Part | Topic |
|------|------|-------|
| 0:00 – 0:55 | Part 1 | The Foundation — CREATE TABLE, data types, INSERT, ALTER |
| 0:55 – 1:05 | Break | — |
| 1:05 – 2:00 | Part 2 | The Blueprint — Constraints (PK, FK, NOT NULL, CHECK) |
| 2:00 – 2:10 | Break | — |
| 2:10 – 3:00 | Part 3 | The Pipeline — COPY, Indexes, Views, Export |

## 🎯 Learning Objectives

By the end of this session, you will be able to:

1. **Identify** the hierarchical structure of a relational database (Database > Schema > Table) to organize data effectively.
2. **Apply** `CREATE` statements with constraints (Primary/Foreign Keys) to construct a relational schema that enforces data integrity.
3. **Execute** data pipeline commands (`COPY`) to import raw CSV data into structured tables for analysis.

---

## 🏃 Part 1: The Foundation (55 min)

### Concept Overview

**In Python, you create variables like `my_data = []`. But when you turn off the computer, that data vanishes. To make data specific and permanent, we need a Database.**

Think of it like a physical File Cabinet:
- **The Database:** The Cabinet itself.
- **The Schema:** A specific drawer (labeled 'HR' or 'Sales').
- **The Table:** A folder inside that drawer.
- **The Column:** The specific form fields in that folder (Name, Date, ID).

Today, we aren't just coding; we are building the cabinet.

**Types Matter:**
- **INTEGER:** Math-able numbers (e.g., Age, Count).
- **VARCHAR:** Text (e.g., Name, Email).
- **DATE:** Chronological points. *(Don't store dates as text!)*

We will be using two modern, lightweight tools:

1. **DuckDB:** A fast, "in-process" database engine. Unlike traditional databases (like MySQL), DuckDB doesn't require a complex server setup. It stores everything in a single file on your computer, making it perfect for data science.
2. **DbGate:** A user-friendly database manager. It provides a visual interface to see your tables, run queries, and manage your data.

### 🛠️ Activity 1: The "Profile" Table (Code-Along — 20 min)

**1. Connecting to the Database:**

Open **DbGate** and create a new connection to the DuckDB file provided in this repo:

```
db/unit-1-3.db
```

> If you expand on the `unit-1-3.db`, you should see a few predefined schemas.
> - The default schema is `main`.
> - Any tables you create without specifying a schema will be in `main`.
> - You can also create additional schemas to organize your tables.

**2. Creating a Schema:**

```sql
CREATE SCHEMA lesson;
```

Or, to avoid errors if it already exists:

```sql
CREATE SCHEMA IF NOT EXISTS lesson;
```

**3. Creating your first Table:**

```sql
CREATE TABLE lesson.users (
  id INTEGER,
  name VARCHAR,
  email VARCHAR
);
```

> If you just want to create tables in the default (`main`) schema, you can omit the `lesson.` prefix.

**4. Basic Data Entry:**

```sql
INSERT INTO lesson.users (id, name, email)
VALUES (1, 'John Doe', 'john.doe@gmail.com');
```

Insert multiple rows at once:

```sql
INSERT INTO lesson.users (id, name, email)
VALUES (2, 'Jane Doe', 'jane.doe@gmail.com'),
       (3, 'John Smith', 'john.smith@gmail.com');
```

> Insert two more rows with contiguously increasing `id` values, random `name`s and `email`s.

**5. Alter Tables:**

Add a column:

```sql
ALTER TABLE lesson.users ADD COLUMN start_date DATE;
```

Rename a column:

```sql
ALTER TABLE lesson.users RENAME id TO uid;
```

Drop a column:

```sql
ALTER TABLE lesson.users
DROP COLUMN email;
```

**6. TRUNCATE vs. DROP:**

```sql
TRUNCATE TABLE lesson.users;  -- removes all rows, keeps table structure
```

```sql
DROP TABLE lesson.users;      -- removes the table entirely
```

> **Tip:** "Always run a SELECT before a DROP. Double-check your table name. It's better to be slow and correct than fast and sorry."

### 💬 Reflection

- Why is organizing data into schemas important in a production environment?
- What happens if you try to insert a second row with the same `id`?

---

## 🏃 Part 2: The Blueprint — Constraints (55 min)

### Concept Overview

"In Excel, you can type 'Banana' into a column meant for Prices. Excel doesn't care. Databases *do* care. **Constraints** are the 'Bouncers' at the door. If you try to enter a duplicate User ID? **Blocked.** If you try to enter a Class for a Teacher that doesn't exist? **Blocked.**"

**The Keys:**
- **Primary Key (PK):** The Fingerprint. Unique and Not Null.
- **Foreign Key (FK):** The Link. It points to a PK in another table.

<details>
  <summary>Primary Keys, Foreign Keys, NOT NULL, CHECK, and DEFAULT — Reference</summary>

  - `primary key` or `unique` define a column, or set of columns, that are a unique identifier for a row.
  - `primary key` constraints also enforce that keys cannot be NULL.
  - `foreign key` defines a column that refers to a primary key in another table — enforcing that the referenced record exists.
  - `not null` specifies that the column cannot contain any NULL values.
  - `default` specifies a default value for the column when no value is specified.
  - `check` allows you to specify an arbitrary boolean expression that all rows must satisfy.
</details>

### 🛠️ Activity 2: The "Missing Link" Challenge (25 min)

**Part A: Code-Along — Creating Tables with Constraints:**

```sql
CREATE TABLE lesson.teachers (
  id INTEGER PRIMARY KEY,       -- primary key
  name VARCHAR NOT NULL,        -- not null
  age INTEGER CHECK(age > 18 AND age < 70),    -- check
  address VARCHAR,
  phone VARCHAR,
  email VARCHAR CHECK(CONTAINS(email, '@'))    -- check
);

CREATE TABLE lesson.classes (
  id INTEGER PRIMARY KEY,       -- primary key
  name VARCHAR NOT NULL,        -- not null
  teacher_id INTEGER REFERENCES lesson.teachers(id)  -- foreign key
);
```

**Part B: Your Challenge**

Complete the `CREATE TABLE` statement for the `students` table based on the ERD below.

> In a school system whose classes have students and teachers. Each student belongs to a single class. Each teacher may teach more than one class, but each class only has one teacher.

```dbml
Table students {
  id int [pk]
  name varchar
  address varchar
  phone varchar
  email varchar
  class_id int
}

Table teachers {
  id int [pk]
  name varchar
  age int
  address varchar
  phone varchar
  email varchar
}

Table classes {
  id int [pk]
  name varchar
  teacher_id int
}

Ref: students.class_id > classes.id // many-to-one

Ref: classes.teacher_id > teachers.id // many-to-one
```

<details>
<summary>Solution</summary>

```sql
CREATE TABLE lesson.students (
    id INTEGER PRIMARY KEY,
    name VARCHAR,
    email VARCHAR,
    class_id INTEGER REFERENCES lesson.classes(id)
);
```

</details>

### 💬 Reflection

- What happens if you try to `INSERT` a student with a `class_id` that doesn't exist?
- How do constraints prevent "bad data" from entering your analysis pipeline?

---

## 🏃 Part 3: The Pipeline (50 min)

### Concept Overview

"We have empty tables. Typing `INSERT INTO...` one row at a time is fine for 10 rows. It is impossible for 1 million rows. The **COPY** command is the industrial vacuum cleaner of SQL — it inhales an entire CSV file in seconds."

<details>
  <summary>What are Indexes?</summary>

  Indexes are used to improve the performance of queries. Think of a book's index — instead of reading every page (full table scan), the database jumps directly to the relevant rows. Indexes are created using one or more columns of a database table. Users cannot see the indexes; they are just used to speed up searches/queries.
</details>

<details>
  <summary>Tables vs. Views</summary>

  A **table** is a physical copy of the data (materialized), while a **view** is a virtual copy — a query that is run on the fly when you access the view. A view is not stored in the database, but the query that defines it is. Views are perfect for giving analysts a simplified, pre-filtered view of complex tables without storing duplicate data.
</details>

### 🛠️ Activity 3: The CSV Dump (Code-Along)

**1. Importing Data:**

```sql
COPY lesson.teachers FROM '<full directory path>' (AUTO_DETECT TRUE);
```

> **Note:** Use the full directory path to the CSV files, e.g. `/Users/yourname/Dev/6m-data-1.3-sql-basic-ddl/data/teachers.csv`

**2. Exercise:** Import data for the `classes` and `students` tables.

**3. Updating Data:**

```sql
UPDATE lesson.students
SET email = 'linda.g@example.com'
WHERE id = 4;
```

**4. Exporting Data:**

Export with a `|` delimiter:

```sql
COPY (SELECT * FROM lesson.students) TO '<full directory path>' WITH (HEADER 1, DELIMITER '|');
```

Export as JSON:

```sql
COPY (SELECT * FROM lesson.students) TO '<full directory path>';
```

**5. Challenge:** Repeat the export steps for the `teachers` and `classes` tables.

### 🛠️ Activity 4: Index, Table & View (Code-Along)

**Creating Indexes:**

```sql
-- Create a unique index on the name column of teachers
CREATE UNIQUE INDEX teachers_name_idx ON lesson.teachers(name);

-- Create a non-unique index on the name column of students
CREATE INDEX students_name_idx ON lesson.students(name);

-- View indexes for a specific table
SELECT index_name, sql
FROM duckdb_indexes
WHERE table_name = 'students';
```

**Creating a View:**

```sql
CREATE VIEW lesson.students_view AS
SELECT id, name, email
FROM lesson.students;
```

**Challenge:** Create a `teachers_view` with the same columns but for the `teachers` table.

### 💬 Reflection

- When should you *not* use an index?
- How does using a View simplify the work for a Data Analyst who only needs specific columns?

---

## 🎯 Wrap-Up

**Key Takeaways:**
1. DDL (`CREATE`, `ALTER`, `DROP`) defines the skeleton of your database — the structure that everything else depends on.
2. Constraints prevent bad data from entering — always cheaper to reject invalid data at entry than to clean it up later.
3. `COPY`, Indexes, and Views are production tools — bulk loading, performance, and access control separate a prototype from a real database.

**Next Steps:**
- Complete the [Assignment](./assignment.md) — schema design and data import challenges.
- Next lesson: Lesson 1.4 builds on today's structure by teaching you to *query* data with DML — SELECT, WHERE, GROUP BY, and more.
