# Non-Relational Database
### Problems
1. **The Schema Rigidity Problem**
> Today they send {sensor_id, temperature, timestamp}\
> Tomorrow you add humidity sensors that send {sensor_id, humidity, temperature, timestamp, battery_level}\
> The tables have become a mess...
2. **The Scale Problem**
> SQL databases scale vertically -> you buy bigger servers (But there's a ceiling)\
> Horizontal scaling (adding more machines) is difficult because of ACID transactions and JOINs
3. **The Impedance Mismatch**
> Your Python objects and React state are nested, hierarchical structures\
> Flattening them into normalized tables and reconstructing them with JOINs is so bad
### Use Case
1. Caching(Redis): Cache DB query results, API responses
2. Sessions(Redis): Store user login sessions with auto-expiry
3. Flexible schema(MongoDB): User profiles with varying fields
4. Real-time counters(Redis): Page views, likes, online users
5. Rate limiting(Redis): API request limits per user
### Core Concepts You Must Know
1. **CAP Theorem**: You can only guarantee two of three: Consistency, Availability, Partition tolerance. Relational databases prioritize consistency. Most NoSQL systems let you choose—often favoring availability and partition tolerance, accepting "eventual consistency."
2. **Eventual Consistency**: After an update, given enough time with no new updates, all replicas will converge to the same value. For your temperature dashboard, showing a reading that's 100ms stale is usually fine.
3. **Denormalization**: Instead of normalizing data across tables, you duplicate it to avoid joins. Store the sensor name with every reading, even though it's redundant. Disk is cheap; joins across distributed systems are expensive.
4. **Sharding**: Data is split across machines by a shard key. Choose poorly (like sharding by sensor type when 90% are temperature sensors) and you get hot spots. Choose well (sensor_id) and load distributes evenly.

### MongoDB
```
{
  "_id": "sensor_42",
  "location": "warehouse_A",
  "type": "TH485",
  "config": {
    "polling_interval": 5000,
    "alert_threshold": 30
  },
  "recent_readings": [
    {"temp": 23.5, "humidity": 45, "ts": "2025-01-15T10:00:00Z"},
    {"temp": 23.7, "humidity": 44, "ts": "2025-01-15T10:05:00Z"}
  ]
}
```

### Redis
```
sensor:42:latest → {"temp": 23.5, "ts": "2025-01-15T10:05:00Z"}
sensor:42:status → "online"
alerts:pending → ["sensor_42", "sensor_17"]
```

### Others — Know it exists
**Column-Family (Cassandra, HBase)**\
- Optimized for write-heavy time-series data across massive clusters
- Think Netflix tracking every play event, or industrial IoT with thousands of sensors.\
**Graph Databases (Neo4j)**\
- For highly connected data: social networks, recommendation engines, fraud detection\
- Relationships are first-class citizens

### Code Examples
1. Caching (Most Common)
```python
# Before hitting database, check Redis
cached = redis.get("user:123")
if cached:
    return json.loads(cached)

user = db.query("SELECT * FROM users WHERE id = 123")
redis.setex("user:123", 3600, json.dumps(user))  # Cache 1 hour
```
2. Sessions
```python
# Login
redis.setex(f"session:{token}", 86400, json.dumps({"user_id": 123}))

# Check auth
session = redis.get(f"session:{token}")
```
3. Flexible Documents
```python
# MongoDB - each user can have different fields
db.users.insert_one({"name": "Alice", "age": 25})
db.users.insert_one({"name": "Bob", "company": "Acme", "skills": ["python"]})
```
4. Counters
```python
redis.incr("page:home:views")           # Increment by 1
redis.incrby("product:123:stock", -1)   # Decrement stock
```
5. Rate Limiting
```python
def is_limited(user_id):
    key = f"rate:{user_id}"
    count = redis.incr(key)
    if count == 1:
        redis.expire(key, 60)  # 60 second window
    return count > 100  # 100 requests per minute
```
### Decision Rule
```
Need speed + expiry?     → Redis
Need flexible documents? → MongoDB
Need transactions?       → SQL
```
