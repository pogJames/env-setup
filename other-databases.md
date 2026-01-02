## GraphQL
**PROBLEM**

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
## Parquet
