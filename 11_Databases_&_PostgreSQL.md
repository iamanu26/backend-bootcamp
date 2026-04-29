# 🗄️ Databases & PostgreSQL — Complete Notes

> A comprehensive guide covering database fundamentals, PostgreSQL-specific features, data modeling, advanced querying, and performance optimization.

---

## Table of Contents

1. [Introduction to Databases and Persistence](#1-introduction-to-databases-and-persistence)
2. [Database Management Systems (DBMS)](#2-database-management-systems-dbms)
3. [Relational vs. Non-Relational Databases](#3-relational-vs-non-relational-databases)
4. [Data Types in PostgreSQL](#4-data-types-in-postgresql)
5. [Database Migrations and Seeding](#5-database-migrations-and-seeding)
6. [Practical Database Modeling](#6-practical-database-modeling)
7. [Advanced Querying and Performance](#7-advanced-querying-and-performance)

---

## 1. Introduction to Databases and Persistence

### What is a Database?

At its core, a **database is a mechanism to persist information across different sessions**. Persistence means your data survives even after the program that created it has stopped running.

### Everyday Examples of Databases

You interact with basic databases all the time:

| Example | Type |
|---|---|
| Smartphone contact list | Structured local storage |
| Browser local storage | Key-value store |
| Plain text files | Flat-file database |

> These are all valid, simple forms of data persistence — but they have severe limitations at scale.

### Why Disk-Based Storage in Backend Engineering?

In backend engineering, "database" almost always refers to **disk-based storage (HDD or SSD)**, not in-memory storage (RAM).

| Storage Type | Speed | Cost | Capacity |
|---|---|---|---|
| RAM | ⚡ Very Fast | 💰 Expensive | Limited |
| Disk (HDD/SSD) | 🐢 Slower | 💲 Cheap | Massive |

**Conclusion:** Disk is the practical choice for persistent, large-scale data storage despite being slower than RAM.

---

## 2. Database Management Systems (DBMS)

### What is a DBMS?

A **Database Management System (DBMS)** is a software layer that sits between your application and raw data storage. It manages data and exposes efficient **CRUD** operations:

```
Create → Read → Update → Delete
```

### Core Responsibilities of a DBMS

- **Data Organization** — Structures data in a consistent, queryable format
- **Access Control** — Manages who can read/write what data
- **Data Integrity** — Ensures accuracy and validity of stored data
- **Security** — Protects data from unauthorized access

### Why Not Just Use Text Files?

Text files seem simple but fail at scale for three key reasons:

| Problem | Explanation |
|---|---|
| **No structure** | Parsing arbitrary text is error-prone and slow |
| **Poor performance** | No indexing; must scan entire file for queries |
| **No concurrency** | Two simultaneous writes corrupt the file |

**Concurrency problem example:**

```
User A reads balance: $500
User B reads balance: $500
User A deducts $100 → writes $400
User B deducts $200 → writes $300  ❌ (should be $200)
```

A DBMS handles this with **locking and transactions**, preventing data corruption.

---

## 3. Relational vs. Non-Relational Databases

### Relational Databases (SQL)

- Organizes data into **tables**, **rows**, and **columns**
- Requires a **predefined schema** (structure defined upfront)
- Enforces **strong data integrity** via constraints
- Uses **SQL (Structured Query Language)** for all operations

**Popular choices:** PostgreSQL, MySQL, SQLite, Microsoft SQL Server

```sql
-- Example: Simple relational table
CREATE TABLE users (
    id     SERIAL PRIMARY KEY,
    name   VARCHAR(100),
    email  VARCHAR(255) UNIQUE NOT NULL
);
```

### Non-Relational Databases (NoSQL)

- Uses **flexible schemas** — no rigid structure required
- Stores data as **documents**, **key-value pairs**, **graphs**, or **wide columns**
- Ideal for **rapid prototyping** or **unstructured data** (e.g., CMS content, logs)

**Popular choices:** MongoDB, Redis, DynamoDB, Cassandra

```json
// Example: MongoDB document
{
  "_id": "abc123",
  "name": "Alice",
  "preferences": { "theme": "dark", "language": "en" },
  "tags": ["admin", "beta-tester"]
}
```

### SQL vs. NoSQL — Quick Comparison

| Feature | SQL (Relational) | NoSQL (Non-Relational) |
|---|---|---|
| Schema | Fixed / predefined | Flexible / dynamic |
| Data integrity | Strong | Varies |
| Query language | SQL (standardized) | Varies by database |
| Best for | Complex relationships, transactions | Unstructured data, high-speed reads |
| Examples | PostgreSQL, MySQL | MongoDB, Redis |

### Why PostgreSQL?

PostgreSQL stands out for several reasons:

- ✅ **Open-source** — Free to use with a strong community
- ✅ **SQL standards compliant** — Follows ANSI SQL closely
- ✅ **Highly extensible** — Supports custom types, functions, and extensions
- ✅ **JSONB support** — Binary JSON storage that enables NoSQL-like flexibility within a relational database

> **JSONB** lets PostgreSQL handle dynamic, schema-less data that you'd traditionally need MongoDB for — giving you the best of both worlds.

---

## 4. Data Types in PostgreSQL

Choosing the right data type is critical for performance, accuracy, and storage efficiency.

### Numeric Types

| Type | Use Case | Notes |
|---|---|---|
| `SERIAL` | Auto-incrementing IDs | Shorthand for `INTEGER` + sequence |
| `INTEGER` | Whole numbers | Standard integer, 4 bytes |
| `DECIMAL` / `NUMERIC` | Money, precise values | Exact precision, slightly slower |
| `FLOAT` | Scientific data | Faster but can have rounding errors |

```sql
-- Use DECIMAL for money, never FLOAT
price   DECIMAL(10, 2),   -- up to 99,999,999.99
rating  FLOAT             -- fine for approximate values
```

> ⚠️ **Never use `FLOAT` for currency.** Floating point arithmetic is imprecise and can cause cents-level errors at scale.

### String Types

| Type | Use Case | Notes |
|---|---|---|
| `CHAR(n)` | Fixed-length strings | Pads with spaces; rarely used |
| `VARCHAR(n)` | Variable-length with a limit | Good when enforcing a max length |
| `TEXT` | Unlimited-length strings | **Preferred in PostgreSQL** — same performance as VARCHAR |

```sql
-- PostgreSQL recommendation
username   VARCHAR(50),   -- enforce max length when needed
bio        TEXT           -- prefer TEXT for flexibility
```

> In PostgreSQL, `TEXT` and `VARCHAR` have **identical performance**. Use `TEXT` unless you specifically need to enforce a character limit.

### Other Essential Types

| Type | Use Case | Example |
|---|---|---|
| `BOOLEAN` | True/false flags | `is_active BOOLEAN DEFAULT true` |
| `DATE` | Calendar dates | `2024-01-15` |
| `TIMESTAMP WITH TIME ZONE` | Precise moments in time | `2024-01-15 10:30:00+05:30` |
| `UUID` | Globally unique identifiers | `a1b2c3d4-e5f6-...` |
| `JSONB` | Binary JSON documents | Fast indexing, flexible structure |

```sql
-- UUID primary key (more secure than serial IDs)
id         UUID DEFAULT gen_random_uuid() PRIMARY KEY,

-- JSONB for flexible metadata
metadata   JSONB
```

---

## 5. Database Migrations and Seeding

### What are Migrations?

**Migrations** are versioned SQL files that describe changes to your database schema over time. They allow your team to:

- Track every schema change in **version control**
- Apply changes consistently across **dev, staging, and production**
- **Roll back** to a previous state if something breaks

### Migration Structure: Up & Down

Every migration has two parts:

```sql
-- UP migration: apply the change
CREATE TABLE projects (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- DOWN migration: undo the change (rollback)
DROP TABLE IF EXISTS projects;
```

### Migration File Naming Convention

```
migrations/
├── 001_create_users.sql
├── 002_create_projects.sql
├── 003_add_status_to_projects.sql
└── 004_create_tasks.sql
```

> Always number migrations sequentially. They **must** run in order.

### What is Seeding?

**Seeding** is inserting test/sample data into the database during development to verify your application logic works correctly.

```sql
-- seed.sql
INSERT INTO users (name, email) VALUES
    ('Alice Johnson', 'alice@example.com'),
    ('Bob Smith',     'bob@example.com'),
    ('Charlie Brown', 'charlie@example.com');
```

| Concept | Purpose | Environment |
|---|---|---|
| Migration | Schema changes | All environments |
| Seeding | Test data | Development only |

---

## 6. Practical Database Modeling

This section walks through designing a **project management system** from scratch.

### Using Enums for Controlled Values

Enums create custom types that restrict a column to a defined set of values, enforcing data integrity at the database level.

```sql
-- Create custom enum type
CREATE TYPE project_status AS ENUM ('active', 'on_hold', 'completed', 'archived');
CREATE TYPE task_priority  AS ENUM ('low', 'medium', 'high', 'urgent');

-- Use it in a table
CREATE TABLE projects (
    id      SERIAL PRIMARY KEY,
    name    TEXT NOT NULL,
    status  project_status DEFAULT 'active'
);
```

> ✅ Benefit: The database itself rejects any value not in the enum — no need to validate in application code.

### Relationships

#### One-to-One — Users & Profiles

```sql
CREATE TABLE users (
    id         SERIAL PRIMARY KEY,
    email      VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE user_profiles (
    id         SERIAL PRIMARY KEY,
    user_id    INTEGER UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    full_name  TEXT,
    avatar_url TEXT,
    bio        TEXT
);
```

- `UNIQUE` on `user_id` enforces the one-to-one constraint
- `ON DELETE CASCADE` removes the profile if the user is deleted

#### One-to-Many — Projects & Tasks

```sql
CREATE TABLE tasks (
    id          SERIAL PRIMARY KEY,
    project_id  INTEGER REFERENCES projects(id) ON DELETE CASCADE,
    title       TEXT NOT NULL,
    priority    task_priority DEFAULT 'medium',
    due_date    DATE
);
```

- One project can have **many tasks**
- Each task belongs to exactly **one project**

#### Many-to-Many — Projects & Users (via linking table)

```sql
CREATE TABLE project_members (
    project_id  INTEGER REFERENCES projects(id) ON DELETE CASCADE,
    user_id     INTEGER REFERENCES users(id) ON DELETE CASCADE,
    role        TEXT DEFAULT 'member',
    joined_at   TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Composite primary key prevents duplicate memberships
    PRIMARY KEY (project_id, user_id)
);
```

- A user can belong to **many projects**
- A project can have **many users**
- The **linking table** (`project_members`) resolves the many-to-many relationship

### Referential Integrity: ON DELETE Behavior

| Option | Behavior | Use When |
|---|---|---|
| `ON DELETE CASCADE` | Deletes child rows automatically | Child data is meaningless without parent (e.g., tasks without a project) |
| `ON DELETE RESTRICT` | Blocks deletion if children exist | Child data must be preserved (e.g., don't delete a user with active orders) |
| `ON DELETE SET NULL` | Sets foreign key to NULL | Child can exist independently |

```sql
-- Example: Prevent deleting a user who owns projects
project_owner_id INTEGER REFERENCES users(id) ON DELETE RESTRICT
```

### Full Schema Example

```sql
-- 1. Enums
CREATE TYPE project_status AS ENUM ('active', 'on_hold', 'completed', 'archived');
CREATE TYPE task_priority  AS ENUM ('low', 'medium', 'high', 'urgent');

-- 2. Users
CREATE TABLE users (
    id         SERIAL PRIMARY KEY,
    email      VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 3. User Profiles (One-to-One)
CREATE TABLE user_profiles (
    id        SERIAL PRIMARY KEY,
    user_id   INTEGER UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    full_name TEXT,
    bio       TEXT
);

-- 4. Projects
CREATE TABLE projects (
    id         SERIAL PRIMARY KEY,
    name       TEXT NOT NULL,
    status     project_status DEFAULT 'active',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 5. Tasks (One-to-Many with Projects)
CREATE TABLE tasks (
    id          SERIAL PRIMARY KEY,
    project_id  INTEGER REFERENCES projects(id) ON DELETE CASCADE,
    assigned_to INTEGER REFERENCES users(id) ON DELETE SET NULL,
    title       TEXT NOT NULL,
    priority    task_priority DEFAULT 'medium',
    due_date    DATE,
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 6. Project Members (Many-to-Many)
CREATE TABLE project_members (
    project_id INTEGER REFERENCES projects(id) ON DELETE CASCADE,
    user_id    INTEGER REFERENCES users(id)    ON DELETE CASCADE,
    role       TEXT DEFAULT 'member',
    joined_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    PRIMARY KEY (project_id, user_id)
);
```

---

## 7. Advanced Querying and Performance

### JOINs

Use `LEFT JOIN` to fetch a user along with their profile, **even if the profile doesn't exist yet**.

```sql
-- INNER JOIN: only users WITH a profile
SELECT u.id, u.email, p.full_name
FROM users u
INNER JOIN user_profiles p ON p.user_id = u.id;

-- LEFT JOIN: all users, profile data is NULL if missing
SELECT u.id, u.email, p.full_name, p.bio
FROM users u
LEFT JOIN user_profiles p ON p.user_id = u.id;
```

### JSON Aggregation

Embed related data into a single result row using `to_jsonb`:

```sql
SELECT
    u.id,
    u.email,
    to_jsonb(p) AS profile
FROM users u
LEFT JOIN user_profiles p ON p.user_id = u.id;
```

**Result:**
```json
{
  "id": 1,
  "email": "alice@example.com",
  "profile": {
    "full_name": "Alice Johnson",
    "bio": "Backend developer"
  }
}
```

### Parameterized Queries — Preventing SQL Injection

**Never** interpolate user input directly into SQL strings.

```javascript
// ❌ DANGEROUS — SQL Injection vulnerability
const query = `SELECT * FROM users WHERE email = '${userInput}'`;

// ✅ SAFE — Parameterized query
const query = 'SELECT * FROM users WHERE email = $1';
const result = await db.query(query, [userInput]);
```

**Why it matters:** A malicious user could input `' OR '1'='1` and gain access to all records.

### Pagination and Sorting

```sql
-- Get page 2 of results (10 items per page), newest first
SELECT id, title, created_at
FROM tasks
WHERE project_id = $1
ORDER BY created_at DESC
LIMIT 10
OFFSET 10;  -- Skip first 10 results (page 1)
```

| Clause | Purpose |
|---|---|
| `ORDER BY` | Sort results by column |
| `LIMIT` | Max number of rows to return |
| `OFFSET` | Number of rows to skip (for pagination) |

> **Page formula:** `OFFSET = (page_number - 1) * page_size`

### Indexes

An **index** is a separate lookup structure that dramatically speeds up `WHERE` and `JOIN` queries.

```sql
-- Primary keys are indexed automatically ✅

-- Foreign keys need manual indexing ⚠️
CREATE INDEX idx_tasks_project_id    ON tasks(project_id);
CREATE INDEX idx_tasks_assigned_to   ON tasks(assigned_to);
CREATE INDEX idx_project_members_uid ON project_members(user_id);

-- Partial index for a specific query pattern
CREATE INDEX idx_active_projects ON projects(status) WHERE status = 'active';
```

**How indexes work:**

```
Without index:  Scan ALL rows → O(n)
With index:     Jump directly to matching rows → O(log n)
```

> ⚠️ **Tradeoff:** Indexes speed up reads but slow down writes (INSERT/UPDATE/DELETE), since the index must also be updated. Only index columns you frequently query.

### Triggers — Auto-updating `updated_at`

A **trigger** automatically runs a function whenever a row is modified.

```sql
-- Step 1: Create the function
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Step 2: Attach the trigger to a table
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();

-- Repeat for other tables
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON projects
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();
```

> ✅ Now `updated_at` is always accurate without any application-level code.

---

## Summary Cheat Sheet

```
Databases
├── Purpose: Persist data across sessions
├── Backend: Disk-based (HDD/SSD) — cheap & large
│
├── DBMS: Software managing CRUD, integrity, security, concurrency
│
├── Types
│   ├── Relational (SQL): Tables, schema, strong integrity — PostgreSQL ✅
│   └── Non-Relational (NoSQL): Flexible schema, documents — MongoDB
│
├── PostgreSQL Data Types
│   ├── Numeric: SERIAL, INTEGER, DECIMAL, FLOAT
│   ├── String:  CHAR, VARCHAR, TEXT (preferred)
│   └── Other:   BOOLEAN, DATE, TIMESTAMP, UUID, JSONB
│
├── Migrations: Versioned SQL files (Up/Down) for schema changes
├── Seeding: Insert test data for development
│
├── Relationships
│   ├── One-to-One:   users ↔ user_profiles
│   ├── One-to-Many:  projects → tasks
│   └── Many-to-Many: projects ↔ users (via project_members)
│
└── Performance
    ├── Joins: INNER, LEFT JOIN
    ├── JSON Aggregation: to_jsonb()
    ├── Security: Parameterized queries (prevent SQL injection)
    ├── Pagination: LIMIT + OFFSET + ORDER BY
    ├── Indexes: Speed up reads on WHERE/JOIN columns
    └── Triggers: Automate updated_at timestamps
```

