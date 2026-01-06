## Graph Database
**TL;DR**
```
Graph DB = Optimized for relationship traversal

SQL:   Rows + JOINs (slow for deep relationships)
Graph: Nodes + Edges (fast traversal at any depth)

Use when: Relationships are the main query pattern
Tool: Neo4j (dominant player)
Query language: Cypher
```
**PROBLEM**
```
SQL Problem: "Find friends of friends of friends"

SELECT DISTINCT f3.name
FROM friendships f1
JOIN friendships f2 ON f1.friend_id = f2.user_id
JOIN friendships f3 ON f2.friend_id = f3.user_id
WHERE f1.user_id = 123;

→ 3 JOINs, gets worse with each level
→ 6 degrees of separation = 6 JOINs = 💀

Graph DB: "Traverse 6 levels"

MATCH (me)-[:FRIEND*1..6]-(person)
WHERE me.id = 123
RETURN person.name

→ Same speed regardless of depth
```
**DATA MODEL**
- Nodes: Entities (User, Post, Product)
- Relationships: Connections with direction and type
- Properties: Data on both nodes and relationships
**Neo4j (The Main Player)**
```cypher
// Create nodes
CREATE (alice:User {name: "Alice", age: 28})
CREATE (bob:User {name: "Bob", age: 32})
CREATE (post:Post {title: "Graph DBs 101", date: "2024-01-15"})

// Create relationships
MATCH (a:User {name: "Alice"}), (b:User {name: "Bob"})
CREATE (a)-[:FOLLOWS {since: "2023-01-01"}]->(b)

MATCH (a:User {name: "Alice"}), (p:Post {title: "Graph DBs 101"})
CREATE (a)-[:WROTE]->(p)
```
**Querying (Cypher Language)**
```cypher
// Find who Alice follows
MATCH (alice:User {name: "Alice"})-[:FOLLOWS]->(friend)
RETURN friend.name

// Friends of friends
MATCH (alice:User {name: "Alice"})-[:FOLLOWS*2]->(fof)
RETURN DISTINCT fof.name

// Shortest path between two users
MATCH path = shortestPath(
  (a:User {name: "Alice"})-[*]-(b:User {name: "Dave"})
)
RETURN path

// Recommend friends (friends of friends I don't follow)
MATCH (me:User {name: "Alice"})-[:FOLLOWS]->(friend)-[:FOLLOWS]->(suggestion)
WHERE NOT (me)-[:FOLLOWS]->(suggestion) AND me <> suggestion
RETURN suggestion.name, COUNT(*) AS mutual_friends
ORDER BY mutual_friends DESC
LIMIT 5
```
**80/20 Use Cases**

| Use Case | Example | Why Graph Wins |
|----------|---------|----------------|
| **Social networks** | Friends, followers, mutual connections | Traversal is O(1) per hop |
| **Recommendations** | "People also bought", "You may know" | Pattern matching is natural |
| **Fraud detection** | Find suspicious transaction rings | Detect cycles/patterns easily |
| **Knowledge graphs** | Wikipedia links, company hierarchies | Arbitrary relationships |
| **Access control** | User → Role → Permission → Resource | Path-based authorization |
## GraphQL
```
REST Problem:

GET /users/123           → {id, name, email, address, phone, ...}
GET /users/123/posts     → [{id, title, content, date, ...}, ...]
GET /posts/456/comments  → [{id, text, author, ...}, ...]

→ 3 round trips
→ Over-fetching (got 20 fields, needed 3)
→ Under-fetching (need related data, make more requests)
```

GraphQL Solution:
```
POST /graphql
{
  user(id: 123) {
    name
    posts(limit: 5) {
      title
      comments(limit: 3) {
        text
      }
    }
  }
}
```
**Query(Read)**
```
# Get specific fields
query {
  user(id: "123") {
    name
    email
    posts {
      title
      createdAt
    }
  }
}

# Response - exactly what you asked
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com",
      "posts": [
        { "title": "Hello World", "createdAt": "2024-01-15" }
      ]
    }
  }
}
```
**Mutation**
```
mutation {
  createPost(input: { title: "New Post", content: "..." }) {
    id
    title
    createdAt
  }
}
```
## InfluxDB
**PROBLEM**
```
Every second, you're collecting:
- 100 servers reporting CPU, memory, disk
- 1000 IoT sensors reporting temperature
- 10000 events from your application

That's millions of timestamped points per day.
Traditional DBs choke on this write load + time-range queries.
```
**DATA MODEL**
```
MEASUREMENT (like a table)
  ├── TAGS (indexed metadata - low cardinality)
  │   └── host=server01, region=us-east
  ├── FIELDS (actual values - not indexed)
  │   └── cpu_usage=78.5, memory_used=4096
  └── TIMESTAMP (automatic, nanosecond precision)
      └── 2025-01-02T10:30:00.000000000Z
```
**WRITING DATA**
```
cpu,host=server01,region=us-east usage=78.5,temp=65 1704192600000000000
cpu,host=server02,region=us-west usage=45.2,temp=58 1704192600000000000
```
**QUERYING**
```
// Get average CPU per host over last hour, in 5-min windows

1. Flux Language (InfluxDB 2.0)
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> filter(fn: (r) => r._field == "usage")
  |> aggregateWindow(every: 5m, fn: mean)
  |> group(columns: ["host"])

2. InfluxQL (InfluxDB 3.0)
SELECT MEAN(usage) 
FROM cpu 
WHERE time > now() - 1h 
GROUP BY time(5m), host
```
### Key Features
```
✓ Automatic data retention policies (delete data older than 30 days)
✓ Continuous queries (pre-aggregate data in background)
✓ Downsampling (keep 1-sec data for 1 day, 1-min data for 30 days)
✓ Optimized for time-range scans
✓ Built-in compression for time-series
```
### Use Cases
- Server/infrastructure monitoring
- IoT sensor data
- Application metrics
- Financial tick data

## DuckDB
### The Problem It Solves
```
You have:
- A 5GB CSV file
- A bunch of Parquet files from your data lake
- JSON logs you want to analyze

You want to:
- Run complex SQL queries
- On your laptop
- Without setting up a server
- And it should be FAST
```
### What DuckDB Actually Is
**An embedded analytical database** - like SQLite, but optimized for analytics:
> DuckDB's superpower is running **fast SQL** on local data without a server.
```
SQLite  → Row-based   → Good for: apps, CRUD, single row lookups
DuckDB  → Column-based → Good for: aggregations, scans, analytics
```
### ARCHITECTURE
```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Traditional row-based DB:        DuckDB (columnar):           │
│  ┌─────┬─────┬─────┐             ┌─────────────────┐           │
│  │ id  │name │ age │             │ id: 1,2,3,4,5   │           │
│  ├─────┼─────┼─────┤             ├─────────────────┤           │
│  │  1  │Alice│ 25  │             │ name: A,B,C,D,E │           │
│  │  2  │Bob  │ 30  │             ├─────────────────┤           │
│  │  3  │Carol│ 28  │             │ age: 25,30,28...│           │
│  └─────┴─────┴─────┘             └─────────────────┘           │
│  Good for: get row by ID         Good for: SUM(age), AVG(age)  │
│  Bad for: aggregate queries      Scans only needed columns     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```
**USAGE**
```python
import duckdb

# Query CSV directly (no import needed!)
duckdb.sql("SELECT * FROM 'sales_data.csv' LIMIT 10").show()

# Query Parquet files (even from S3!)
duckdb.sql("""
    SELECT 
        product_category,
        SUM(revenue) as total_revenue
    FROM 's3://my-bucket/sales/*.parquet'
    GROUP BY product_category
    ORDER BY total_revenue DESC
""").show()

# Query multiple files with glob patterns
duckdb.sql("SELECT * FROM 'logs/2025-*.json'")

# Query pandas DataFrame directly
import pandas as pd
df = pd.read_csv("data.csv")
duckdb.sql("SELECT * FROM df WHERE amount > 1000")

# Persist to a database file (like SQLite)
con = duckdb.connect("my_analysis.db")
con.sql("CREATE TABLE sales AS SELECT * FROM 'sales.csv'")
```
**PERFORMANCE**
```python
# This CSV has 100 million rows, 5GB file

# Pandas: 45 seconds, 8GB RAM
df = pd.read_csv("huge_file.csv")
result = df.groupby("category").agg({"sales": "sum"})

# DuckDB: 3 seconds, 500MB RAM
result = duckdb.sql("""
    SELECT category, SUM(sales) 
    FROM 'huge_file.csv' 
    GROUP BY category
""").df()
```
## Polars
**What**
- Pandas replacement that's 10-50x faster. LMAO
```
Pandas  = DataFrame library (Python, single-threaded, eager)
Polars  = DataFrame library (Rust, multi-threaded, lazy)
```
**When to use**
- Data too big/slow for Pandas
- You prefer Python method chaining over SQL
**How to Use**
```python
import polars as pl

# Read
df = pl.scan_parquet("data.parquet")  # or scan_csv

# Transform (chain operations)
result = (
    df
    .filter(pl.col("year") == 2024)
    .group_by("category")
    .agg(pl.col("amount").sum())
    .sort("amount", descending=True)
    .collect()  # executes here
)
```
The 5 Operations You'll Use 80% of the Time
```python
df.filter(pl.col("x") > 10)                    # WHERE
df.select("col1", "col2")                       # SELECT columns
df.with_columns((pl.col("x") * 2).alias("y"))  # Add/modify column
df.group_by("cat").agg(pl.col("val").sum())    # GROUP BY
df.sort("col", descending=True)                 # ORDER BY
```

### Polars vs DuckDB
```
Same speed. Pick by preference:

Polars → df.filter(pl.col("x") > 10).select("a", "b")
DuckDB → "SELECT a, b FROM df WHERE x > 10"

Like Python chaining? → Polars
Like SQL? → DuckDB
```

## Parquet
**What**
> A file format (like CSV), but binary, columnar, and compressed.

**Why it exists**
```
CSV:     10 GB file, query 2 columns → reads all 10 GB
Parquet: 1 GB file,  query 2 columns → reads only those 2 columns
```
**When to use**
- Storing analytical data (logs, exports, data lake)
- Files > 100MB that you'll query repeatedly
- When you often need only some columns

**When NOT to use**
- Need human-readable (use CSV)
- Streaming writes (use database or Avro)
- Small files < 10MB (overhead not worth it)

**How to Use**
```python
import polars as pl

# Write
df.write_parquet("data.parquet")

# Read (lazy = fast, only loads what query needs)
result = (
    pl.scan_parquet("data.parquet")
    .filter(pl.col("year") == 2024)
    .select("name", "amount")
    .collect()
)

# Query with DuckDB (no loading!)
import duckdb
duckdb.sql("SELECT dept, SUM(sales) FROM 'data.parquet' GROUP BY dept")
```

**Why It's Fast**
1. Columnar: Reads only columns you need
2. Compression: 5-10x smaller than CSV
3. Statistics: Skips data blocks that can't match your filter

One Tip
```python
# Use zstd compression for best size/speed balance
df.write_parquet("data.parquet", compression="zstd")
```
