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

## 👨‍🏫 SQL Lessons

**[SQL introduction lessons](https://sqlbolt.com/)** - Spend 2-4 days (**NOT MORE**) to master those lessons.  
This is for you to get basic idea on SQL. Skip or prompt for help, **you wont remember everything** anyway.

## 🔠 Types of databases

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

## ⚔️ ACID vs BASE

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

**Transaction** - group several steps into one **atomic** operation. Either all succeed or nether do.  
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

Before we proceed any further, really really understand the following:

- **Isolation levels** - define what you can see.
- **Locks** define what others can do.

Isolation levels, Locks **work together**.

### 🔄 Transaction Execution

- **Sequential execution**: One transaction finishes before the next begins. **Safe but slow**.
- **Parallel execution**: Multiple transactions run at the same time. Faster, but **can introduce conflicts**.

❗ Understand possible anomalies, then undersand system constraints and **only then decide** how to combat them.

Databases like `PostgreSQL` and `MongoD` **allow parallel execution** with safeguards (isolation levels, locks).

**Transaction Anomalies** - Those **might** appear if transactions run **in parallel**. Listed below

### Two lists of anomaly types

**Read-related**

- Dirty read
- Non-repeatable read
- Phantom read

**Write-related**

- Lost update
- Write skew

> 💡 These **write anomalies** are usually what causes **more problems** in modern databases

**Global concern**

- Serialization anomaly - transactions running in parallel give us result that is not achievable with sequential execution.

---

### 1️⃣ Dirty Read

You see data that is **not even commited** by others. **Not possible** in postgres.

```sql
-- Initial state:
-- status = 'free'

T1 BEGIN
T1 UPDATE status = 'occupied'
-- not committed yet

T2 BEGIN
T2 SELECT status → 'occupied' ❌
T2 COMMIT

T1 ROLLBACK

-- Status is actually 'free'
-- T2 read data that never existed
```

---

### 2️⃣ Non-repeatable Read

You read **different commited** states of the same data.

```sql
-- Initial state:
-- status = 'free'

T1 BEGIN
T1 SELECT status → 'free'

T2 BEGIN
T2 UPDATE status = 'borrowed'
T2 COMMIT

T1 SELECT status → 'borrowed' ❌
T1 COMMIT
```

---

### 3️⃣ Phantom Read

Every query gets **different number** of results.

```sql
-- Initial state:
-- no bookings for today

T1 BEGIN
T1 SELECT COUNT(*) FROM bookings WHERE date = 'today' → 0

T2 BEGIN
T2 INSERT INTO bookings(date) VALUES ('today')
T2 COMMIT

T1 SELECT COUNT(*) FROM bookings WHERE date = 'today' → 1 ❌
T1 COMMIT
```

### 4️⃣ Lost Update

Transactions **overide** each other. **Last writer wins**

```sql
-- Initial state:
-- balance = 100

-- T1: apply 10% interest
T1 BEGIN
T1 SELECT balance → 100
T1 UPDATE balance = 110

T2 BEGIN -- in parallel
T2 SELECT balance → 100 -- T2: deposit 100
T2 UPDATE balance = 200

T1 COMMIT
T2 COMMIT
```

---

### 5️⃣ Write Skew

Two transactions read **overlapping data**, but update **disjoint** rows, and together they **violate an invariant**.

```sql
-- Initial state:
-- Alice = on_call
-- Bob   = on_call

-- Invariant: At least one employee MUST be on call

T1 BEGIN
T1 SELECT Alice, Bob → on, on
T1 UPDATE Alice = off

T2 BEGIN
T2 SELECT Alice, Bob → on, on
T2 UPDATE Bob = off

T1 COMMIT
T2 COMMIT

-- They did NOT update the same state, but result is wrong.
-- Alice = off
-- Bob   = off ❌
```

### 6️⃣ Serialization Anomaly

PostgreSQL tracks when one transaction **reads a row** that another concurrent transaction **later modifies**.

```sql
-- Initial state:

-- Alice = on_call
-- Bob   = on_call

T1 BEGIN
-- T1 read Alice, Bob, but T2 updates Bob. Well, to get this read we need T1 -> T2
T1 SELECT Alice, Bob → on, on
T1 UPDATE Alice = off

T2 BEGIN
-- But T2 read Alice, Bob, T1 updates Alice! Okay, then order should be T2 -> T1
T2 SELECT Alice, Bob → on, on
T2 UPDATE Bob = off

T1 COMMIT
T2 COMMIT

-- What we basically end up with T1 <--> T2 depend on each other - not possible sequentially
```

---

## 🔐 SQL Isolation Levels

A few **important concepts** to understand first.

Imagine you have a transaction that does multiplt things:

```sql
SELECT ...
UPDATE ...
INSERT ...
DELETE ...
```

**statement / query** - Each one of those `SELECT`, `UPDATE` ...
**transaction** - A group of those (as you already know)

---

### 1️⃣ Read Uncommitted (🚫 Not actually used in PostgreSQL)

Even every statement inside transaction sees **uncommited** updates.

- ❌ None generally wants it. Not even supported in PostgreSQL (treated as Read Committed)

---

### 2️⃣ Read Committed (🔁 Default in PostgreSQL)

Each **statement** gets it's own snapshot

```sql
T1 BEGIN
T1 SELECT balance → 100
T2 UPDATE balance → 200 COMMIT
T1 SELECT balance → 200 ← changed!
T1 COMMIT
```

As you can see, between T1 **statements** other transactions can commit.

---

### 3️⃣ Repeatable Read (🧊 Snapshot of your session)

Transaction gets **one snapshot** of data that **each SELECT** insisde it sees.

```sql
-- balance = 100
T1 BEGIN -- snapthot balance = 100
T1 SELECT balance → 100
T2 UPDATE balance → 200 COMMIT
T1 SELECT balance → 100 ← unchanged
T1 COMMIT
```

It is like each **SELECT** gets it's own copy (snapshot).

> ❗ Very **critically important bit**. If **parallel** transactions try to change **same row** - ERROR

```sql
-- balance = 150

-- T1
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE user_id = 123;  -- 150 (from T1 snapshot)

-- T2 (in parallel)
BEGIN ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts
SET balance = balance - 100
WHERE user_id = 123;
COMMIT;  -- balance becomes 50

-- T1 (continues)
SELECT balance FROM accounts WHERE user_id = 123;  -- still 150 (snapshot!)
UPDATE accounts
SET balance = balance - 100  -- PG is like, let me check if balance was updated in the meantime ... it was, ABORT !
WHERE user_id = 123;
COMMIT;

-- ERROR: could not serialize access due to concurrent update
```

> ❗ Select then write **makes no sense**. Use one-statement invariants `UPDATE accounts SET balance ... WHERE ...`

---

## 📦 Serializable

Serialization anomaly is **not possible**

---

> 💡 Default isolation level in PG is `READ COMMITED`

> 💡 Isolation levels can be configured _per-query_, _per-session_

## 🔒 Locks in PostgreSQL

### Types of Locks

Locks differ by their _behaviour_ and _scale_

### 📏 Scales of locking

| Lock Type          | Description                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------- |
| **Row-level**      | (**99%** of the time) Locks individual rows - `SELECT ... FOR UPDATE`                       |
| **Table-level**    | (Maintenance, one-off migrations) Locks the entire table (`LOCK TABLE users`)               |
| **Database-level** | (Mostly **implicit**: cooordination, backups) Locks entire DB (e.g., during backup/restore) |

> 💡 In real life **locking** almost always refers to row-level locks.

**Small note on table locks**

Table locks have different modes. They are out of scope for normal interviews and daily backend work, but it’s important to know that table locking can be used:

- For maintenance,
- To allow only reads,
- Or to block everything (including reads).

### 🧩 Pessimistic Locking behaviour

**Prevent conflicts** by locking the data bofore modifying.

You assume **conflict will** happen or **retries are expensive** for us.

Mechanism: `SELECT ... FOR UPDATE`

**Effect**:

- other transactions block
- no retry needed
- lower concurrency

### 🧩 Optimistic Locking behaviour

**Detect conflicts** by retrying either on database level (Serializable) or app level (retries)

Conflicts are **rare**, retries are **cheap**. High throughput needed - locking data **will slow** us down the app.

**Mechanisms**:

- `Serializable` on DB
- detect-retry **by app**

**Effect**:

- No blocking
- May abort, needs retry

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
