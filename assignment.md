# **📝 Post-Class Assignments: SQL DDL & Schema Design**

Estimated Time: 45-60 Minutes

**Post to Discord Peer Review Channel if You want to share your answers and discuss with peers.** https://discord.com/channels/1165846570177150996/1457586759667028094

## **Case Study 1: The "Tiny Library" (Schema Design)**

Scenario:

You have been hired by a local community library to digitize their catalog. They currently use a paper notebook. They need a database to track Books, Members, and Loans (who borrowed what).

**Requirements:**

1. **Books Table:** Needs a title, author, and a unique ISBN (13 characters).  
2. **Members Table:** Needs a name, email, and join date. Email must be unique.  
3. **Loans Table:** Tracks which member borrowed which book and when.  
   * *Constraint:* You cannot loan a book that doesn't exist.  
   * *Constraint:* You cannot loan to a member who doesn't exist.  
   * *Constraint:* The 'return\_date' can be null (meaning they haven't returned it yet), but the 'loan\_date' is required.

Task:

Write the SQL DDL statements to create these three tables in a new schema called library.

## **Case Study 2: The "Marketing Dump" (Data Ingestion)**

Scenario:

The marketing team has sent you a raw CSV file named [leads\_raw.csv](./data/leads_raw.csv) containing potential customer data collected from a web form. The file is messy and unstructured.

**Data Preview (leads\_raw.csv):**

```

id,full_name,contact_info,signup_date
101,"Alice Smith","alice@gmail.com",2023-01-01
102,"Bob Jones","555-0199",2023-01-02
103,"Charlie Brown","charlie@yahoo.com",2023-01-03

```

**Task:**

1. **Create a Staging Table:** Create a table called marketing\_leads in the public schema. Since the source data is messy, be generous with your data types (use VARCHAR for almost everything to prevent import errors).  
2. **Import Data:** Write the COPY command to load the CSV data into your table.  
3. **Create a View:** Create a view called valid\_leads that filters out rows where the contact\_info does NOT contain an '@' symbol (excluding phone numbers).

---

## **Advanced Parts for learner with Python knowledge(Optional)**
### **Local Environment Setup**

If you want to create the database file from scratch:

1. Create a new conda environment from `environment.yml`
```
  conda env create -f environment.yml
```
2. Activate the conda environment
```
  conda activate ddb
```   
3. Run [create_duckdb.py](./db/create_duckdb.py) to create the database file.
```
  python db/create_duckdb.py
```

---

## **🛑 STOP\! Solutions Below**

*Try to solve the problems above before scrolling down.*

## **✅ Solutions**

### **Solution 1: The Tiny Library**

```

CREATE SCHEMA IF NOT EXISTS library;

-- 1. Books (The Inventory)
CREATE TABLE library.books (
    isbn VARCHAR(13) PRIMARY KEY, -- ISBN is a natural unique identifier
    title VARCHAR NOT NULL,
    author VARCHAR NOT NULL,
    published_year INTEGER
);

-- 2. Members (The Users)
CREATE TABLE library.members (
    member_id INTEGER PRIMARY KEY,
    full_name VARCHAR NOT NULL,
    email VARCHAR UNIQUE, -- Ensures no two members share an email
    join_date DATE
);

-- 3. Loans (The Transaction/Link Table)
CREATE TABLE library.loans (
    loan_id INTEGER PRIMARY KEY,
    loan_date DATE NOT NULL,
    return_date DATE, -- Can be NULL (currently borrowed)
    
    -- Foreign Keys ensure data integrity
    book_isbn VARCHAR(13) REFERENCES library.books(isbn),
    member_id INTEGER REFERENCES library.members(member_id)
);

```





### **Solution 2: The Marketing Dump**

**Step 1 & 2: Create Table & Import**

```

-- Create a "loose" table to accept messy input
CREATE TABLE marketing_leads (
    id INTEGER,
    full_name VARCHAR,
    contact_info VARCHAR, -- Could be email OR phone, so we keep it generic
    signup_date DATE
);

-- Import the data
-- Note: Ensure the file path matches where you saved the CSV
COPY marketing_leads 
FROM 'leads_raw.csv' 
(AUTO_DETECT TRUE);

```

**Step 3: The Cleaning View**

```

CREATE VIEW valid_leads AS
SELECT 
    id, 
    full_name, 
    contact_info AS email, -- Rename it since we know these are emails
    signup_date
FROM marketing_leads
WHERE contains(contact_info, '@');
-- OR for standard SQL: WHERE contact_info LIKE '%@%';

```
