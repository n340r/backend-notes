# Database Introduction

[SQL introduction lessons](https://sqlbolt.com/)  
(_Spend 3-4 days (not more) to master those lessons_)

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

### Shorthands used in PostgreSQL world

| Аббревиатура | Название                     | Пример           | Что делает            |
| ------------ | ---------------------------- | ---------------- | --------------------- |
| **DML**      | Data Manipulation Language   | INSERT, UPDATE   | Changed data          |
| **DDL**      | Data Definition Language     | CREATE, DROP     | Changes the structure |
| **DCL**      | Data Control Language        | GRANT, REVOKE    | Changes rights        |
| **TCL**      | Transaction Control Language | COMMIT, ROLLBACK | Manages transactions  |

## NoSQL (Not Only SQL)

- Schema-less or flexible schema (documents, key-value, graph, etc.).
- **ref** (stores document id) - used to indicate **pseudo-relationships** between docs in collections.  
  Called **pseudo** because it does not have such strict constraints as in relational DBs.
- Good for scalability and handling unstructured data.
- Examples: **MongoDB (document)**, **Redis (key-value)**.

**Use case**:

- **SQL:** Banking system with strict relations and data integrity.
- **NoSQL:** Real-time analytics on clickstream data.

**Normalization**:

- Split data across multiple related tables to **reduce redundancy**.
- **Example**: Contacts in your phone: names in one place, phone numbers stored separately.
- **Normalize** for data consistency and storage efficiency.

**Denormalization**:

- Combine data into one table to optimize **read performance**.
- Example: Include address fields directly in the `users` table if you rarely update them.
- **Denormalize** for faster reads and simpler queries (e.g., analytics dashboards).

## Scaling

**Vertical Scaling (Scaling Up)**:

- Add more resources (CPU, RAM, SSD) to a single server.
- Simple to implement, limited by hardware capacity — eventually hits a ceiling.

**Horizontal Scaling (Scaling Out)**:

- Add more servers and distribute the data/workload.
- More complex but allows for much higher scalability.

## 📚 Database Scalability Concepts: Sharding and Replication

When a single database instance can no longer handle the load, we scale the system by distributing data **across multiple instances**.  
The two main approaches are **Sharding** and **Replication**.

### 🔹 Sharding (Horizontal Partitioning)

- Data is **split into slices (shards)** based on some criteria (e.g., user IDs, regions).
- **Each shard holds a portion of the data** and operates on its own database instance.
- Used to **scale write operations** and distribute data storage.

🧩 **Example**:

- **Shard 1** → Users with IDs from 1 → 10,000
- **Shard 2** → Users with IDs from 10,001 → 20,000

📌 **Pros**:

- Write operations can be distributed across shards.
- Each shard handles a smaller dataset → faster queries.

📌 **Cons**:

- Complex to manage and maintain.
- Cross-shard queries become harder.
- Requires a good sharding key to avoid data imbalance.

> 💡 **Sharding key** decides which partition request goes to so a **good sharding key** is `user_id`. Bad choise would be `created_at` since all new queries would go into one partition.

### 🔹 Replication (Master-Slave Model)

- One **Master** handles all **write operations**.
- One or more **Slaves (Replicas)** handle **read operations**.
- The Master replicates changes to the Slaves to keep them updated.

🧩 **Example Workflow**:

1. Insert new user → Goes to Master.
2. Read user profile → Served by a Slave (replica).

📌 **Pros**:

- Scales **read operations** effectively (which are typically ~80-90% of queries in web apps).
- Reduces load on the Master.
- Improves **fault tolerance**: if a Slave fails, others keep working.

📌 **Cons**:

- Writes are still limited by the capacity of a single Master.
- Data on Slaves might be **slightly stale** (due to replication lag).

**How to choose ?**

| Scenario          | Use Sharding                             | Use Replication                    |
| ----------------- | ---------------------------------------- | ---------------------------------- |
| High write load   | ✅ Yes                                   | ❌ No (writes still go to Master)  |
| High read load    | ✅ Sometimes (if data split makes sense) | ✅ Yes (scale reads with replicas) |
| Fault tolerance   | ✅ Yes (data spread across nodes)        | ✅ Yes (failover to replicas)      |
| Simplicity needed | ❌ Complex                               | ✅ Easier to implement             |

## PostgreSQL Scaling

**PG Vertical Scaling**

- PostgreSQL works well with vertical scaling.
- But you can only scale a single machine so far.

**PG Horizontal Scaling**

- PostgreSQL **does not** support automatic sharding out of the box !
- Sharding is usually implemented via **extensions** (e.g., `Citus`).
- Other config on application level is required when **sharding** PG:
  - Divide data based on a key (e.g., user ID).
  - Send reads/writes to appropriate server.

**PG Replication**

- _Replication_ = copy of the primary database to one or more read-only replicas.
- Great for read-heavy applications.
- Application must route **read queries** to replicas and **write queries** to the primary.
- `primary_conninfo` on a replica shows where the master is
- `standby_mode=on` tells us that this is a replica
- `pg_basebackup` copy DB from master when creating a replcia

## MongoDB Scaling

**Mongo Vertical Scaling**

- MongoDB is **designed with horizontal scaling in mind**.
- However, supports vertical scaling similarly to PostgreSQL.

**Mongo Horizontal Scaling (Sharding)**

- MongoDB **natively supports automatic sharding**:
  - Data is split across shards using a **shard key**.
  - A **mongos** query router handles routing.
  - **Config servers** store metadata about the cluster.
  - Each shard is deployed as a **replica set**, so every shard has its own primary and secondaries for redundancy.
- Shard key can be a **composite key** (multiple fields). This is often used

**Mongo Replication**

- MongoDB uses **replica sets** (1 Primary + N Secondary nodes).
- If Primary fails:
  - Secondaries hold an **election**.
  - Consider: node priority, latest data, MongoDB version.
  - **Quorum-based voting** elects the new Primary.
- Sharding and replication are configured through MongoDB's built-in tooling (`mongod`, `mongos`, `mongo/mongosh` shell commands, and replica set configuration commands);

## Summary about scaling

| Feature                      | PostgreSQL                   | MongoDB                    |
| ---------------------------- | ---------------------------- | -------------------------- |
| Vertical Scaling             | ✅ Good                      | ✅ Good                    |
| Horizontal Scaling           | ⚠️ Manual (no auto sharding) | ✅ Automatic via sharding  |
| Read Scaling via Replication | ✅ Yes                       | ✅ Yes                     |
| Auto Failover in Replication | ⚠️ Needs external tooling    | ✅ Built-in with elections |
| Shard Key Support            | ❌ No built-in               | ✅ Supports composite keys |

## Performance

**MongoDB can be faster** for some queries because:

- No joins or relationships — all related data (e.g. user, products) is usually one document.
- This avoids extra queries to merge data from other tables that in turn can also be large.

## Normalization vs Denormalization

//todo

## Normalization/Denormalization in MongoDB

- **Normalization:** separate data into individual documents that are related to each other.  
  This is convenient when data is frequently updated.

- **Denormalization:** duplicate related data in one document or collection.  
  This is convenient when data is frequently read but rarely updated.  
  **Side effects:** increase in document size and greater memory consumption.

## Mongoose

**Mongoose** is an Object Data Modeling (ODM) library for MongoDB and Node.js

- Adds schemas and structure to MongoDB's otherwise schema-less collections.
- Easier queries, defaults, hooks, validation, other built-in utils.

## Aggregate operations Mongoose

**Aggregation** - powerful way to process and transform data in MongoDB.

**`$match`**

Filters documents

```ts
await Model.aggregate([{ $match: { age: { $gt: 20 } } }]);
```

↪️ Returns documents where age > 20
Also supports `$lt`, `$eq`, etc.

**`$group`**
Groups documents by a field and **applies aggregations** (like count, sum, avg).

```ts
await Model.aggregate([{ $group: { _id: "$hobbies", count: { $sum: 1 } } }]);
```

↪️ Groups by hobbies and counts how many docs in each group.
Result:

```ts
{ _id: "reading", count: 2 }
{ _id: "sports", count: 2 }
{ _id: "music", count: 2 }
```

**`$unwind`**

```ts
await Model.aggregate([{ $unwind: "$hobbies" }]);
```

↪️ Each item in hobbies array becomes a separate document.

**Other useful aggregation operators**

- `$sort`: Sorts the documents.
- `$limit`: Limits number of returned documents.
- `$skip`: Skips a number of documents.
- `$sum`, `$avg`, `$min`, `$max`: Aggregates numbers.
- `$project`: Picks specific fields to return from each document.

## Aggregation: PostgreSQL vs MongoDB

Keep it simple — most SQL clauses map to a MongoDB aggregation stage:

- `WHERE` → `$match`
- `SELECT col AS ...` → `$project`
- `JOIN` → `$lookup` (left-outer join–like)
- `GROUP BY + COUNT/SUM/AVG/...` → `$group` + `$sum/$avg/...`
- `HAVING` → `$match` after `$group`
- `ORDER BY` → `$sort`
- `LIMIT / OFFSET` → `$limit` / `$skip`

> 💡 `$lookup` is the Mongo way to “join” collections during reads.

### Tiny example

PostgreSQL (active users with at least 1 order, top 5 by count):

```sql
SELECT u.id, COUNT(o.id) AS orders
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.active = true
GROUP BY u.id
HAVING COUNT(o.id) > 0
ORDER BY orders DESC
LIMIT 5;
```

MongoDB (same idea via pipeline starting from `users`):

```js
db.users.aggregate([
  { $match: { active: true } },
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "user_id",
      as: "orders",
    },
  },
  { $addFields: { ordersCount: { $size: "$orders" } } },
  { $match: { ordersCount: { $gt: 0 } } },
  { $project: { orders: 0 } },
  { $sort: { ordersCount: -1 } },
  { $limit: 5 },
]);
```

## Database Access and Tools in Node.js

### ORM (Object-Relational Mapping)

ORM provides an abstraction layer over the database, allowing us to interact with DB in an object-oriented paradigm.  
You define models, those map to database tables, and then these models are used to perform DB operations.

**✅ Pros:**

- **Abstraction**: Hides complex SQL queries, allowing you to work with database entities as objects.
- **Productivity**: Speeds up development with less boilerplate code.
- **Security**: Helps prevent SQL injection attacks by sanitizing inputs.
- **Maintainability**: Easier to refactor and maintain code, especially in large projects.
- **Database Agnostic (to some extent)**: Many ORMs support multiple database systems, allowing easier switching.

**🔴 Cons:**

- **Performance Overhead**: Can generate slower queries for complex operations, leading to n+1 query issues.
- **Limited SQL Control**: Hard to predict which SQL will be executed
- **ORM magic\***: Has it's own context, cache and so on.

**Popular Node.js ORMs for SQL Databases**

- **Prisma**: Modern, simple. Generates a type-safe client.
- **TypeORM**: More established, well-recognized. Emphasizes TypeScript and decorators.
- **Sequelize**: robust with strong transaction support and migrations.

> 💡 ORMs for Node.js are promise-based

### Lightweight SQL Builders

SQL builders offer a more programmatic way to construct SQL queries compared to writing raw SQL. One important difference is that they do not provide object mapping like an ORM; instead, they offer a fluent API to build queries, allowing for greater control over the generated SQL.

**✅ Pros:**

- **Control**: Offers more control over the generated SQL compared to ORMs.
- **Flexibility**: Easier to write complex or database-specific queries that might be cumbersome in an ORM.
- **Readability**: Programmatic construction of queries can be more readable than concatenated raw SQL strings.
- **Security**: Helps prevent SQL injection through parameter binding.

**Popular Node.js SQL Builders:**

- `pgtyped`: Generates types from SQL queries for type-safe raw SQL with PostgreSQL.
- `Kysely`
- `Knex.js`

### Raw Inline SQL

Raw inline SQL involves writing and executing SQL queries directly as strings in your application code.  
At least an SQL builder is used nowdays.

**✅ Pros**

- **Full Control** – Direct access to all SQL features and optimizations.
- **Performance** – No abstraction overhead.
- **Simplicity** – Convenient for small, straightforward queries.

**🔴 Cons:**

- **Security**– Must use parameterized queries to avoid SQL injection.
- **Maintainability** – Harder to refactor and debug in large codebases.
- **Vendor Lock-In** – Queries tightly coupled to one SQL dialect.

# PostgreSQL, MongoDB concepts

**PostgreSQL** is a powerful open-source **relational database system**:

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

-- Создаем промежуточную таблицу "student_courses" для связи между студентами и курсами
CREATE TABLE student_courses (
  student_id INT REFERENCES students(student_id),
  course_id INT REFERENCES courses(course_id),
  PRIMARY KEY (student_id, course_id)
);
```

## Relationships: PostgreSQL vs MongoDB

> 💡 TLDR: `PostgreSQL` ensures links with _FKs_; `MongoDB` models links with
> embedded docs or stored `_id` references and resolves them via `$lookup` or separate queries.

### PostgreSQL (foreign keys + joins)

- Uses **foreign keys** to enforce referential integrity.
- Reads combine data with **JOINs**; the planner uses indexes to keep it fast.
- Supports cascade rules: `ON DELETE/UPDATE CASCADE | RESTRICT | SET NULL`.
- Many-to-many is modeled via a **junction table** (two FKs, composite PK).

### MongoDB (embed vs reference)

- Uses two main patterns:
  - **Embedded documents** (nested in parent)
  - **References** (store another document's `_id`)
- You can join at read time with **`$lookup`** in an aggregation pipeline.

**When to embed**

- One-to-few, data is **read together** most of the time.
- Data **changes together**; keep a single write (atomic per document).

**When to reference**

- One-to-many with **unbounded growth** (`comments`, `events`, `logs`).
- Many-to-many (`users ↔ roles`, `products ↔ categories`).
- The same child is shared by many parents (avoid duplication).

## Normalization

**Normalization** reduces data redundancy by organizing data into multiple related tables.  
This is done using **decomposition**—splitting larger tables into smaller ones and defining relationships between them.

### 1NF (First Normal Form)

**Before 1NF**

| StudentID | Name  | Phones       |
| --------- | ----- | ------------ |
| 1         | Alice | 12345, 67890 |
| 2         | Bob   | 55555        |

**Why normalize?**  
Each field should contain only atomic values.  
A single cell should not hold multiple values. Queries on individual phone numbers become difficult.

**After 1NF** (atomic values only):

| StudentID | Name  | Phone |
| --------- | ----- | ----- |
| 1         | Alice | 12345 |
| 1         | Alice | 67890 |
| 2         | Bob   | 55555 |

### 2NF (Second Normal Form)

Must be in 1NF.

**Before 2NF** (partial dependency on composite key):

| StudentID | Course | Professor |
| --------- | ------ | --------- |
| 1         | Math   | John      |
| 1         | Bio    | Mary      |
| 2         | Math   | John      |
| 2         | Bio    | Mary      |

**Why normalize?**  
The key is (StudentID, Course), but `Professor` depends only on `Course`, not on the full composite key → **partial dependency**.

**After 2NF** (split into two tables):

**StudentsCourses**:

| StudentID | Course |
| --------- | ------ |
| 1         | Math   |
| 1         | Bio    |
| 2         | Math   |
| 2         | Bio    |

**Courses**:

| Course | Professor |
| ------ | --------- |
| Math   | John      |
| Bio    | Mary      |

### 3NF (Third Normal Form)

Must be in 2NF.

**Before 3NF** (transitive dependency):

| StudentID | Course | Professor | Department |
| --------- | ------ | --------- | ---------- |
| 1         | Math   | John      | Science    |
| 1         | Bio    | Mary      | Science    |
| 2         | Math   | John      | Science    |
| 2         | Bio    | Mary      | Science    |

Here composite key --> (StudentID, Course)

**Why normalize?**  
`Department` depends on `Professor`, not on the key (StudentID, Course) → **transitive dependency** between non-key columns.

**After 3NF** (split transitive dependency):

**StudentsCourses**:

| StudentID | Course |
| --------- | ------ |
| 1         | Math   |
| 1         | Bio    |
| 2         | Math   |
| 2         | Bio    |

**Courses**:

| Course | Professor |
| ------ | --------- |
| Math   | John      |
| Bio    | Mary      |

**Professors**:

| Professor | Department |
| --------- | ---------- |
| John      | Science    |
| Mary      | Science    |

> 🔎 Beyond 3NF (like BCNF, 4NF, 5NF...) are rarely used in dev.

## Joins

| JOIN Type    | Concept        | What It Does                                                                   |
| ------------ | -------------- | ------------------------------------------------------------------------------ |
| `INNER JOIN` | 🔍 Filter      | Returns **only rows that have matching values in both tables**                 |
| `LEFT JOIN`  | ➕ Expansion   | Returns **all rows from the left table**, and matching rows from the right     |
| `RIGHT JOIN` | ➕ Expansion   | Returns **all rows from the right table**, and matching rows from the left     |
| `FULL JOIN`  | 🌀 Combination | Returns **all rows when there is a match in one of the tables**, left or right |
| `CROSS JOIN` | 🔁 Cartesian   | Returns **all possible combinations** of rows from both tables                 |

---

### 🔍 INNER JOIN

**Table A**

| id  | name  |
| --- | ----- |
| 1   | Alice |
| 2   | Bob   |
| 3   | Carl  |

**Table B**

| id  | score |
| --- | ----- |
| 2   | 80    |
| 3   | 90    |
| 4   | 85    |

```sql
SELECT A.id, A.name, B.score
FROM A
INNER JOIN B ON A.id = B.id;
```

**Result:**

| id  | name | score |
| --- | ---- | ----- |
| 2   | Bob  | 80    |
| 3   | Carl | 90    |

---

### ➕ LEFT JOIN

**Table A**

| id  | name  |
| --- | ----- |
| 1   | Alice |
| 2   | Bob   |
| 3   | Carl  |

**Table B**

| id  | score |
| --- | ----- |
| 2   | 80    |
| 3   | 90    |
| 4   | 85    |

```sql
SELECT A.id, A.name, B.score
FROM A
LEFT JOIN B ON A.id = B.id;
```

**Result:**

| id  | name  | score |
| --- | ----- | ----- |
| 1   | Alice | NULL  |
| 2   | Bob   | 80    |
| 3   | Carl  | 90    |

---

### ➕ RIGHT JOIN

**Table A**

| id  | name  |
| --- | ----- |
| 1   | Alice |
| 2   | Bob   |
| 3   | Carl  |

**Table B**

| id  | score |
| --- | ----- |
| 2   | 80    |
| 3   | 90    |
| 4   | 85    |

```sql
SELECT A.id, A.name, B.score
FROM A
RIGHT JOIN B ON A.id = B.id;
```

**Result:**

| id  | name | score |
| --- | ---- | ----- |
| 2   | Bob  | 80    |
| 3   | Carl | 90    |
| 4   | NULL | 85    |

---

### 🌀 FULL JOIN

**Table A**

| id  | name  |
| --- | ----- |
| 1   | Alice |
| 2   | Bob   |
| 3   | Carl  |

**Table B**

| id  | score |
| --- | ----- |
| 2   | 80    |
| 3   | 90    |
| 4   | 85    |

```sql
SELECT A.id, A.name, B.score
FROM A
FULL JOIN B ON A.id = B.id;
```

**Result:**

| id  | name  | score |
| --- | ----- | ----- |
| 1   | Alice | NULL  |
| 2   | Bob   | 80    |
| 3   | Carl  | 90    |
| 4   | NULL  | 85    |

---

### 🔁 CROSS JOIN

**Table A**

| id  | name  |
| --- | ----- |
| 1   | Alice |
| 2   | Bob   |

**Table B**

| id  | score |
| --- | ----- |
| 10  | 80    |
| 20  | 90    |

```sql
SELECT A.name, B.score
FROM A
CROSS JOIN B;
```

**Result:**

| name  | score |
| ----- | ----- |
| Alice | 80    |
| Alice | 90    |
| Bob   | 80    |
| Bob   | 90    |

## Transaction

**Transaction** - group several operations into one atomic operation. Either all succeed or nether do.  
It is a fundamental operation in a database.

**Example:**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

If anything **goes wrong:**
`sql ROLLBACK; `

## ACID vs BASE in Databases

### ✅ ACID

Used in SQL, supported in MongoDB with transactions

**ACID** stands for:

- **A**tomicity – all or nothing: if one part fails, everything rolls back
- **C**onsistency – database always moves from one valid state to another
- **I**solation – parallel transactions don’t interfere with each other
- **D**urability – once committed, changes persist even after crashes

### Examples of ACID-safe operations:

- Transferring money between bank accounts (`UPDATE account1 - $100`, `UPDATE account2 + $100`)
- Making a hotel reservation (only commit if all data—room, payment, etc.—is valid)
- Updating multiple related tables (e.g. orders and inventory)

**💸 Financial operations (like money transfers) must be ACID!**

### 🌍 BASE

**BASE** is used in distributed NoSQL systems like MongoDB, Cassandra

**BASE** stands for:

- **B**asically Available – the system responds (but not always with up-to-date data)
- **S**oft State – system state may change over time without input
- **E**ventual Consistency – data becomes consistent _eventually_, not immediately

### Examples of BASE-style operations:

- Adding a comment or like in a social media app
- Updating a shopping cart in a high-traffic e-commerce site
- Writing to analytics logs

**📄 Working with Google Docs is BASE-style**: changes sync across devices eventually.

---

### ⚖️ ACID vs BASE: Key Differences

| Feature        | ACID                                                                                     | BASE                         |
| -------------- | ---------------------------------------------------------------------------------------- | ---------------------------- |
| Consistency    | Strong                                                                                   | Eventual                     |
| Availability   | May sacrifice for consistency                                                            | High                         |
| Suitable for   | Transactions, money, bookings                                                            | Big data, caching, analytics |
| Use in SQL     | Fully supported (via `BEGIN`, `COMMIT`)                                                  | Not typical                  |
| Use in MongoDB | **ACID for single documents**<br>**BASE for multi-document ops (unless in transaction)** | Yes                          |

### 🧠 Real-World Examples

| Operation                     | ACID or BASE? | Notes                                   |
| ----------------------------- | ------------- | --------------------------------------- |
| Transferring money            | ✅ ACID       | Needs atomicity and consistency         |
| Editing Google Doc            | 🌍 BASE       | Edits sync eventually                   |
| Reserving a flight seat       | ✅ ACID       | Must avoid double booking               |
| Adding a product view counter | 🌍 BASE       | Slight delays are acceptable            |
| Library book database         | ✅ ACID       | Updates must be reliable and consistent |

---

> 💡 **MongoDB is ACID only at single-document level by default**.  
> 💡 To get ACID for multiple documents, you need to explicitly use **multi-document transactions** (requires replica set).

**PostgreSQL** and other relational DBs fully support ACID via transactions:

```sql
BEGIN;
UPDATE books SET status = 'borrowed' WHERE id = 123;
UPDATE users SET active_loans = active_loans + 1 WHERE id = 456;
COMMIT;
```

## Transaction Problems and Phenomena (ACID Issues)

### 🔄 Transaction Execution

- **Sequential execution**: One transaction finishes before the next begins. **Safe but slow**.
- **Parallel execution**: Multiple transactions run at the same time. Faster, but **can introduce conflicts**.

**Databases like PostgreSQL and MongoDB allow parallel execution** with safeguards (isolation levels, locks).  
If transactions touch **the same data**, they are executed **sequentially**, using locks.

### ⚠️ Transaction Anomalies (Phenomena)

These issues can occur when transactions run **in parallel**:

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

**Problem**: A transaction reads data modified by another **uncommitted** transaction.

**Why it happens**: Reading "unconfirmed" changes.

---

### 3️⃣ Non-repeatable Read

**Real-world example**:  
A library system loads a book's status → later in the same transaction, the book appears "borrowed."

**Problem**: A transaction reads the same row twice, but gets different results due to another committed transaction in between.

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

Hope for the best

**Google Docs analogy**:  
Everyone can edit freely. When you click save, Google checks if someone else has changed the same paragraph.  
If yes, you get a **conflict warning**.

??? is it a db feature or do we have to check something on the application level ?

- No locking during read
- Conflict check before write
- **Efficient but may rollback on conflict**

## Deadlocks

Two or more transactions mutually block each other.  
PostgreSQL automatically detects deadlocks and aborts one of the transactions.  
Throws an error that should be handled by the application.

## 📚 Indexes in Databases (PostgreSQL + MongoDB)

Indexes are special data structures that help databases **search faster** by avoiding full table scans.

Without an index:  
🔍 The database checks **every row** (slow).

With an index:  
📖 The database checks an **index** (like a book’s table of contents), finds the value, and jumps straight to the correct row.

### 🛠 What is an Index?

**Index** - is a separate data structure.

- Stores **sorted values** from one or more columns
- Contains **pointers** to the full row
- Speeds up **SELECT**, **WHERE**, **ORDER BY**, and **JOIN** operations

🔴 Cons:

- Every **INSERT / UPDATE / DELETE** must also **update the index**
- Indexes **consume disk space**
- May slow down **write-heavy** workloads

### 🧠 Common Index Types in PostgreSQL

| Type    | Description                                               |
| ------- | --------------------------------------------------------- |
| B-tree  | 📌 Default, great for most queries using `<`, `=`, `>`    |
| Hash    | ⚠️ Only for `=` comparisons, rarely used                  |
| GiST    | 📍 For geometric and full-text search                     |
| SP-GiST | 🧭 Supports non-balanced trees, advanced use cases        |
| GIN     | 📚 Good for indexing arrays, JSONB, and full-text search  |
| BRIN    | 📦 Efficient for large, ordered tables (e.g. time series) |

> 💡 **JSONB** — a special PostgreSQL data type that stores JSON in binary form, allowing efficient search, filtering, and indexing without requiring a fixed schema.

> 💡 **B-tree** is a _balanced_ tree, NOT binary tree. Complexity of search through it is `O(log N)`

> 💡 Index does not make sense with **low-cardinality** such as sex male / female where each value still matches ~50% of table. Index helps when you skip large portion of rows.

### 🔧 How to Create Indexes

1️⃣ **Single Column**

```sql
CREATE INDEX idx_email ON users (email);
```

- Creates B-tree **by default**. Most common index
- Improves filtering, sorting, and joining on that column, `=`, `<`, `>`, `BETWEEN`, and `ORDER BY`

2️⃣ **Composite Index (compound index)**

For multiple columns

```sql
CREATE INDEX idx_status_amount ON orders (status, total_amount);
```

⚠️ **Order matters** in composite indexes:

- Index is built left-to-right.
- It will only be used if the first column is filtered.
- ✅ `sql WHERE status = 'paid'` → index is used
- ✅ `WHERE status = 'paid' AND total_amount > 100` → index is used
- ❌ `WHERE total_amount > 100` → index not used (because status is not used in WHERE)

> 💡 Think of it an address book sorted by **last name** first, then **first name**. You can't efficiently look it up by just **first name**.

3️⃣ **Unique Index**

```sql
CREATE UNIQUE INDEX idx_unique_email ON users (email);
```

Ensures that all values in the indexed column are unique.
If you try to insert a duplicate value, the database will throw an error.

Use cases:

- Enforce uniqueness for username, email, phone, etc.

4️⃣ **Expression-based Index**

```sql
CREATE INDEX idx_lower_email ON users (LOWER(email));
```

This creates an index on the result of an **expression**, not the raw column.

Use case:

- Case-insensitive search:

```sql
SELECT * FROM users WHERE LOWER(email) = 'example@mail.com';
```

5️⃣ **Partial Index**

Use for queries on a **subset** of data. Saves space + **improves write performance**.

```sql
CREATE INDEX idx_large_orders ON orders (total_amount) WHERE total_amount > 1000;
```

Reduces overhead for less-used rows.

### ⏳ Locking and Index Creation in PostgreSQL

By default, creating an index **locks the table for writes**:

```sql
CREATE INDEX idx_email ON users (email); -- ❌ Writes are blocked
```

### Non-blocking with `CONCURRENTLY`:

```sql
CREATE INDEX CONCURRENTLY idx_email_concurrent ON users (email);
```

- ✅ No write lock (reads and writes still happen)
- ❌ Slower index creation
- ❌ Can't be used for unique indexes

### 🧩 Indexing in MongoDB

Same principle, different syntax.

### Single-fiend index

```ts
db.users.createIndex({ email: 1 });
```

### Composite index:

```ts
db.orders.createIndex({ status: 1, totalAmount: -1 });
```

- Like PostgreSQL, **order matters**.
- Index is only efficient if the **first field** (status) is in the filter condition.

## PostgreSQL Join strageties

`PostgreSQL` uses different join algorithms depending on

- table size
- indexes
- cost estimation.

We need to be aware of those to debug out why some *JOIN*s suddenly become slow.

### 🔹 1. Nested Loop Join

Good when:

- Left table = small (e.g., 5 rows)
- Right table = large but indexed

**Real-life analogy:**
You have a **short guest list** → for each name you look up their **phone number in a phonebook** using the index.

### 🔹 2. Hash Join

Good when:

- Both tables _are large_
- No usable index
- Equality join `=`

**Real-life analogy**
You need to join "payments" table, each payment with `order_id` and orders table like so `sql ... payment.order_id = order.id`

### 🔹 3. Merge Join

Good when:

- Two sorted tables
- Not yet sorted but sorting is cheap

**Real-life analogy**
Two sorted customer lists → you run through each once, aligning matches.

## Planner

Built in mechanism in **DBMS** that tells you an "execution plan" for your query before it runs.

## Quick guide on planner settings

### **🧩 Join Algorithms**

Controls how PostgreSQL **joins multiple tables**.

**🔹 `enable_nestloop` (Default: ON)**

- Simple loop: For each row in table A, check every row in table B.
- **Example:** Finding recent comments by a few selected users.

**🔹 `enable_hashjoin`**

- Build a hash table from one table and probes it for matches.
- **Example:** Join orders and customers by customer_id when both tables are large

**🔹 `enable_mergejoin`**

- Sort both tables, merge. Fast when both are already sorted on join key
- **Example:** Joining monthly sales data sorted by date.

### 📚 Scan Strategies

Controls how PostgreSQL **reads data from tables**.

**🔹 `enable_seqscan` (Sequential Scan)**

- Reads **all rows** in the table.
- **Example:** any small table like settings list

**🔹 `enable_indexscan` (Index Scan)**

- **Uses index** to locate needed rows but reads rows info from table (heap)
- **Example:**

```sql
CREATE INDEX idx_users_email ON users (email);
SELECT * FROM users WHERE email = 'test@mail.com';
```

The index helps find the right row(s) fast.  
But because `SELECT *` needs all columns, PostgreSQL still must read the full row from the table.

**🔹 `enable_indexonlyscan` (Index-Only Scan)**

- Reads **only from index**. Does not read table at all. Fastest if all needed columns are in the index

```sql
CREATE INDEX idx_users_email ON users (email);
SELECT email FROM users WHERE email = 'test@mail.com';
```

- Query asks for only the email column, **which is already in the index**.

**🔹 `enable_bitmapscan` (Bitmap Index Scan)**

- Combine **multiple index scans** for complex filters
- Efficient when **multiple conditions** use different indexes

```sql
SELECT * FROM orders
WHERE status = 'shipped' AND total_amount > 100;
```

### 📏 Controlling Join Order

Controls how the planner chooses the **order of joining tables.**

**🔹 join_collapse_limit**

- Limits how many tables the **planner tries to reorder for join optimization.** Higher value = better plans but slower planning time.
- **Example:**

```sql
SET join_collapse_limit = 1; -- Forces planner to keep join order as written
```

**🔹 from_collapse_limit**

- Similar to `join_collapse_limit`, but applies to **subqueries and FROM clauses.**
- **Example:**

```sql
SET from_collapse_limit = 1; -- Prevents flattening of subqueries
```

## Query optimization

### 1. Changing the SQL Query Itself.

- Avoid unnecessary subqueries or joins.
- Use `LIMIT`, `EXISTS`
- Avoid `SELECT *`

### 2. Update Planner Statistics

- Run `ANALYZE` or `VACUUM ANALYZE` to update row count and data distribution stats.

### 3. Denormalization.

- Create temporary tables or materialized views to cache complex query results.
- Add indexes to support frequent queries

### 4. Tuning planner parameters.

- Force or disable specific join methods: `enable_nestloop`, `enable_hashjoin`, `enable_mergejoin`
- Enable/disable certain scan strategies: `enable_seqscan`, `enable_indexscan`, `enable_indexonlyscan`, `enable_bitmapscan`
- Influence how tables are joined: `join_collapse_limit`, `from_collapse_limit`

## 📚 Stored Procedures in PostgreSQL

A **Stored Procedure** is a set of SQL statements saved directly in the database.  
You can call it by name to perform repetitive or complex operations.

### ✅ Why Use Stored Procedures?

- Encapsulate business logic directly in the database.
- Reduce repetitive query code in your application.
- Can be used to group multiple SQL commands and even control transactions (since PostgreSQL 11).

### 📌 **Important Notes:**

- Stored Procedures support **transactions** starting from **PostgreSQL 11**.
- They are different from **Functions**:

  - **Procedures: Use `CALL`**, can manage transactions.
  - **Functions: Use `SELECT`**, must return a value.

### 🛠️ **Creating a Stored Procedure**

```sql
CREATE OR REPLACE PROCEDURE update_price(p_id INT, p_price DECIMAL)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE products
    SET price = p_price
    WHERE id = p_id;
END;
$$;
```

### Call a stored procedure

```sql
CALL update_price(1, 100.00);
```

**⚠️ Considerations:**

- Stored Procedures can **slightly slow down insert, update, or delete** operations due to additional processing layers.

- They **reduce portability between other DBMS** because they may not support them the same way.
- Use them when **logic really belongs inside the database**, otherwise **prefer application-level code** for portability.

## 📚 Triggers in PostgreSQL

A **Trigger** is a special database mechanism that automatically executes **before or after** specific events like `INSERT`, `UPDATE`, or `DELETE`.

### Why Use Triggers?

- Perform **automatic actions** when data changes.
- Useful for **auditing**, **logging**, or **notifications**.

### 🛠️ **How Triggers Work**

1. You define a **Trigger Function** (logic to execute).
2. You create a **Trigger** that calls this function on certain events.

**Trigger function:**

```sql
CREATE OR REPLACE FUNCTION notify_change()
RETURNS TRIGGER AS $$
BEGIN
    NOTIFY my_channel, 'Data changed in table';
    RETURN NEW; -- Required for AFTER/BEFORE INSERT/UPDATE triggers
END;
$$;
```

**Trigger that uses it:**

```sql
CREATE TRIGGER my_trigger
AFTER INSERT OR UPDATE ON my_table
FOR EACH ROW
EXECUTE FUNCTION notify_change();
```

### Trigger scope

- `FOR EACH ROW` Executes once **for every affected row**.
- `FOR EACH STATEMENT` Executes once per SQL statement.

**Temporary Triggers** work until the end of session. Useful for debugging.

## EXPLAIN ANALYZE

| Command           | What It Does                                                 |
| ----------------- | ------------------------------------------------------------ |
| `EXPLAIN`         | Shows the **execution plan** without running the query.      |
| `EXPLAIN ANALYZE` | **Executes** the query and shows actual performance metrics. |

This command shows:

- Actual time of each step of a plan
- Actual rows that has been handled
- Number of loops

> 💡 So it is a tree of steps that can be analyzed to find a bottleneck

### 🚩 Common red flags to look for

> `**Seq Scan**` (Sequential Scan) in `EXPLAIN` means db searches **every row** of the table. Right choise for small tables, **almost always** means room for optimization otherwise

- sequential scan
- Huge difference between **estimated** vs actual rows
- **Nested Loop** with large datasets
- Seq Scan on large table with selective condition
- High cost + long actual time
- **Missing index** on join condition

## 💥 `EXPLAIN ANALYZE DROP DATABASE postgres`?

So it drops the database and shows you the time it took to do it

## Query Fingerprints

**Query fingerprint** — normalized SQL where literals are replaced with placeholders.

```sql
SELECT * FROM users WHERE id = 777 →  SELECT * FROM users WHERE id = ?
```

## N+1 Problem

Imagine one query fetches `N` records, and then for each record another query is executed — totaling `N+1` queries.  
This is bad and results in performance issues.

**Way to fix it**
Use JOIN + aggregate (`JOIN`, `GROUP BY`) to fetch everything in one query.

## Extra notes on lock types

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
