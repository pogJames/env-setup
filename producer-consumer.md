# Producer-Consumer Pattern & Inter-Process Communication (IPC)

## What is Producer-Consumer?

**Producer**: Creates/generates data
**Consumer**: Processes/uses data
**Buffer/Queue**: Holds data between them

```
Producer → [Queue/Buffer] → Consumer
```

**Real-world examples:**
- Sensor (producer) → Data buffer → Analysis (consumer)
- Web scraper (producer) → Queue → Database writer (consumer)
- Camera (producer) → Frame buffer → Display (consumer)

---

## Basic Producer-Consumer with Queue

### Simple Example - Same Rate

```python
import threading
import queue
import time

# Shared queue
data_queue = queue.Queue(maxsize=10)

def producer():
    """Produces items every second"""
    for i in range(10):
        item = f"Item-{i}"
        print(f"[Producer] Creating {item}")
        data_queue.put(item)
        time.sleep(1)  # Produce every 1 second
    
    # Signal end
    data_queue.put(None)
    print("[Producer] Finished")

def consumer():
    """Consumes items"""
    while True:
        item = data_queue.get()
        
        if item is None:  # End signal
            break
        
        print(f"[Consumer] Processing {item}")
        time.sleep(1)  # Process every 1 second
        data_queue.task_done()
    
    print("[Consumer] Finished")

# Run
producer_thread = threading.Thread(target=producer)
consumer_thread = threading.Thread(target=consumer)

producer_thread.start()
consumer_thread.start()

producer_thread.join()
consumer_thread.join()
```

---

## Different Rate Loops - Key Concept!

### Fast Producer, Slow Consumer

```python
import threading
import queue
import time

data_queue = queue.Queue(maxsize=5)  # Limited buffer

def fast_producer():
    """Produces 10 items per second"""
    for i in range(20):
        item = f"Data-{i}"
        print(f"[Producer] Creating {item} (queue size: {data_queue.qsize()})")
        data_queue.put(item)  # Blocks when queue is full!
        time.sleep(0.1)  # Fast: 10 items/second
    
    data_queue.put(None)
    print("[Producer] Done")

def slow_consumer():
    """Processes 1 item per second"""
    while True:
        item = data_queue.get()
        
        if item is None:
            break
        
        print(f"[Consumer] Processing {item} (queue size: {data_queue.qsize()})")
        time.sleep(1)  # Slow: 1 item/second
        data_queue.task_done()
    
    print("[Consumer] Done")

# Run
t1 = threading.Thread(target=fast_producer)
t2 = threading.Thread(target=slow_consumer)

t1.start()
t2.start()

t1.join()
t2.join()
```

**What happens:**
- Producer creates items fast (0.1s)
- Consumer processes slow (1s)
- Queue fills up → Producer blocks until space available
- This is **backpressure** - automatic rate limiting!

### Slow Producer, Fast Consumer

```python
import threading
import queue
import time

data_queue = queue.Queue()

def slow_producer():
    """Produces 1 item per 2 seconds"""
    for i in range(10):
        item = f"Data-{i}"
        print(f"[Producer] Creating {item}")
        data_queue.put(item)
        time.sleep(2)  # Slow: 1 item per 2 seconds
    
    data_queue.put(None)

def fast_consumer():
    """Can process instantly"""
    while True:
        print(f"[Consumer] Waiting... (queue size: {data_queue.qsize()})")
        item = data_queue.get(timeout=3)  # Wait up to 3 seconds
        
        if item is None:
            break
        
        print(f"[Consumer] Got {item}, processing instantly")
        data_queue.task_done()

t1 = threading.Thread(target=slow_producer)
t2 = threading.Thread(target=fast_consumer)

t1.start()
t2.start()

t1.join()
t2.join()
```

**What happens:**
- Consumer waits for data (queue empty)
- When data arrives, processes immediately
- Consumer is idle most of the time

---

## Multiple Producers, Multiple Consumers

```python
import threading
import queue
import time
import random

data_queue = queue.Queue()

def producer(producer_id, num_items):
    """Producer with random rate"""
    for i in range(num_items):
        item = f"P{producer_id}-Item{i}"
        delay = random.uniform(0.1, 0.5)
        
        print(f"[Producer-{producer_id}] Creating {item} (delay: {delay:.2f}s)")
        data_queue.put(item)
        time.sleep(delay)
    
    print(f"[Producer-{producer_id}] Finished")

def consumer(consumer_id):
    """Consumer processes until poison pill"""
    while True:
        try:
            item = data_queue.get(timeout=2)
            
            if item == "STOP":  # Poison pill
                data_queue.put("STOP")  # Re-add for other consumers
                break
            
            processing_time = random.uniform(0.1, 0.3)
            print(f"[Consumer-{consumer_id}] Processing {item} ({processing_time:.2f}s)")
            time.sleep(processing_time)
            data_queue.task_done()
            
        except queue.Empty:
            break
    
    print(f"[Consumer-{consumer_id}] Finished")

# Create 3 producers, 2 consumers
producers = [threading.Thread(target=producer, args=(i, 5)) for i in range(3)]
consumers = [threading.Thread(target=consumer, args=(i,)) for i in range(2)]

# Start all
for p in producers:
    p.start()
for c in consumers:
    c.start()

# Wait for producers
for p in producers:
    p.join()

# Send poison pill
data_queue.put("STOP")

# Wait for consumers
for c in consumers:
    c.join()

print("All done!")
```

---

## Signaling Multiple Concurrent Loops

### Method 1: Event - Broadcast Signal

```python
import threading
import time

# Shared event
start_event = threading.Event()
stop_event = threading.Event()

def loop_worker(worker_id, rate):
    """Worker loop running at specific rate"""
    print(f"[Worker-{worker_id}] Waiting for start signal...")
    start_event.wait()  # Block until signaled
    
    print(f"[Worker-{worker_id}] Started! Running at {rate}Hz")
    
    iteration = 0
    while not stop_event.is_set():
        print(f"[Worker-{worker_id}] Iteration {iteration}")
        iteration += 1
        time.sleep(1.0 / rate)  # Run at specified rate
    
    print(f"[Worker-{worker_id}] Stopped")

# Create workers with different rates
workers = [
    threading.Thread(target=loop_worker, args=(0, 10)),   # 10 Hz
    threading.Thread(target=loop_worker, args=(1, 5)),    # 5 Hz
    threading.Thread(target=loop_worker, args=(2, 2)),    # 2 Hz
]

# Start all workers (they wait)
for w in workers:
    w.start()

time.sleep(1)
print("\n[Main] Signaling START to all workers!")
start_event.set()  # Release all workers simultaneously

time.sleep(5)
print("\n[Main] Signaling STOP to all workers!")
stop_event.set()  # Stop all workers

for w in workers:
    w.join()

print("\nAll workers stopped")
```

### Method 2: Condition - Coordinated Signaling

```python
import threading
import time

condition = threading.Condition()
shared_state = {"ready": False, "data": None}

def producer_loop():
    """Producer generates data at 2 Hz"""
    for i in range(10):
        time.sleep(0.5)  # 2 Hz
        
        with condition:
            shared_state["data"] = f"Data-{i}"
            shared_state["ready"] = True
            print(f"[Producer] Generated: {shared_state['data']}")
            condition.notify_all()  # Wake all consumers

def consumer_loop(consumer_id):
    """Consumer waits for data"""
    while True:
        with condition:
            # Wait for data to be ready
            while not shared_state["ready"]:
                condition.wait()
            
            # Process data
            data = shared_state["data"]
            if data == "Data-9":  # Last item
                break
            
            print(f"[Consumer-{consumer_id}] Received: {data}")
            
            # Clear ready flag
            shared_state["ready"] = False

# Create threads
producer = threading.Thread(target=producer_loop)
consumers = [threading.Thread(target=consumer_loop, args=(i,)) for i in range(3)]

producer.start()
for c in consumers:
    c.start()

producer.join()
for c in consumers:
    c.join()
```

### Method 3: Barrier - Synchronization Point

```python
import threading
import time

# Barrier waits for N threads before releasing all
barrier = threading.Barrier(4)  # 3 workers + 1 coordinator

def synchronized_worker(worker_id):
    """Worker that synchronizes at each iteration"""
    for iteration in range(5):
        # Do work
        work_time = (worker_id + 1) * 0.2
        print(f"[Worker-{worker_id}] Working for {work_time}s...")
        time.sleep(work_time)
        
        print(f"[Worker-{worker_id}] Waiting at barrier (iteration {iteration})")
        barrier.wait()  # Wait for all workers
        print(f"[Worker-{worker_id}] Released from barrier!")

def coordinator():
    """Coordinates all workers"""
    for iteration in range(5):
        print(f"\n[Coordinator] Iteration {iteration} - waiting for all workers...")
        barrier.wait()  # Wait for all workers
        print(f"[Coordinator] All workers synchronized!\n")
        time.sleep(0.5)

# Create workers
workers = [threading.Thread(target=synchronized_worker, args=(i,)) for i in range(3)]
coord = threading.Thread(target=coordinator)

coord.start()
for w in workers:
    w.start()

coord.join()
for w in workers:
    w.join()
```

---

## Inter-Process Communication (IPC) Types

### 1. Queue (multiprocessing.Queue)

**Use case:** Safe data passing between processes

```python
import multiprocessing as mp
import time

def producer(queue):
    """Producer process"""
    for i in range(5):
        data = f"Item-{i}"
        print(f"[Producer] Sending: {data}")
        queue.put(data)
        time.sleep(1)
    
    queue.put(None)  # End signal

def consumer(queue):
    """Consumer process"""
    while True:
        data = queue.get()
        
        if data is None:
            break
        
        print(f"[Consumer] Received: {data}")

if __name__ == '__main__':
    # Create queue
    queue = mp.Queue()
    
    # Create processes
    p1 = mp.Process(target=producer, args=(queue,))
    p2 = mp.Process(target=consumer, args=(queue,))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```

### 2. Pipe (Two-way Communication)

**Use case:** Direct communication between two processes

```python
import multiprocessing as mp
import time

def process_a(conn):
    """Process A sends and receives"""
    for i in range(5):
        # Send
        msg = f"Hello from A-{i}"
        print(f"[Process A] Sending: {msg}")
        conn.send(msg)
        
        # Receive
        response = conn.recv()
        print(f"[Process A] Got response: {response}")
        
        time.sleep(1)
    
    conn.close()

def process_b(conn):
    """Process B receives and responds"""
    while True:
        try:
            # Receive
            msg = conn.recv()
            print(f"[Process B] Received: {msg}")
            
            # Send response
            response = f"ACK: {msg}"
            conn.send(response)
            
        except EOFError:
            break
    
    conn.close()

if __name__ == '__main__':
    # Create pipe (two-way)
    parent_conn, child_conn = mp.Pipe()
    
    p1 = mp.Process(target=process_a, args=(parent_conn,))
    p2 = mp.Process(target=process_b, args=(child_conn,))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```

### 3. Shared Memory (Value/Array)

**Use case:** Fast shared state between processes

```python
import multiprocessing as mp
import time

def producer(shared_value, shared_array):
    """Updates shared memory"""
    for i in range(10):
        # Update shared integer
        shared_value.value = i
        
        # Update shared array
        for j in range(len(shared_array)):
            shared_array[j] = i * (j + 1)
        
        print(f"[Producer] Updated: value={i}, array={list(shared_array)}")
        time.sleep(0.5)

def consumer(shared_value, shared_array):
    """Reads shared memory"""
    for i in range(10):
        # Read shared data
        val = shared_value.value
        arr = list(shared_array)
        
        print(f"[Consumer] Read: value={val}, array={arr}")
        time.sleep(0.5)

if __name__ == '__main__':
    # Create shared memory
    shared_value = mp.Value('i', 0)  # integer
    shared_array = mp.Array('i', [0, 0, 0, 0])  # integer array
    
    p1 = mp.Process(target=producer, args=(shared_value, shared_array))
    p2 = mp.Process(target=consumer, args=(shared_value, shared_array))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```

### 4. Manager (Shared Objects)

**Use case:** Share complex objects (dict, list) between processes

```python
import multiprocessing as mp
import time

def producer(shared_dict, shared_list):
    """Updates shared dict and list"""
    for i in range(5):
        # Update dict
        shared_dict[f'key{i}'] = f'value{i}'
        
        # Update list
        shared_list.append(i)
        
        print(f"[Producer] Dict: {dict(shared_dict)}")
        print(f"[Producer] List: {list(shared_list)}")
        time.sleep(1)

def consumer(shared_dict, shared_list):
    """Reads shared objects"""
    time.sleep(0.5)  # Let producer start first
    
    for i in range(5):
        print(f"[Consumer] Dict has {len(shared_dict)} items")
        print(f"[Consumer] List has {len(shared_list)} items")
        time.sleep(1)

if __name__ == '__main__':
    # Create manager
    manager = mp.Manager()
    shared_dict = manager.dict()
    shared_list = manager.list()
    
    p1 = mp.Process(target=producer, args=(shared_dict, shared_list))
    p2 = mp.Process(target=consumer, args=(shared_dict, shared_list))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
    
    print(f"\nFinal dict: {dict(shared_dict)}")
    print(f"Final list: {list(shared_list)}")
```

### 5. Lock (Synchronization)

**Use case:** Prevent race conditions in shared memory

```python
import multiprocessing as mp
import time

def increment_with_lock(shared_counter, lock, worker_id):
    """Safely increment counter"""
    for i in range(5):
        with lock:  # Acquire lock
            current = shared_counter.value
            print(f"[Worker-{worker_id}] Read: {current}")
            time.sleep(0.1)  # Simulate work
            shared_counter.value = current + 1
            print(f"[Worker-{worker_id}] Wrote: {shared_counter.value}")

if __name__ == '__main__':
    shared_counter = mp.Value('i', 0)
    lock = mp.Lock()
    
    # Create 3 workers
    workers = [
        mp.Process(target=increment_with_lock, args=(shared_counter, lock, i))
        for i in range(3)
    ]
    
    for w in workers:
        w.start()
    for w in workers:
        w.join()
    
    print(f"\nFinal counter: {shared_counter.value}")
    print(f"Expected: {3 * 5}")
```

### 6. Event (Process Signaling)

**Use case:** Signal events between processes

```python
import multiprocessing as mp
import time

def waiter(event, worker_id):
    """Wait for event"""
    print(f"[Worker-{worker_id}] Waiting for event...")
    event.wait()  # Block until set
    print(f"[Worker-{worker_id}] Event received! Starting work...")
    time.sleep(2)
    print(f"[Worker-{worker_id}] Done")

def coordinator(event):
    """Coordinate workers"""
    print("[Coordinator] Preparing...")
    time.sleep(3)
    print("[Coordinator] Signaling all workers!")
    event.set()  # Release all waiters

if __name__ == '__main__':
    event = mp.Event()
    
    # Create workers
    workers = [mp.Process(target=waiter, args=(event, i)) for i in range(3)]
    coord = mp.Process(target=coordinator, args=(event,))
    
    for w in workers:
        w.start()
    coord.start()
    
    for w in workers:
        w.join()
    coord.join()
```

---

## IPC Comparison Table

| IPC Type | Speed | Complexity | Use Case |
|----------|-------|------------|----------|
| **Queue** | Medium | Low | General message passing |
| **Pipe** | Fast | Low | Two-process communication |
| **Shared Memory** | Fastest | Medium | High-performance data sharing |
| **Manager** | Slow | Low | Complex shared objects |
| **Lock** | N/A | Low | Synchronization |
| **Event** | Fast | Low | Signaling |
| **Semaphore** | Fast | Low | Resource limiting |

---

## When to Use What?

**Queue:**
- ✅ Multiple producers/consumers
- ✅ Different rates
- ✅ Safe message passing

**Pipe:**
- ✅ Two processes only
- ✅ Bidirectional communication
- ✅ Simple request/response

**Shared Memory:**
- ✅ High-performance needs
- ✅ Large data sharing
- ⚠️ Need manual locking

**Manager:**
- ✅ Complex objects (dict, list)
- ✅ Easy to use
- ⚠️ Slower than raw shared memory

**Event:**
- ✅ Start/stop signals
- ✅ Broadcast to multiple processes
- ✅ Coordination

**Lock:**
- ✅ Protect shared resources
- ✅ Prevent race conditions
- ⚠️ Can cause deadlocks if misused

---

## Quick Reference

```python
# Threading (same process)
import queue
q = queue.Queue()
q.put(item)
item = q.get()

# Multiprocessing (different processes)
import multiprocessing as mp
q = mp.Queue()
q.put(item)
item = q.get()

# Shared memory
val = mp.Value('i', 0)
arr = mp.Array('i', [1, 2, 3])

# Synchronization
lock = mp.Lock()
event = mp.Event()
barrier = mp.Barrier(3)

# Signal patterns
event.set()    # Signal
event.wait()   # Wait for signal
event.clear()  # Reset
```
