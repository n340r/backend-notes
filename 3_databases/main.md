# Database main

This is what **most** real-interview questions ask about. The key is to understand sinse **memorising**.  
Cmmit to prompting for examples if you do not understand something.

## 📚 Sharding vs Replication

<!-- todo: add partitioning -->

When a single database instance can no longer handle the load, we scale the system by distributing data **across multiple instances**.  
The two main approaches are **Sharding** and **Replication**.

### 🔹 Sharding (Split data)

- Data is **split into slices (shards)** based on some criteria (e.g., user IDs, regions).
- **Each shard holds a portion of the data** and operates on its own database instance.
- Used to **scale write operations** and distribute data storage.

🧩 **Example**:

- **Shard 1** → Users with IDs from 1 → 10,000
- **Shard 2** → Users with IDs from 10,001 → 20,000

📌 **Pros**:

- **Write operations** can be distributed across shards.
- Each shard handles a smaller dataset → faster queries.

📌 **Cons**:

- Complex to manage and maintain.
- **Cross-shard** queries become harder.
- Requires a good sharding key to avoid data imbalance.

> 💡 **Sharding key** decides which partition request goes to so a **good sharding key** is `user_id`. Bad choise would be `created_at` since all new queries would go into one partition.

### 🔹 Replication (Copy data)

**Master-Replica Model**

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

- **Writes are still limited** by the capacity of a single Master.
- Data on Slaves might be **slightly stale** (due to replication lag).

**How to choose ?**

| Scenario          | Sharding                                 | Replication                        |
| ----------------- | ---------------------------------------- | ---------------------------------- |
| High write load   | ✅ Yes                                   | ❌ No (writes still go to Master)  |
| High read load    | ✅ Sometimes (if data split makes sense) | ✅ Yes (scale reads with replicas) |
| Fault tolerance   | ✅ Yes (data spread across nodes)        | ✅ Yes (failover to replicas)      |
| Simplicity needed | ❌ Complex                               | ✅ Easier to implement             |

## 📈 Database Scaling

### Vertical Scaling

- Add **more resources** (CPU, RAM, SSD) to a single server.
- Simple to implement, limited by hardware capacity.

**Best for**: Postgres

### Horizontal Scaling

- Add **more nodes** and distribute the data/workload.
- More complex but allows for much higher scalability.

**Best for**: MongoDB since it allows this **out of the box**

## 📈 PostgreSQL Scaling

**✅ PG Vertical Scaling**

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
- `standby_mode=on` tells node that it is a replica
- `pg_basebackup` copy DB from master when creating a replcia

## 📈 MongoDB Scaling

**Mongo Vertical Scaling**

- Defeats the pros of Mongo sinse Mongo is designed with **horizontal scaling in mind**.
- Supports vertical scaling similarly to PostgreSQL.

**✅ Mongo Horizontal Scaling (Sharding)**

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

## Relationships: PostgreSQL vs MongoDB

> 💡 TLDR: `PostgreSQL` ensures links with **foreign keys**; `MongoDB` models links with embedded docs or stored `_id` references and resolves them via `$lookup` or separate queries.

### PostgreSQL (foreign keys + joins)

- Uses **foreign keys** to enforce referential integrity.
- Reads combine data with **JOINs**; the planner uses indexes to keep it fast.
- Supports cascade rules: `ON DELETE/UPDATE CASCADE | RESTRICT | SET NULL`.
- Many-to-many is modeled via a **junction table** (two FKs, composite PK).

<!-- todo: this needs example of foreigh key -->

<!-- todo: two FKs, composite PK, whatis PK here ? -->

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

## Joins

### 🔍 INNER JOIN (default join)

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

---

| JOIN Type    | Concept        | What It Does                                                                   |
| ------------ | -------------- | ------------------------------------------------------------------------------ |
| `INNER JOIN` | 🔍 Filter      | Returns **only rows that have matching values in both tables**                 |
| `LEFT JOIN`  | ➕ Expansion   | Returns **all rows from the left table**, and matching rows from the right     |
| `RIGHT JOIN` | ➕ Expansion   | Returns **all rows from the right table**, and matching rows from the left     |
| `FULL JOIN`  | 🌀 Combination | Returns **all rows when there is a match in one of the tables**, left or right |
| `CROSS JOIN` | 🔁 Cartesian   | Returns **all possible combinations** of rows from both tables                 |

## Aggregate functions, order of queries

**Common aggregate functions**:

- `COUNT`
- `SUM`
- `MIN`
- `MAX`

**Where they can be used**:

- `SELECT`
- `HAVING`
- ❌ **NOT** in `WHERE`

```sql
FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT
```

```sql
SELECT country, COUNT(*) AS users
FROM users
WHERE active = true -- filters rows
GROUP BY country
HAVING COUNT(*) > 100 -- filters groups
```

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

### 🔧 Postgres Indexes

1️⃣ **Single Column**

```sql
CREATE INDEX idx_email ON users (email);
```

- Creates B-tree **by default**. Most common index
- Improves filtering, sorting, and joining on that column, `=`, `<`, `>`, `BETWEEN`, and `ORDER BY`

2️⃣ **Composite Index (compound index)**

For multiple columns

```sql
CREATE INDEX idx_status_amount ON orders (use_id, city, status);
```

TODO: order of index - matters. order of select, where - does not.
what matters in a query is if you include leftmost elements.
/
index looks like this in memory:
/

```text
user_id →
  inside of it: city →
    inside of it: status
```

WHERE user_id --> works
WHERE user_id, city --> works
WHERE city, user_id --> still works, postgres can search index all the way from left to right
WHERE status, city, user_id --> still works
//
WHERE status -> does not work
WHERE city, status --> does not work

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
Reduces overhead for **less-used** rows.

```sql
CREATE INDEX idx_large_orders ON orders (total_amount) WHERE total_amount > 1000;
```

### 🔧 Basic index creation rules

Suppose you have this query

```sql
SELECT user_id, status, amount
FROM orders
WHERE
	user_id = 'uuid' AND
	city = 'Mexico' AND
	status = 'PENDING' AND
	amount < 1000 AND
	created_at > '2025-02-02'
ORDER BY created_at DESC
```

I'll explain for composite index which is B-tree for **multiple columns**.

Index is a **sorted tree** of **groups of columns**. I'll show you what i mean:

If your index is:

```sql
(user_id, city, status, created_at, amount)
```

Then it looks like this in memory:

```sql
first sorted by user_id
inside of it by city
inside of it by status
inside of it by created_at
insdie of it by amount
```

There is a rule for **creating B-tree index**

1. **Equality** (we jump into the right branch)
2. **Range** (we scan from X to Y)

We first select column to match exactly - `user_id`, `city`, `status`

Then decide on which **range** column to include first because searching by it will be most effective. We **consider**:

- `ORDER BY` usage in a query (Most important)
- Selectivity
- Typical **query patterns**

In our case we clearly use `... ORDER BY created_at DESC` so we place it **right after equality** columns.

Again, if there were not `ORDER BY` we think about **selectivity**. If `amount < 1000` **reduces 80%** or rows, then take amount.

Final index:

```sql
CREATE INDEX idx_orders_main
ON orders (user_id, city, status, created_at DESC, amount);
```

> 💡 Second range `amount` can **no longer be used** for tree navigaiton same as `created_at`. But can be used for filtering

### ⏳ Concurrent index creation

<!-- todo: feels like this should be renamed to creating index in a running app under load or something -->

`CONCURRENTLY` - Do not lock the table for writes when creating index.

```sql
CREATE INDEX CONCURRENTLY idx_email_concurrent ON users (email);
```

- ✅ No write lock (reads and writes still happen)
- ❌ Slower index creation
- ❌ Can't be used for unique indexes

### 🟢 Mongo Indexes

<!-- todo: add simple examples -->
<!-- todo: simplify to the ones actually used in prod -->

- **Single field** - speed up queries filtering or sorting by that field
- **Compound** - index on multiple fields; supports queries using the left-prefix of the index
- **Multikey** (arrays) - auto-created when indexing an array field
- **Text** - special index for a full-text search
- **Geospatial** - index for location stuff `$near`, `$geoWithin`
- **Hashed** - hashes the field, supports `=` only
- **Partial**- index document based on condition
- **Sparse** - index document only if certain field exists. legacy-ish

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

> $$ ... $$; is a pg dollar-quoting string delimiter. Everything between these `$$` tokens is a single string - the procedure body.

### Call a stored procedure

```sql
CALL update_price(1, 100.00);
```

**⚠️ Considerations:**

- Stored Procedures can **slightly slow down insert, update, or delete** operations due to additional processing layers.
- They **reduce portability** between other `DBMS` because they **may not support** them the same way.
- Use them when **logic really belongs inside the database**, otherwise **prefer application-level code** for portability.

## plpgsql (PL/pgSQL)

**plpgsql** - procedural language PostgreSQL

It isLike **SQL + IF/LOOP/variables**

Used for functions, triggers, procedures

## 📈 Query optimization

### MVCC

> 💡 MVCC - Multi-Version Concurrency Control

- Rows are **never updated in place**
- Update = new version, **old version kept**
- Readers don’t block writers, Writers don’t block readers

This enables:

- Non-blocking reads
- Snapshot isolation

### Vacuum, Analyze, Autovacuum

**Vacuum**

- Removes dead tuples
- Prevents table bloat
- Frees space for reuse (not OS)

**Analyze**

- Updates statistics
- Used by query planner

**Autovacuum**

- Runs **both automatically**
- Critical for Postgres health

They’re grouped because: Vacuum **cleans**, Analyze **informs planner**

### 🤔 EXPLAIN ANALYZE

| Command           | What It Does                                                 |
| ----------------- | ------------------------------------------------------------ |
| `EXPLAIN`         | Shows the **execution plan** without running the query.      |
| `EXPLAIN ANALYZE` | **Executes** the query and shows actual performance metrics. |

`EXPLAIN ANALYZE` command shows:

- Actual time of each step of a plan
- Actual rows that has been handled
- Number of loops

> 💡 So it is a tree of steps that can be analyzed to find a bottleneck

### 🤔 Explain in Mongo

```js
db.users.find({ email: "a@b.com" }).explain("executionStats");
```

- **IXSCAN** - index used
- **COLLSCAN** -full scan ❌

### 👣 Step by step query optimization

1️⃣ **Change the SQL Query Itself.**

- Avoid unnecessary subqueries or joins.
- Use `LIMIT`, `EXISTS`
- Avoid `SELECT *`

2️⃣ **Update Planner Statistics**

- Run `ANALYZE` or `VACUUM ANALYZE` to update row count and data distribution stats.

3️⃣ **Denormalization**

- Create temporary tables or materialized views to cache complex query results.
- Add indexes to support frequent queries

4️⃣ **Tuning planner parameters** (Rarely)

- Force or disable specific join methods: `enable_nestloop`, `enable_hashjoin`, `enable_mergejoin`
- Enable/disable certain scan strategies: `enable_seqscan`, `enable_indexscan`, `enable_indexonlyscan`, `enable_bitmapscan`
- Influence how tables are joined: `join_collapse_limit`, `from_collapse_limit`

### 🚩 Common red flags to look for in EXPLAIN output

> `**Seq Scan**` (Sequential Scan) in `EXPLAIN` means db searches **every row** of the table. Right choise for small tables, **almost always** means room for optimization otherwise

- sequential scan
- Huge difference between **estimated** vs actual rows
- **Nested Loop** with large datasets
- Seq Scan on large table with selective condition
- High cost + long actual time
- **Missing index** on join condition

### 💥 `EXPLAIN ANALYZE DROP DATABASE`?

Common joke since this command **drops the database** and outputs the time it took to do it..

## Query Fingerprints

**Query fingerprint** — normalized SQL where literals are replaced with placeholders.

```sql
SELECT * FROM users WHERE id = 777 -- normal SQL
SELECT * FROM users WHERE id = ? -- fingerprint
```

## 🚨 N+1 Problem

Imagine one query fetches `N` records, and then for each record another query is executed — totaling `N+1` queries.  
This is bad and results in performance issues.

**Way to fix it**
Use JOIN + aggregate (`JOIN`, `GROUP BY`) to fetch everything in one query.

## 🟢 MongoDB a few important things

### 🟢 Mongoose

**Mongoose** is an Object Data Modeling (ODM) library for MongoDB and Node.js

- Adds schemas and structure to MongoDB's otherwise schema-less collections.
- Easier queries, defaults, hooks, validation, other built-in utils.

### Mongo Terms that are good to know:

**write concern** - How many nodes must confirm a write.  
Larger number `{w: 1}`, `{w: "majority"}` means safer, slower

**read concern** - Which data you are allowed to read.  
`local` - fastest, `majority` - commited by mojority, `linearizable` - strongest

## 📚 Triggers in PostgreSQL

> 💡 Should be used with caution, many teams **don't use them**.

A **Trigger** is a special database mechanism that automatically executes **before or after** specific events like `INSERT`, `UPDATE`, or `DELETE`.

### Why Use Triggers?

- Perform **automatic actions** on `INSERT`, `UPDATE`, `DELETE`
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

## Aggregation: PostgreSQL vs MongoDB

> 💡 You are already familiar with SQL aggregation from **interactive tutorial** you did. I've never been asked mongo aggregation on an interview.

Lets **keep it simple** — most SQL clauses map to a MongoDB aggregation stage:

- `WHERE` → `$match`
- `SELECT col AS ...` → `$project`
- `JOIN` → `$lookup` (left-outer join–like)
- `GROUP BY + COUNT/SUM/AVG/...` → `$group` + `$sum/$avg/...`
- `HAVING` → `$match` after `$group`
- `ORDER BY` → `$sort`
- `LIMIT / OFFSET` → `$limit` / `$skip`

> 💡 `$lookup` is the Mongo way to “join” collections during reads.

### Most useful mongo aggregation commands

<!-- todo: $match, $group, $unwind, what else ? -->

<!-- todo: do i get it right that those: $set, deleteMany, drop, projection, any, all, elemMatch are just commands
and not related to Mongo aggregation ? -->

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
