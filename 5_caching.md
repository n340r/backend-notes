# Caching

## 📚 Caching Strategies Explained (Beginner-Friendly)

What you **need to understand** are concepts behind those names.  
There are **two questions to answer**

1. Who controls the cache ?

- App
- Cache system

2. When and how does data move between **cache** and **DB**

- on read
- on write
- sync
- async

Those names below are just **labels for common** combinations used in real-world to **simplify things**:

### TTL

Cache for a **fixed period of time** and then expire when it pases

---

### (Read) Cache-Aside

**Very popular** one

1. **App controls** the cache.
2. Cache miss -> read from DB.
3. Later decide if you need to cache it

---

### (Read) Read-Through

1. **Cache layer controls** the cache.
2. Cache Miss -> **cache itself** does its workflow, saves things how it wants, returns result

**Much less common** in simple apps, less customization

---

### (Write sync) Write-Through

1. Write to one, and then another **synchronously** (order is up to you)
2. After a write completes, cache and DB are **guaranteed to be consistent**

**Cache is consistent**. Used in finance, inventory, when **stale data is unecceptable**. Writes are slow

---

### (Write async) Write-behind / Write-back

1. App **writes to cache** first
2. Background worker **asynchronously** writes data to DB

**Cache is eventually consistent**, writes are fast

### Write-around

- Writes **go only to DB**, not cache.
- Cache is updated later on read.

**Avoid polluting cache** with rarely-read data

### Refresh-ahead

- Cache refreshes data **before TTL expires**

Used for **hot products**, currently **trending** searches

## 📊 **Summary Table**

| Strategy          | Read Speed | Write Speed | Consistency | Use Case             |
| ----------------- | ---------- | ----------- | ----------- | -------------------- |
| **Cache-Aside**   | Fast       | Normal      | Eventual    | API, product catalog |
| **Lazy Caching**  | Fast       | Normal      | Eventual    | Product prices       |
| **Write-Through** | Fast       | Slow        | Strong      | Financial data       |
| **Write-Behind**  | Fast       | Fast        | Eventual    | Analytics, logging   |

## 📖 Cache Invalidation and Best Practices

### 🚫 **Invalidation Strategies**

- **Direct Invalidation:**  
  Immediately remove or update cached data when the underlying data changes.

- **Time-Based Invalidation (TTL):**  
  Set an expiration time for cached data (e.g., using Redis TTL). After this time, data is automatically removed.

- **Lazy Invalidation:**  
  On each cache access, check if the data is still fresh. If not, refresh it before returning.

- **Tag-Based Invalidation:**  
  Assign tags to related cached data. Invalidate entire groups by removing data associated with a specific tag.

---

### 📌 **Caching Recommendations**

- **Cache Size Management:**  
  Ensure your cache has enough memory for important data.  
  _Tip:_ Monitor Redis memory usage to decide if scaling is needed.

- **Data Consistency Consideration:**  
  If strong consistency is required (e.g., financial transactions), be cautious with caching.  
  _Caching can lead to stale or inconsistent data if not handled carefully._

- **Optimize Complex Queries:**  
  Cache the results of heavy or frequently executed database queries.  
  _This reduces database load and improves application performance._

---

## 🗃️ PostgreSQL Level Caching

---

### 📚 **Built-In Caching Mechanism**

- PostgreSQL **automatically caches frequently accessed data and query results** using its internal memory structures.
- The primary component for this is the **Shared Buffer Cache**.

### 🔧 **Configuration Example:**

```plaintext
shared_buffers = 1GB
```

- **Recommended:** Set shared_buffers to **25-40% of total system RAM** for performance optimization.

### 📌 Additional Caching Strategies

**Materialized views** store the results of complex queries physically on disk.

```sql
CREATE MATERIALIZED VIEW my_view AS
SELECT product_id, SUM(sales) AS total_sales
FROM sales_data
GROUP BY product_id;
```

Refresh when needed:

```sql
REFRESH MATERIALIZED VIEW my_view;
```

## Pooling

Connection pooling is a technique that **reuses existing database connections** instead of creating and closing a new connection

Creating database connections is **resource-intensive and slow**.
A **"pool"** is a collection of **pre-established, ready-to-use connections**.

### 📌 **Real-World Example:**

- Without pooling:
  - Each API call opens a new DB connection. High load ➔ DB crashes.
- With pooling (e.g., using **PgBouncer**):
  - API reuses existing connections. Handles more requests with fewer open connections.

> Tools: `PgBouncer`, `Pgpool-II`

## Redis: Everyday Commands

- `SET`, `GET` Basic key-value storage

```
SET user:1 "Alex"
GET user:1 -> "Alex"
```

- `HSET` (Hashes) Store object-like structure

```
HSET user:1 name "Alex"
HSET user:1 age "24"

HGET user:1 name -> "Alex"
```

- `SETs` **unique unordered** values

```
SADD online_users 1 2 3
SADD online_users 2   # ignored (already exists)

SMEMBERS online_users -> {1,2,3}
```

- `ZADD` - `Sorted sets` Set + score
  `-1` - last elemtnt
  `-2` - second from the end
  `ZREVRANGE` - range in reverce order

```
ZADD leaderboard 100 user1
ZADD leaderboard 200 user2
ZADD leaderboard 150 user3

ZREVRANGE leaderboard 0 1 - user2, user3
```

- `Pub/Sub` Lightweight messaging

```
SUBSCRIBE orders
PUBLISH orders "order_created"
```

- `WATCH` / `WATCH2` Optimistic locking. Abort if someone accesses while i'm executing

```
WATCH balance
GET balance
SET balance 90
EXEC
```

> 💡 Resis can be configured (though not widely used) for pub/sub. `PUBLISH channel message` / `SUBSCRIBE channel`: lightweight notifications.

📌 Common patterns, examples

- Cache with TTL: `SETEX user:123 60 {json}`.
- Cache-if-absent (and simple lock): `SET lock:job abc EX 10 NX`.
- Rolling queue: `LPUSH q item`; worker uses `BRPOP q 5`.
- Leaderboard: `ZADD lb 1200 user:42`; top 10 via `ZREVRANGE lb 0 9`.
