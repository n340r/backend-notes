# Database extra stuff

This is for you to **read once** and have an idea about. Most of those things are **NOT** or **VERY rarely**
asked on real interviews. Your goal is to **know those exist** and be able to quickly prompt or look up here.

Those are sorted from **most usefull** to less usefull.

## 🗂️ Normalization

**Normalization** reduces data redundancy by organizing data into multiple related tables.  
This is done using **decomposition** - split larger tables **into smaller** ones and define relationships between them.

Examples are given for **relational** data but those conceps also exist in **NoSQL**. See below.

### 1NF (First Normal Form)

**Before 1NF**

| StudentID | Name  | Phones       |
| --------- | ----- | ------------ |
| 1         | Alice | 12345, 67890 |
| 2         | Bob   | 55555        |

**Why normalize?**  
It might turn **ugly** if we have, say **15 phone numbers** for Alice.  
Queries on individual phone numbers become **difficult**.
Each field should contain only **atomic values**.

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

## 🗂️ Denormalization

- Combine data into one table to optimize **read performance**.
- Example: Include address fields directly in the `users` table if you rarely update them.
- **Denormalize** for faster reads and simpler queries (e.g., analytics dashboards).

## 🗂️ Normalization/Denormalization in MongoDB

- **Normalization:** separate data into individual documents that are related to each other.  
  This is convenient when data is frequently updated.

- **Denormalization:** duplicate related data in one document or collection.  
  This is convenient when data is frequently read but rarely updated.  
  **Side effects:** increase in document size and greater memory consumption.

## 📂 PostgreSQL Extra

**Planner** - Built in mechanism in **DBMS** that tells you an "execution plan" for your query before it runs.

Below is a Quick guide on **planner settings**:

- Join Strategies
- Scan Strategies
- Join order

### PostgreSQL Join strageties

`PostgreSQL` uses different join algorithms depending on

- table size
- indexes
- cost estimation.

We need to be aware of those to debug out why some *JOIN*s suddenly become slow.

🔹 **1. Nested Loop Join**

`enable_nestloop` - **ON** by default

Good when:

- Left table = small (e.g., 5 rows)
- Right table = large but indexed

**Real-life analogy:**
You have a **short guest list** → for each name you look up their **phone number in a phonebook** using the index.

🔹 **2. Hash Join**

`enable_hashjoin`

Good when:

- Both tables _are large_
- No usable index
- Equality join `=`

**Real-life analogy**
You need to join "payments" table, each payment with `order_id` and orders table like so `sql ... payment.order_id = order.id`

🔹 **3. Merge Join**

`enable_hashjoin`

Good when:

- Two sorted tables
- Not yet sorted but sorting is cheap

**Real-life analogy**
Two sorted customer lists → you run through each once, aligning matches.

### PostgreSQL Scan Strategies

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

### PostgreSQL Controlling Join Order

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

## 🟢 Mongo Extra

### Mongo failover

If **primary drops** MongoDB replica sets automatically elect a new primary; writes pause briefly but data is preserved if a majority is available.

### Mongo operations

<!-- todo: this needs to be qualified and given more explanation -->

1. **update one field** without replacing the document

```js
// without $set - whole document replaced
db.users.updateOne({ _id: 1 }, { $set: { age: 31 } });
```

2. **delete documents with condition** deleteMany vs drop

```js
// deleteMany
db.users.deleteMany({ age: { $lt: 18 } });
```

- Removes matching documents
- **Keeps** collection
- **Keeps** indexes

```js
// drop
db.users.drop();
```

- Deletes entire collection
- Deletes indexes
- Faster
- Irreversible

3. **projection** select some fields from a document

```js
db.users.find(
  { age: { $gt: 18 } },
  { name: 1, email: 1, \_id: 0 }
)
```

4. **$in**

```js
tags: ["backend", "go", "db"];

// any match
{
  tags: {
    $in: ["go", "java"];
  }
}
```

5. **$all**

```js
tags: ["backend", "go", "db"];

{
  tags: {
    $all: ["backend", "go"];
  }
}
```

6. **$elemMatch**

```js
grades: [
  { score: 90, subject: "math" },
  { score: 85, subject: "english" }
]

// match complex condition on same element
{ grades: { $elemMatch: { score: { $gt: 88 }, subject: "math" } } }
```
