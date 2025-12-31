# Non-Relational Database
### Problems
1. The Schema Rigidity Problem
> Today they send {sensor_id, temperature, timestamp}\
> Tomorrow you add humidity sensors that send {sensor_id, humidity, temperature, timestamp, battery_level}\
> The tables have become a mess...
2. The Scale Problem
> SQL databases scale vertically -> you buy bigger servers (But there's a ceiling)\
> Horizontal scaling (adding more machines) is difficult because of ACID transactions and JOINs
3. The Impedance Mismatch
> Your Python objects and React state are nested, hierarchical structures\
> Flattening them into normalized tables and reconstructing them with JOINs is so bad
### Solution
1. Schema Flexibility 
> Store documents as they are, Different documents in the same collection can have different fields. Your sensor data evolves without migrations.
2. Horizontal Scaling
> Data is distributed across nodes by design. Need more capacity? Add machines. The database handles sharding automatically.
3. Data Model Alignment
> Store data the way your application uses it. If your React dashboard needs a sensor with its recent readings, store it that way—no joins required

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
**Cheatsheet**
```
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017")
db = client["iot_monitoring"]
sensors = db["sensors"]

# Insert (no schema definition needed)
sensors.insert_one({
    "sensor_id": "TH485_001",
    "location": "warehouse_A",
    "readings": []
})

# Query
sensor = sensors.find_one({"sensor_id": "TH485_001"})

# Update - push to embedded array
sensors.update_one(
    {"sensor_id": "TH485_001"},
    {"$push": {"readings": {"temp": 24.1, "ts": datetime.utcnow()}}}
)

# Query with conditions
hot_sensors = sensors.find({
    "readings.temp": {"$gt": 30},
    "location": "warehouse_A"
})

# Aggregation pipeline (like SQL GROUP BY but more powerful)
avg_by_location = sensors.aggregate([
    {"$unwind": "$readings"},
    {"$group": {
        "_id": "$location",
        "avg_temp": {"$avg": "$readings.temp"}
    }}
])
```
### Redis
```
sensor:42:latest → {"temp": 23.5, "ts": "2025-01-15T10:05:00Z"}
sensor:42:status → "online"
alerts:pending → ["sensor_42", "sensor_17"]
```
```
import redis
import json

r = redis.Redis(host='localhost', port=6379)

# Cache latest reading (expires in 60s)
r.setex(
    "sensor:TH485_001:latest",
    60,
    json.dumps({"temp": 24.1, "ts": "2025-01-15T10:05:00Z"})
)

# Get it back
latest = json.loads(r.get("sensor:TH485_001:latest"))

# Pub/Sub (like your MQTT pattern!)
# Publisher
r.publish("sensor_updates", json.dumps({"sensor_id": "TH485_001", "temp": 24.1}))

# Subscriber (in another process)
pubsub = r.pubsub()
pubsub.subscribe("sensor_updates")
for message in pubsub.listen():
    if message["type"] == "message":
        data = json.loads(message["data"])
        print(f"Update: {data}")

# Sorted sets for time-series (scores = timestamps)
r.zadd("sensor:TH485_001:history", {
    json.dumps({"temp": 24.1}): 1705312800,  # Unix timestamp
    json.dumps({"temp": 24.3}): 1705312860,
})

# Get readings from last hour
now = time.time()
hour_ago = now - 3600
recent = r.zrangebyscore("sensor:TH485_001:history", hour_ago, now)
```
### Others — Know it exists
**Column-Family (Cassandra, HBase)**\
- Optimized for write-heavy time-series data across massive clusters
- Think Netflix tracking every play event, or industrial IoT with thousands of sensors.\
**Graph Databases (Neo4j)**\
- For highly connected data: social networks, recommendation engines, fraud detection\
- Relationships are first-class citizens

