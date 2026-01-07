#🖱️ ClickHouse

## ⚡ What is ClickHouse?

**ClickHouse** is a fast, open-source **columnar database management system** designed for OLAP
**ClickHouse** is designed for fast reading.
Write performance is **traded for fast reads**

> **OLAP** (Online Analytical Processing).

**Use Cases:**
- **Analytics** dashboards
- **Real-time monitoring** (metrics, logs)
- **Event tracking at scale** (billions of rows)
- **User activity** (page views, activity, session time)

### 🔑 Key Features

**Columnar Storage**
- Stores data **by columns**, not rows
- Speeds up reading, filtering, and aggregation of large datasets

**Advanced Aggregation Functions**
- Built-in support for `quantiles`, `histograms`, `approximate counts`, etc.
- Not available in traditional row-based SQL systems

**Unique Data Types**
- `LowCardinality`: optimized for columns with repeated string values
- `Nullable`: supports optional fields

**Distributed Queries**
- Run a single query **across multiple nodes**
- Scales horizontally for big data processing

**Variety of indexes**
- primary and secondary indexes
- `MERGE TREE` engine family

**Supports variaty of formats**
- `JSON`
- `CSV`
- `TSV`
- `Parquet`
- And many other structured formats

> 💡 ClickHouse is optimized for **reading**, **not writing**. Writing involves buffering and merging to improve performance.

### 🔄 Notes about data insertion into ClickHouse

- **Buffering**
  - ClickHouse does NOT write rows directly to final storage.
  - Each client INSERT produces a data part.
  - Parts are first stored in **in-memory buffers** before hitting disk.

- **Merging**
  - MergeTree creates many small parts.
  - Background threads **merge parts into bigger, sorted, compressed ones**.
  - Improves read speed, but merges are **asynchronous and unpredictable**.

- **Batch Insertion**
  - Always insert **batches** (10–10k rows).
  - Small inserts create many parts → expensive merges.

> 💡 ClickHouse buffering does not eliminate the need for batching. If you send many small inserts, ClickHouse still creates many parts, merges fall behind, and performance degrades.