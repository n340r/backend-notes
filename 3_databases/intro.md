# Databases intro

Those are **must-know**. Everything else builds on those concepts.

## SQL (Structured Query Language):

- Uses structured schemas (tables with fixed columns).
- Ideal for complex queries, relationships, and transactions.
- Examples: **PostgreSQL, MySQL, SQLite**.

**Referential Integrity and Strict Relationships**
In relational databases, **strict relationships** enforce _referential integrity_, meaning that all references between tables must remain valid and consistent.

**Example:**

- **Cars** table: contains car records.
- **Drivers** table: each driver references a car via a foreign key.
- It is **impossible to delete a car** while there are still **drivers pointing to it**.

<!-- TODO:
real example of those tables -->

## SQL Lessons

**[SQL introduction lessons](https://sqlbolt.com/)** - Spend 2-4 days (**NOT MORE**) to master those lessons.  
This is for you to get basic idea on SQL. Skip or prompt for help, **you wont remember everything** anyway.

## Types of databases

<!-- TODO: emoji for all of those types -->

### Relational

> 💡 OLTP - OnLine Transaction Processing

Examples:

- **PostgreSQL** (must know)
- MySQL

Used when:

- Data can be split into exactt columns of **tables**.
- **Relationships** between tables **are clear**.
- We want **powerful SQL** language.

**Use case**:

- **Banking system** relational data, need strict transactions for safe money transfers
- **Hotel booking system** relational data, need transactions for safe bookings

### 📄 NoSQL

Examples:

- **Redis** (must know)
- **MongoDB** (must know)
- DynamoDB (AWS)

Used when:

- **Schema-less** or flexible schema (documents, key-value, graph, etc.). When data data does not have a strict structure
  and it would be doing migrations too often.
- Need sharding, partitioning for **scalability**. Mongo does this out of the box.
- **ref** (stores document id) - used to indicate **pseudo-relationships** between docs in collections.  
  Called **pseudo** because it does not have such strict constraints as in relational DBs.

<!-- TODO:
use-cases -->

### Columnar ???

<!-- todo -->

### Any other ???

<!-- todo -->

## ACID vs BASE

### ✅ ACID

Used in SQL, supported in MongoDB with transactions

**ACID** stands for:

- **A**tomicity – all or nothing: if one part fails, everything rolls back
- **C**onsistency – database always moves from one valid state to another
- **I**solation – parallel transactions don’t interfere with each other
- **D**urability – once committed, changes persist even after crashes

**Examples:**

- Transferring money between bank accounts (`UPDATE account1 - $100`, `UPDATE account2 + $100`)
- Making a hotel reservation (only commit if all data—room, payment, etc.—is valid)
- Updating multiple related tables (e.g. orders and inventory)

### 🌍 BASE

**BASE** is used in distributed NoSQL systems like MongoDB, Cassandra

**BASE** stands for:

- **B**asically **A**vailable – the system responds (but not always with up-to-date data)
- **S**oft State – system state may change over time without input
- **E**ventual Consistency – data becomes consistent _eventually_, not immediately

**Examples:**

- Adding a **comment or like** in a social media app
- Updating a **shopping cart** in a high-traffic e-commerce site
- Writing to analytics logs
- Working with **google goc**

---

> 💡 **MongoDB is ACID only at single-document level by default**.

> 💡 To get ACID for **multiple documents**, you need to explicitly use **multi-document transactions** (requires replica set).

## 🔐 Transactions

**Transaction** - group several operations into one **atomic** operation. Either all succeed or nether do.  
It is a **fundamental thing for ACID** operations in Postgres.

**Example:**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

It is **all or nothing**. If anything goes wrong: - `ROLLBACK`

## ⚠️ Transaction Problems and Phenomena (ACID Issues)

> 💡 Please **DO NOT** memorize transaction anomalies and isolation levels. You **should understand** them and be able give an **example**.

### 🔄 Transaction Execution

- **Sequential execution**: One transaction finishes before the next begins. **Safe but slow**.
- **Parallel execution**: Multiple transactions run at the same time. Faster, but **can introduce conflicts**.

**Databases like PostgreSQL and MongoDB allow parallel execution** with safeguards (isolation levels, locks).  
If transactions touch **the same data**, they are executed **sequentially**, using locks.

**Transaction Anomalies** - Those **might** appear if transactions run **in parallel**. Listed below

### 1️⃣ Lost Update

> Not classified as a separate SQL anomaly but rather a consequence of insufficient isolation and lack of proper locking.

**Real-world example**:  
Two support-agents update a user's profile at the same time. Only one change is saved.

**Problem**: Two transactions update the same row. One overwrites the other's changes.

**Why it happens**: No locking or poor isolation. Both read the same initial data, then write over each other.

---

### 2️⃣ Dirty Read

**Real-world example**:  
An app shows a hotel as "available" because a parallel transaction marked it free—but that transaction later rolls back.

**Problem**: A transaction reads **uncommited data** that might be rolled back midway

**Why it happens**: Reading "unconfirmed" changes.

---

### 3️⃣ Non-repeatable Read

**Real-world example**:  
A library system checks a book's status, sees `free`, later in the same transaction check again, the book appears `borrowed`, then again, book is `free` again

**Problem**: A transaction reads **different commited data** in the process

**Why it happens**: Another transaction updated and committed the row mid-process.

---

### 4️⃣ Phantom Read

**Real-world example**:  
A report is generated for "all bookings today" → another booking happens mid-report → second query includes more rows.

**Problem**: A transaction re-runs a query and finds **new rows** that weren’t there before.

**Why it happens**: Rows are inserted/deleted between identical `SELECT` queries.

---

### 5️⃣ Serialization Anomaly

**Real-world example**:

- Starting balance = $1000
- **Transaction A**: Adds $1000 to account.
- **Transaction B**: Applies 10% interest.

Possible outcomes if done **sequentially**:

- A then B: $2000 → +10% → **$2200**
- B then A: $1000 → +10% → $1100 → +$1000 → **$2100**

But with poor isolation, a parallel execution might cause a **wrong result**, like **$3200**, which is **impossible** in any sequential order.

**Problem**: The result of executing multiple parallel transactions **does not match any possible sequential order** of those transactions.

**Why it matters**: Data integrity is broken. You can’t trace the outcome to any valid sequence of operations.

---

### 🧠 Summary of Anomalies

| Phenomenon            | Description                                                 | Real-life Example                        |
| --------------------- | ----------------------------------------------------------- | ---------------------------------------- |
| Lost Update           | One update overwrites another                               | Two users edit profile simultaneously    |
| Dirty Read            | Read uncommitted changes from another transaction           | See hotel room that will later be locked |
| Non-repeatable Read   | Row value changes within same transaction                   | Book status changed while borrowing      |
| Phantom Read          | New rows appear on same query within transaction            | New bookings appear in report            |
| Serialization Anomaly | Result doesn’t match _any_ sequential order of transactions | Bank account ends up with invalid amount |

## 🔐 SQL Isolation Levels

Isolation levels define how visible changes from other transactions are **during** a transaction.

Imagine you're working on a shared Google Doc.

**Google Docs analogy**:  
Each **person editing** is like a **transaction**.  
The **Google Doc** is your **database**.  
The **text you're editing** is the **data**.

---

### 1️⃣ Read Uncommitted (🚫 Not actually used in PostgreSQL)

**Google Docs analogy**:  
You're watching another user type in real time—even if they haven't saved their changes yet.

- You see **live edits** (even if the other user deletes them seconds later).
- You might copy text that vanishes because it was never saved.

**Behavior**:

- ✅ Fastest
- ❌ Dirty reads possible
- ❌ Not supported in PostgreSQL (internally treated as Read Committed)

---

### 2️⃣ Read Committed (🔁 Default in PostgreSQL)

**Google Docs analogy**:  
You only see **what was saved** (committed) live _before each action you do_.

- You see **live edits** of saved edits.

**Prevents**:

- ✅ Dirty reads  
  **Allows**:
- ❌ Non-repeatable reads
- ❌ Phantom reads

---

### 3️⃣ Repeatable Read (🧊 Snapshot of your session)

**Google Docs analogy**:  
Once you open the doc, you get a **frozen snapshot** of it.

- Others can still edit behind the scenes but your version **doesn’t change** until you leave and reopen the doc.

**Prevents**:

- ✅ Dirty reads
- ✅ Non-repeatable reads
- ✅ Phantom reads (in PostgreSQL)

---

### 4️⃣ Serializable (📦 Strictest, safest)

**Google Docs analogy**:  
You're the **only one allowed to edit** for now.  
Like **Repeatable Read** + others are not allowed to edit at all

- If two people try to edit the same text, one must wait—or be forced to restart.
- All parallel edits behave **as if** they happened **one at a time**, in some sequence.

**Prevents**:

- ✅ Dirty reads
- ✅ Non-repeatable reads
- ✅ Phantom reads
- ✅ Serialization anomalies

**Tradeoff**:

- ✅ Maximum data safety
- ❌ Slower (may block or rollback under high concurrency)

<!-- todo:
revisit examles on those isolation levels because i found out serializable is not needed even for mone -->

---

### 🗂️ Summary Table

| Isolation Level  | Dirty Read | Non-repeatable Read | Phantom Read | Serializable Anomaly   | Notes                               |
| ---------------- | ---------- | ------------------- | ------------ | ---------------------- | ----------------------------------- |
| Read Uncommitted | ✅ Allowed | ✅ Allowed          | ✅ Allowed   | ✅ Allowed             | PostgreSQL treats as Read Committed |
| Read Committed   | ❌ No      | ✅ Allowed          | ✅ Allowed   | ✅ Allowed             | Default in PostgreSQL               |
| Repeatable Read  | ❌ No      | ❌ No               | ❌ No        | ✅ Allowed (in theory) | PostgreSQL blocks phantoms          |
| Serializable     | ❌ No      | ❌ No               | ❌ No        | ❌ No                  | Most strict, safest                 |

> ℹ️ **Lost Update** is not listed as a formal "read phenomenon" but is a real issue.

> Default isolation level in PG is `READ COMMITED`

> Isolation levels can be configured _per-query_, _per-session_

## 🔒 Locks in PostgreSQL

Locks designed to ensure data consistency when multiple transactions access it simultaneously.

Think **edit access** rules in Google Docs.

### Types of Locks

Locks differ by their _behaviour_ and _scale_

### 📏 Different types of lock scale

| Lock Type          | Description                                     |
| ------------------ | ----------------------------------------------- |
| **Row-level**      | Locks individual rows (`SELECT ... FOR UPDATE`) |
| **Table-level**    | Locks the entire table (`LOCK TABLE users`)     |
| **Database-level** | Locks entire DB (e.g., during backup/restore)   |

### 🧩 Pessimistic Locking behaviour

Assume conflict will happen

**Google Docs analogy**:  
You “lock” a paragraph so others **can’t edit it** while you’re working.

- Use: `SELECT ... FOR UPDATE`
- Others can **see** the data, but not change it.
- Prevents race conditions.
- **Slower** due to many locks.

### 🧩 Optimistic Locking behaviour

<!-- todo: Do i get it right that in this case app should detect conflicts ? -->

Optimistic locking assumes **conflicts are rare** and **app detects** them at write time, **not by locking** data upfront.

**Google Docs analogy**:  
Everyone can edit freely. When you click save, Google checks if someone else has changed the same paragraph.  
If yes, you get a **conflict warning**.

Process in a nutshell:

- Do `transaction 1` on the row `WHERE id = 10 AND version = 10`, then **increment** version
- Next `transaction 2` tries to work on the same row `WHERE id = 10 ADN version = 10` but encounters `version = 11` hence `0` rows affected.
- App checks, if `0` rows affected - error or retry

### Extra notes on lock types

<!-- todo: those conceps sound too much to me: row exclusive, row share, access excluseive, share update exclusive - do i
actually need to know all this for real life and even interviews ? -->

> 💡 `SHARE` in case of DB means read as long as you are not writing.

> All of those locks below are **table-scoped**

| Lock Mode                  | Blocks                           | Typical usage                        | Google Docs analogy                                      |
| -------------------------- | -------------------------------- | ------------------------------------ | -------------------------------------------------------- |
| **Row Exclusive**          | DDL like DROP/TRUNCATE           | Auto by `INSERT`, `UPDATE`, `DELETE` | Editing a paragraph; can’t delete document while editing |
| **Row Share**              | DDL requiring exclusive lock     | `SELECT ... FOR SHARE`               | Reading, commenting but doc can’t be deleted             |
| **Share**                  | Writers (`INSERT/UPDATE/DELETE`) | `LOCK TABLE ... IN SHARE MODE`       | Everyone can read, no one can type                       |
| **Share Update Exclusive** | Other VACUUM/DDL                 | Internal (VACUUM)                    | System doing background cleanup                          |
| **Exclusive**              | All reads/writes by others       | `LOCK TABLE ... IN EXCLUSIVE MODE`   | You fully “checked out” document; others wait            |
| **Access Exclusive**       | Everything                       | `ALTER`, `DROP`, `TRUNCATE`          | Admin locks doc for restructure; no access               |

## Deadlocks

Two or more transactions mutually block each other.  
PostgreSQL automatically detects deadlocks and aborts one of the transactions.  
Throws an error that should be handled by the application.

<!-- todo: example of a deadlock from wallrm interview -->

## Database Access and Tools in Node.js

### ORM (Object-Relational Mapping)

ORM provides an **abstraction layer** over the SQL language and database, you interact with DB in an **object-oriented** paradigm.  
You define models, those map to database tables, and then **ORM itself decides** how to generate SQL and run it in your DB.

> 💡 Most **experienced** backend devs **dislike** ORMs. Some ORMs **fail** to do their job. **Prisma** cannot use same models on both **MySQL** and **Postgres**.

**✅ Pros:**

- **Abstraction**: hides complex SQL, you work with DB entities as objects.
- **Productivity**: Speeds up development with less boilerplate code.
- **Security**: Helps prevent SQL injection attacks by sanitizing inputs.
- **Maintainability**: Easier to refactor and maintain code, especially in large projects.
- **Database Agnostic (to some extent)**: Many ORMs support multiple database systems, allowing easier switching.

**🔴 Cons:**

- **Performance Overhead**: Can generate **slower queries** for complex operations, leading to **n+1** query issues.
- **Limited SQL Control**: Hard to predict which SQL will be executed
- **ORM magic**: Has it's own context, cache and so on.

**Popular Node.js ORMs for SQL Databases**

- **Prisma**: Modern, simple. Generates a type-safe client.
- **TypeORM**: More established, well-recognized. Emphasizes TypeScript and decorators.
- **Sequelize**: robust with strong transaction support and migrations.

> 💡 ORMs for Node.js are promise-based

### Lightweight SQL Builders

Simple API to build queries, greater control over the generated SQL. **DO NOT** actually execute SQL in the DB, the only thing they do is help write, validate SQL.

**✅ Pros:**

- **Control**: Offers more control over the generated SQL compared to ORMs.
- **Simplicity**: Often more readable than raw SQL strings.
- **Security**: Helps prevent SQL injection through parameter binding.

**Popular Node.js SQL Builders:**

- `pgtyped`: Generates types from SQL queries for type-safe raw SQL with PostgreSQL.
- `Kysely`
- `Knex.js`

### Raw SQL

**✅ Pros:**

- **Control**: Mostly used via `execute` command in ORM when it generates bad SQL.

**🔴 Cons:**

- **Be careful**: Should watch out for typos, security, readability

## Table Relationships

- One-to-One: `User` <--> `UserProfile`
- One-to-Many: `Author` <--> `Books`
- Many-to-Many `Students` <--> `Courses`

**Link Table (Junction Table)**
Helps manage **many-to-many** relationships.  
Contains foreign keys from both related tables.
**Example:** `People` <--> `Events` through a `registrations` table.

**Example** Junction table

```sql
CREATE TABLE students (
  student_id SERIAL PRIMARY KEY,
  student_name VARCHAR(100) NOT NULL,
  student_age INT NOT NULL
);

CREATE TABLE courses (
  course_id SERIAL PRIMARY KEY,
  course_name VARCHAR(100) NOT NULL,
  course_duration VARCHAR(50) NOT NULL
);

-- Connect students <--> courses table via this junction table
CREATE TABLE student_courses (
  student_id INT REFERENCES students(student_id),
  course_id INT REFERENCES courses(course_id),
  PRIMARY KEY (student_id, course_id)
);
```
