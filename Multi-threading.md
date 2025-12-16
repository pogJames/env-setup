# Multi-threading and CPU Affinity Guide

## Python Threading Fundamentals

### What is Threading?

**Thread**: Lightweight execution unit within a process that shares memory space
- Multiple threads = concurrent execution paths in same program
- Share process resources (memory, file handles)
- Each has own stack and program counter

### Basic Threading in Python

```python
import threading
import time

def worker(name, delay):
    """Simple worker function"""
    print(f"Thread {name} starting")
    time.sleep(delay)
    print(f"Thread {name} finishing")

# Create and start threads
t1 = threading.Thread(target=worker, args=("A", 2))
t2 = threading.Thread(target=worker, args=("B", 1))

t1.start()
t2.start()

# Wait for completion
t1.join()
t2.join()
print("All threads complete")
```

### Thread with Class

```python
import threading

class WorkerThread(threading.Thread):
    def __init__(self, name, count):
        super().__init__()
        self.name = name
        self.count = count
    
    def run(self):
        for i in range(self.count):
            print(f"{self.name}: {i}")
            time.sleep(0.5)

# Usage
t1 = WorkerThread("Thread-1", 3)
t2 = WorkerThread("Thread-2", 3)
t1.start()
t2.start()
t1.join()
t2.join()
```

### Thread Synchronization

```python
import threading

# Shared resource
counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:  # Acquire lock automatically
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Final counter: {counter}")  # Should be 1,000,000
```

### Common Synchronization Primitives

```python
# Lock - mutual exclusion
lock = threading.Lock()
lock.acquire()
# critical section
lock.release()

# RLock - reentrant lock (same thread can acquire multiple times)
rlock = threading.RLock()

# Semaphore - limit concurrent access
semaphore = threading.Semaphore(3)  # Max 3 threads

# Event - signal between threads
event = threading.Event()
event.set()    # Signal
event.wait()   # Wait for signal
event.clear()  # Reset

# Condition - complex coordination
condition = threading.Condition()
```

---

## Linux Process Types and Threads

### Process Hierarchy in Linux

**Process**: Independent program execution with own memory space
- Each process has unique PID (Process ID)
- Parent-child relationships form process tree
- Init/systemd (PID 1) is ancestor of all processes

### Process Types

**1. Regular Processes**
- User-initiated programs
- Run in user space
- Examples: web browsers, editors

**2. Kernel Threads**
- Run in kernel space
- Manage system resources
- Examples: `kworker`, `ksoftirqd`
- Shown in brackets: `[kthreadd]`

**3. Daemon Processes**
- Background services
- No controlling terminal
- Examples: `sshd`, `cron`, `systemd`

**4. Zombie Processes**
- Terminated but not reaped by parent
- Shows as `<defunct>` in ps output
- Holds minimal resources (just PID)

**5. Orphan Processes**
- Parent terminated before child
- Adopted by init/systemd

### Process vs Thread Relationship

```
Process (PID 1234)
├── Main Thread (LWP 1234)
├── Worker Thread (LWP 1235)
├── I/O Thread (LWP 1236)
└── Network Thread (LWP 1237)
```

**Key Differences:**

| Aspect | Process | Thread |
|--------|---------|--------|
| Memory | Separate address space | Shared address space |
| Creation | fork() - expensive | pthread_create() - cheap |
| Communication | IPC required | Direct memory access |
| Overhead | High | Low |
| Isolation | Strong | Weak |
| Linux implementation | Task with separate memory | Task sharing memory |

**In Linux:**
- Both processes and threads are "tasks"
- LWP (Light Weight Process) = thread
- `ps -eLf` shows all threads
- Threads share: memory, file descriptors, signal handlers
- Threads have separate: stack, registers, thread-local storage

### Viewing Processes and Threads

```bash
# View all processes
ps aux

# View threads for specific process
ps -T -p <PID>

# View all threads system-wide
ps -eLf

# Thread count for process
ps -o nlwp <PID>

# Detailed thread view with top
top -H -p <PID>

# Using htop (press H to toggle threads)
htop
```

---

## CPU Affinity

### What is CPU Affinity?

**CPU Affinity**: Binding process/thread to specific CPU core(s)
- Improves cache locality
- Reduces context switching overhead
- Enables deterministic performance

### Why Use CPU Affinity?

**Benefits:**
- **Cache efficiency**: Data stays in core's cache
- **Predictable latency**: Real-time applications
- **Workload isolation**: Prevent interference
- **NUMA optimization**: Keep threads near memory

**Use cases:**
- Real-time systems
- High-frequency trading
- Audio/video processing
- Embedded systems

### Check Available CPUs

```python
import os
import multiprocessing

# Number of logical CPUs
cpu_count = os.cpu_count()
print(f"Available CPUs: {cpu_count}")

# Or using multiprocessing
cpu_count = multiprocessing.cpu_count()
print(f"CPU count: {cpu_count}")
```

### Setting CPU Affinity in Python

```python
import os
import threading

def worker_on_core(core_id):
    """Run this function on specific core"""
    # Set affinity to single core
    os.sched_setaffinity(0, {core_id})
    
    # Verify affinity
    affinity = os.sched_getaffinity(0)
    print(f"Thread {threading.current_thread().name} running on cores: {affinity}")
    
    # Do work
    result = sum(i*i for i in range(1000000))
    return result

# Create thread for core 2
t = threading.Thread(target=worker_on_core, args=(2,))
t.start()
t.join()
```

### Pin Thread to Multiple Cores

```python
import os
import threading

def worker_on_cores(core_set):
    """Run on subset of cores"""
    os.sched_setaffinity(0, core_set)
    print(f"Running on cores: {os.sched_getaffinity(0)}")
    
    # Intensive computation
    for _ in range(5):
        sum(i*i for i in range(10000000))

# Pin to cores 0, 1, 2
t = threading.Thread(target=worker_on_cores, args=({0, 1, 2},))
t.start()
t.join()
```

### Process-Level Affinity

```python
import os

# Set affinity for current process
os.sched_setaffinity(0, {0, 1})  # Cores 0 and 1

# All threads inherit this affinity
# Child processes also inherit

# Get current affinity
current = os.sched_getaffinity(0)
print(f"Process bound to cores: {current}")
```

---

## Distributing Workload Across CPU Pool

### Using ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor
import os

def process_chunk(chunk_id, data):
    """Process data chunk on available core"""
    affinity = os.sched_getaffinity(0)
    print(f"Chunk {chunk_id} on cores {affinity}")
    
    # Processing logic
    return sum(data)

# Create data chunks
data_chunks = [[i for i in range(1000000)] for _ in range(8)]

# Use pool of 4 worker threads
with ThreadPoolExecutor(max_workers=4) as executor:
    results = executor.map(
        lambda args: process_chunk(*args),
        enumerate(data_chunks)
    )
    
    total = sum(results)
    print(f"Total: {total}")
```

### Manual Core Distribution

```python
import os
import threading

def worker_with_affinity(core_id, task_id, data):
    """Worker pinned to specific core"""
    # Pin to single core
    os.sched_setaffinity(0, {core_id})
    
    print(f"Task {task_id} on core {core_id}")
    result = sum(x * x for x in data)
    return result

# Distribute tasks across 4 cores
num_cores = 4
tasks = [list(range(i*1000000, (i+1)*1000000)) for i in range(8)]

threads = []
for i, task_data in enumerate(tasks):
    core_id = i % num_cores  # Round-robin distribution
    t = threading.Thread(
        target=worker_with_affinity,
        args=(core_id, i, task_data)
    )
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

### Advanced: NUMA-Aware Distribution

```python
import os
import threading

def get_numa_info():
    """Get NUMA node information (Linux-specific)"""
    # This is simplified - real implementation needs libnuma
    # Just demonstrating concept
    return {
        0: [0, 1, 2, 3],    # NUMA node 0: cores 0-3
        1: [4, 5, 6, 7]     # NUMA node 1: cores 4-7
    }

def worker_numa_aware(numa_node, core_id, data):
    """Worker respecting NUMA topology"""
    os.sched_setaffinity(0, {core_id})
    print(f"NUMA node {numa_node}, core {core_id}")
    
    # Process data
    return sum(data)

# Distribute work by NUMA node
numa_topology = get_numa_info()
threads = []

for numa_node, cores in numa_topology.items():
    for core in cores:
        t = threading.Thread(
            target=worker_numa_aware,
            args=(numa_node, core, range(1000000))
        )
        threads.append(t)
        t.start()

for t in threads:
    t.join()
```

### Using multiprocessing for True Parallelism

**Note**: Python's GIL limits threading for CPU-bound tasks

```python
import multiprocessing as mp
import os

def cpu_bound_worker(core_id, data):
    """CPU-intensive work on specific core"""
    os.sched_setaffinity(0, {core_id})
    
    # Heavy computation (not blocked by GIL in separate process)
    result = 0
    for x in data:
        result += x * x
    return result

if __name__ == '__main__':
    num_cores = mp.cpu_count()
    data_chunks = [range(i*1000000, (i+1)*1000000) for i in range(num_cores)]
    
    # Create process pool
    with mp.Pool(processes=num_cores) as pool:
        results = pool.starmap(
            cpu_bound_worker,
            [(i, chunk) for i, chunk in enumerate(data_chunks)]
        )
    
    print(f"Results: {sum(results)}")
```

---

## Practical Example: Temperature Monitoring with Affinity

```python
import threading
import time
import os
import random

class SensorReader(threading.Thread):
    def __init__(self, sensor_id, core_id):
        super().__init__()
        self.sensor_id = sensor_id
        self.core_id = core_id
        self.running = True
        self.data = []
        
    def run(self):
        # Pin to specific core
        os.sched_setaffinity(0, {self.core_id})
        print(f"Sensor {self.sensor_id} pinned to core {self.core_id}")
        
        while self.running:
            # Simulate sensor reading
            temp = 20 + random.uniform(-5, 5)
            self.data.append(temp)
            time.sleep(0.1)
    
    def stop(self):
        self.running = False

# Create sensor readers on different cores
sensors = [
    SensorReader(sensor_id=0, core_id=0),
    SensorReader(sensor_id=1, core_id=1),
    SensorReader(sensor_id=2, core_id=2),
]

# Start all sensors
for sensor in sensors:
    sensor.start()

# Run for 5 seconds
time.sleep(5)

# Stop all sensors
for sensor in sensors:
    sensor.stop()
    sensor.join()

# Analyze data
for sensor in sensors:
    avg_temp = sum(sensor.data) / len(sensor.data)
    print(f"Sensor {sensor.sensor_id}: {len(sensor.data)} readings, avg {avg_temp:.2f}°C")
```

---

## Quick Reference

### Threading Commands

```python
# Create thread
t = threading.Thread(target=func, args=(arg1, arg2))
t.start()
t.join()

# Current thread
threading.current_thread()
threading.get_ident()  # Thread ID

# Lock
with lock:
    # critical section
    pass
```

### CPU Affinity Commands

```python
# Set affinity
os.sched_setaffinity(0, {0, 1, 2})  # Cores 0,1,2

# Get affinity
cores = os.sched_getaffinity(0)

# CPU count
os.cpu_count()
```

### Linux Commands

```bash
# View threads
ps -eLf
ps -T -p <PID>
top -H

# Set affinity from command line
taskset -c 0,1 python script.py

# View current affinity
taskset -p <PID>
```

---

## Important Considerations

**Global Interpreter Lock (GIL)**
- Python threads can't execute Python bytecode simultaneously
- One thread executes Python code at a time
- Use threads for I/O-bound tasks
- Use multiprocessing for CPU-bound tasks

**When to Use What:**
- **Threading**: I/O operations, waiting for events, network requests
- **Multiprocessing**: CPU-intensive calculations, parallel computation
- **CPU Affinity**: Real-time requirements, cache optimization, workload isolation

**Best Practices:**
- Don't over-subscribe cores (more threads than cores)
- Consider NUMA topology on multi-socket systems
- Monitor cache misses and context switches
- Test performance before and after pinning
- Leave cores for system processes (don't pin everything)
