# Creational Design Patterns - Quick Notes

## Singleton

### Problem
You need exactly one instance of a class shared across the entire system.
> Multiple instances cause inconsistencies, resource conflicts, or waste. Like a government—a country needs one official government, not several competing ones.

### Solution
Make the class responsible for creating and managing its single instance.
> Use a private constructor and static method to access the instance. The class controls when and how it's instantiated.

```python
class DatabaseConnection:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.host = "localhost"
            cls._instance.port = 5432
        return cls._instance
    
    def connect(self):
        return f"Connected to {self.host}:{self.port}"

# Usage - always returns same instance
db1 = DatabaseConnection()
db2 = DatabaseConnection()

print(db1 is db2)  # True
db1.host = "192.168.1.100"
print(db2.host)    # "192.168.1.100"
```

### Usage 
**General:**
- Global logger
- Configuration manager

**IoT:**
- One Modbus Master per process
- Shared MQTT client or cloud connection pool

---

## Factory Method

### Problem
You need to create objects but don't know the exact type until runtime.
> Hardcoding object creation makes code rigid—adding new types requires modifying existing code everywhere. Like ordering "a vehicle" without specifying car, truck, or motorcycle until delivery.

### Solution
Define an interface for creating objects, but let subclasses decide which class to instantiate.
> The creator class works with the product interface, while concrete creators return specific implementations. Client code stays the same when new types are added.

```python
from abc import ABC, abstractmethod

class DataSource(ABC):
    @abstractmethod
    def read(self): pass

class FileSource(DataSource):
    def read(self):
        return "Reading from file..."

class DatabaseSource(DataSource):
    def read(self):
        return "Reading from database..."

# Factory
class DataProcessor:
    def create_source(self, source_type: str) -> DataSource:
        if source_type == "file":
            return FileSource()
        elif source_type == "db":
            return DatabaseSource()
        raise ValueError(f"Unknown source: {source_type}")

# Usage
processor = DataProcessor()
source = processor.create_source("file")
data = source.read()
```

### Usage
**General:**
- Payment processing systems
- Document exporters
- Notification delivery

**IoT:**
- Sensor driver selection
- Protocol switching at runtime
- Data formatter selection

---

## Abstract Factory

### Problem
You need to create families of related objects that must work together.
> Creating incompatible combinations leads to errors. Like mixing furniture styles—a Windows button with Mac scrollbar looks wrong and may not work properly.

### Solution
Provide an interface for creating families of related objects.
> Each concrete factory produces a complete set of compatible products. Switch factories to switch entire product families at once.

```python
from abc import ABC, abstractmethod

# Abstract products
class Button(ABC):
    @abstractmethod
    def render(self): pass

class Checkbox(ABC):
    @abstractmethod
    def render(self): pass

# Windows family
class WindowsButton(Button):
    def render(self): return "Windows button"

class WindowsCheckbox(Checkbox):
    def render(self): return "Windows checkbox"

# Mac family
class MacButton(Button):
    def render(self): return "Mac button"

class MacCheckbox(Checkbox):
    def render(self): return "Mac checkbox"

# Abstract factory
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button: pass
    @abstractmethod
    def create_checkbox(self) -> Checkbox: pass

# Concrete factories
class WindowsFactory(GUIFactory):
    def create_button(self): return WindowsButton()
    def create_checkbox(self): return WindowsCheckbox()

class MacFactory(GUIFactory):
    def create_button(self): return MacButton()
    def create_checkbox(self): return MacCheckbox()

# Usage
factory = WindowsFactory()
button = factory.create_button()
checkbox = factory.create_checkbox()
```

### Usage
**General:**
- UI theme components
- Cross-platform widget sets
- Database driver families

**IoT:**
- Modbus RTU vs TCP protocol stacks
- Cloud provider service sets
- Sensor ecosystem components

---

## Builder

### Problem
Creating complex objects with many optional parameters leads to unreadable constructors.
> The telescoping constructor problem makes code hard to read and error-prone. Like ordering a custom sandwich—easier to say "add this, add that" than listing everything upfront.

### Solution
Construct objects step-by-step using a builder interface.
> Build only what you need, in any order, with readable method chaining. Each method returns the builder for fluent interface.

```python
class HttpRequest:
    def __init__(self):
        self.method = "GET"
        self.url = ""
        self.headers = {}
        self.timeout = 30

class HttpRequestBuilder:
    def __init__(self):
        self._request = HttpRequest()
    
    def method(self, method: str):
        self._request.method = method
        return self
    
    def url(self, url: str):
        self._request.url = url
        return self
    
    def header(self, key: str, value: str):
        self._request.headers[key] = value
        return self
    
    def timeout(self, seconds: int):
        self._request.timeout = seconds
        return self
    
    def build(self) -> HttpRequest:
        return self._request

# Usage - fluent interface
request = (HttpRequestBuilder()
           .method("POST")
           .url("https://api.example.com/data")
           .header("Content-Type", "application/json")
           .timeout(10)
           .build())
```

### Usage
**General:**
- Complex configuration objects
- SQL query builders
- Test data creation

**IoT:**
- Modbus connection configuration
- Sensor setup and calibration
- MQTT client initialization

---

## Prototype

### Problem
Creating new objects from scratch is expensive when initialization is complex.
> Recreating objects with database queries, file loading, or heavy computation wastes resources. Like photocopying—faster to copy than retype the entire document.

### Solution
Clone existing objects instead of creating new ones.
> Objects implement their own cloning logic, including deep copies of internal state. Modify the clone without affecting the original.

```python
from copy import deepcopy

class SensorConfig:
    def __init__(self, pin, threshold):
        self.pin = pin
        self.threshold = threshold
        self.calibration = []
    
    def clone(self):
        return deepcopy(self)

# Usage
base = SensorConfig(pin="A0", threshold=30)
base.calibration = [1.0, 1.1, 0.9]

sensor1 = base.clone()
sensor1.pin = "A1"
sensor1.threshold = 25

sensor2 = base.clone()
sensor2.pin = "A2"

# Independent copies
sensor1.calibration.append(1.2)
print(len(base.calibration))     # 3
print(len(sensor1.calibration))  # 4
```

### Usage
**General:**
- Game character templates
- Document templates
- Test fixture cloning

**IoT:**
- Sensor configuration templates
- Modbus register map copying
- Network packet templates

---

## Quick Comparison

| Pattern | Creates | When to Use |
|---------|---------|-------------|
| **Factory Method** | One type at a time | Type varies at runtime |
| **Abstract Factory** | Related families | Need compatible sets |
| **Builder** | Complex objects | Many optional parameters |
| **Prototype** | By cloning | Creation is expensive |
| **Singleton** | Exactly one | Need single shared instance |

## Pattern Selection Guide

**Choose based on your problem:**

- Need one global instance? → **Singleton**
- Creating from scratch is slow? → **Prototype**  
- Many optional parameters? → **Builder**
- Need compatible sets? → **Abstract Factory**
- Type determined at runtime? → **Factory Method**

---

# Structural Design Patterns - Quick Notes

## Adapter

### Problem
You need to make incompatible interfaces work together.
> Like using a US plug in a European socket—you need an adapter to convert one interface to another. Without it, components can't communicate despite having the functionality you need.

### Solution
Create a wrapper that converts one interface into another that clients expect.
> The adapter translates calls from the client's expected interface to the adaptee's actual interface. Client code doesn't know about the adaptation.

```python
# Existing class with incompatible interface
class LegacyTemperatureSensor:
    def get_temp_fahrenheit(self):
        return 77.0

# Target interface we want
class TemperatureSensor:
    def read_celsius(self):
        pass

# Adapter
class TemperatureAdapter(TemperatureSensor):
    def __init__(self, legacy_sensor):
        self.legacy_sensor = legacy_sensor
    
    def read_celsius(self):
        fahrenheit = self.legacy_sensor.get_temp_fahrenheit()
        return (fahrenheit - 32) * 5/9

# Usage
legacy = LegacyTemperatureSensor()
sensor = TemperatureAdapter(legacy)
print(sensor.read_celsius())  # 25.0
```

### Usage
**General:**
- Third-party library integration
- Legacy code modernization
- API version compatibility

**IoT:**
- Sensor protocol conversion
- Modbus to MQTT gateway
- Unit conversion wrappers

---

## Bridge

### Problem
You need to decouple abstraction from implementation so both can vary independently.
> Like a remote control and TV—you want any remote to work with any TV brand. Without Bridge, you'd need RemoteSony, RemoteSamsung, RemoteLG multiplied by BasicRemote, AdvancedRemote.

### Solution
Separate the abstraction hierarchy from the implementation hierarchy.
> Abstraction contains a reference to implementation. Changes to either don't affect the other.

```python
# Implementation interface
class DataTransport:
    def send(self, data): pass

# Concrete implementations
class SerialTransport(DataTransport):
    def send(self, data):
        return f"Sending via Serial: {data}"

class WiFiTransport(DataTransport):
    def send(self, data):
        return f"Sending via WiFi: {data}"

# Abstraction
class DataLogger:
    def __init__(self, transport: DataTransport):
        self.transport = transport
    
    def log(self, message):
        return self.transport.send(message)

# Extended abstraction
class SecureDataLogger(DataLogger):
    def log(self, message):
        encrypted = f"[ENCRYPTED]{message}"
        return self.transport.send(encrypted)

# Usage
logger = DataLogger(SerialTransport())
print(logger.log("Temperature: 25°C"))

secure = SecureDataLogger(WiFiTransport())
print(secure.log("Temperature: 25°C"))
```

### Usage
**General:**
- GUI frameworks with multiple platforms
- Database drivers with multiple backends
- Media players with various codecs

**IoT:**
- Data loggers with multiple transports
- Sensor readers with various protocols
- Display systems with different screens

---

## Composite

### Problem
You need to treat individual objects and compositions uniformly.
> Like a file system—folders contain files AND other folders. You want to perform operations on both without checking types.

### Solution
Create a tree structure where individual objects and compositions share the same interface.
> Clients treat single objects and compositions identically. Operations cascade down the tree.

```python
from abc import ABC, abstractmethod

# Component interface
class SensorComponent(ABC):
    @abstractmethod
    def read(self): pass

# Leaf
class TemperatureSensor(SensorComponent):
    def __init__(self, name):
        self.name = name
    
    def read(self):
        return f"{self.name}: 25°C"

# Composite
class SensorGroup(SensorComponent):
    def __init__(self, name):
        self.name = name
        self.children = []
    
    def add(self, component):
        self.children.append(component)
    
    def read(self):
        results = [f"{self.name}:"]
        for child in self.children:
            results.append(f"  {child.read()}")
        return "\n".join(results)

# Usage
floor1 = SensorGroup("Floor 1")
floor1.add(TemperatureSensor("Room A"))
floor1.add(TemperatureSensor("Room B"))

floor2 = SensorGroup("Floor 2")
floor2.add(TemperatureSensor("Room C"))

building = SensorGroup("Building")
building.add(floor1)
building.add(floor2)

print(building.read())
```

### Usage
**General:**
- File system hierarchies
- UI component trees
- Organization structures

**IoT:**
- Sensor network topologies
- Device group management
- Hierarchical configuration systems

---

## Decorator

### Problem
You need to add responsibilities to objects dynamically without affecting other objects.
> Like adding toppings to pizza—each topping wraps the pizza and adds cost/features. Creating PizzaWithCheese, PizzaWithCheeseAndMushrooms, etc. would explode into too many classes.

### Solution
Wrap objects in decorator objects that add new behavior.
> Decorators have the same interface as wrapped objects. Multiple decorators can be stacked.

```python
# Component interface
class DataStream:
    def write(self, data): pass

# Concrete component
class FileStream(DataStream):
    def write(self, data):
        return f"Writing to file: {data}"

# Decorators
class EncryptedStream(DataStream):
    def __init__(self, stream: DataStream):
        self.stream = stream
    
    def write(self, data):
        encrypted = f"[ENCRYPTED({data})]"
        return self.stream.write(encrypted)

class CompressedStream(DataStream):
    def __init__(self, stream: DataStream):
        self.stream = stream
    
    def write(self, data):
        compressed = f"[COMPRESSED({data})]"
        return self.stream.write(compressed)

# Usage
stream = FileStream()
stream = EncryptedStream(stream)
stream = CompressedStream(stream)
print(stream.write("sensor data"))
# Writing to file: [COMPRESSED([ENCRYPTED(sensor data)])]
```

### Usage
**General:**
- I/O stream processing
- GUI component styling
- Logging with filters

**IoT:**
- Data encryption layers
- Protocol wrapping
- Sensor data filtering

---

## Facade

### Problem
You need a simple interface to a complex subsystem.
> Like a car's steering wheel—it hides the complexity of steering mechanism, power steering, wheel alignment. You just turn the wheel.

### Solution
Provide a unified interface to a set of interfaces in a subsystem.
> Facade defines a higher-level interface that makes the subsystem easier to use. It doesn't hide the subsystem, just simplifies access.

```python
# Complex subsystem
class ModbusConnection:
    def open_serial(self, port, baud): return "Serial opened"
    def set_timeout(self, ms): return "Timeout set"

class ModbusProtocol:
    def set_slave_id(self, id): return "Slave ID set"
    def build_request(self, reg): return f"Request for register {reg}"

class ModbusTransport:
    def send(self, data): return f"Sent: {data}"
    def receive(self): return "Response received"

# Facade
class ModbusClient:
    def __init__(self):
        self.connection = ModbusConnection()
        self.protocol = ModbusProtocol()
        self.transport = ModbusTransport()
    
    def read_register(self, port, slave_id, register):
        self.connection.open_serial(port, 9600)
        self.connection.set_timeout(1000)
        self.protocol.set_slave_id(slave_id)
        request = self.protocol.build_request(register)
        self.transport.send(request)
        return self.transport.receive()

# Usage - simple interface
client = ModbusClient()
result = client.read_register("/dev/ttyUSB0", 1, 0)
```

### Usage
**General:**
- Library wrappers
- API simplification
- Framework initialization

**IoT:**
- Modbus client libraries
- Sensor initialization sequences
- Cloud service SDKs

---

## Flyweight

### Problem
You need to support large numbers of fine-grained objects efficiently.
> Like rendering text—storing font, size, color for each character wastes memory. Better to share common properties (intrinsic state) and store only position (extrinsic state).

### Solution
Share common data between multiple objects instead of storing it in each object.
> Split object state into intrinsic (shared) and extrinsic (unique). Factory ensures flyweights are shared properly.

```python
# Flyweight
class SensorType:
    def __init__(self, model, calibration):
        self.model = model
        self.calibration = calibration
    
    def read(self, pin):
        return f"{self.model} on {pin}: {self.calibration}°C"

# Flyweight factory
class SensorFactory:
    def __init__(self):
        self._types = {}
    
    def get_sensor_type(self, model, calibration):
        key = (model, calibration)
        if key not in self._types:
            self._types[key] = SensorType(model, calibration)
        return self._types[key]

# Context
class Sensor:
    def __init__(self, sensor_type, pin):
        self.sensor_type = sensor_type  # Shared
        self.pin = pin  # Unique

# Usage
factory = SensorFactory()
dht22_type = factory.get_sensor_type("DHT22", 1.0)

sensors = [
    Sensor(dht22_type, "A0"),
    Sensor(dht22_type, "A1"),
    Sensor(dht22_type, "A2"),
]

for sensor in sensors:
    print(sensor.sensor_type.read(sensor.pin))
```

### Usage
**General:**
- Text rendering systems
- Game particle systems
- Cached database rows

**IoT:**
- Large sensor networks with few types
- Network packet pools
- Configuration sharing across devices

---

## Proxy

### Problem
You need to control access to an object or add functionality without changing it.
> Like a credit card—it's a proxy for your bank account. It adds access control, logging, lazy loading without changing the bank account itself.

### Solution
Provide a surrogate that controls access to the real object.
> Proxy has the same interface as the real object. It can add pre/post processing, lazy initialization, access control, or logging.

```python
# Subject interface
class SensorReader:
    def read(self): pass

# Real subject
class RemoteSensor(SensorReader):
    def __init__(self, address):
        self.address = address
        print(f"Connecting to sensor at {address}...")
    
    def read(self):
        return f"Data from {self.address}: 25°C"

# Proxy
class SensorProxy(SensorReader):
    def __init__(self, address):
        self.address = address
        self._sensor = None
    
    def read(self):
        if self._sensor is None:  # Lazy initialization
            self._sensor = RemoteSensor(self.address)
        print("Access logged")
        return self._sensor.read()

# Usage
proxy = SensorProxy("192.168.1.10")  # No connection yet
print(proxy.read())  # Connects now
print(proxy.read())  # Uses existing connection
```

### Usage
**General:**
- Lazy loading
- Access control
- Logging and caching

**IoT:**
- Remote sensor access
- Network connection pooling
- Resource-intensive device initialization

---

# BEHAVIORAL DESIGN PATTERNS – Industry Reality (Pareto 80/20)

## Strategy

### Problem
You need to change an object’s behavior/algorithm at runtime without changing its class or polluting it with conditionals.  
> Adding a new payment method, compression algorithm, or ML inference backend should not require modifying existing classes.  
> Like a GPS app: same route, but you can switch between “fastest”, “shortest”, or “eco” mode anytime.

### Solution
Define a family of algorithms, encapsulate each one in its own class, and make them interchangeable at runtime.  
> Context holds a reference to a strategy object and delegates the varying part to it.  
> New strategies can be added without touching existing code (Open/Closed Principle).

```python
from abc import ABC, abstractmethod

class InferenceStrategy(ABC):
    @abstractmethod
    def run(self, input_data): ...

class CpuInference(InferenceStrategy):
    def run(self, input_data): return "Running on CPU"

class GpuInference(InferenceStrategy):
    def run(self, input_data): return "Running on GPU (TensorRT)"

class EdgeInference(InferenceStrategy):
    def run(self, input_data): return "Running on MCU (TFLite Micro)"

class Model:
    def __init__(self, strategy: InferenceStrategy):
        self.strategy = strategy
    
    def set_strategy(self, strategy: InferenceStrategy):
        self.strategy = strategy
    
    def predict(self, data):
        return self.strategy.run(data)

# Usage – same model, different backends
model = Model(CpuInference())
print(model.predict(None))           # Running on CPU
model.set_strategy(EdgeInference())
print(model.predict(None))           # Running on MCU (TFLite Micro)
```

### Usage (real industry 80/20)
**General:**  
- Payment processors  
- Sorting/compression algorithms  
- Validation rules  
**IoT/Embedded:**  
- Modbus transport (RTU vs TCP vs ASCII)  
- Sensor fusion algorithms  
- Power management policies

---

## Observer (Pub/Sub / Event Bus)

### Problem
An object must notify multiple dependent objects when its state changes, without hard-coding who they are.  
> Tight coupling makes it impossible to add or remove listeners later.  
> Like a stock price: many charts, phones, and trading bots need to react instantly.

### Solution
Define a subscription mechanism: subject maintains a list of observers and notifies them automatically.

```python
class TemperatureSensor:
    def __init__(self):
        self._observers = []
        self._temp = 0.0
    
    def attach(self, observer):
        self._observers.append(observer)
    
    @property
    def temperature(self):
        return self._temp
    @temperature.setter
    def temperature(self, value):
        self._temp = value
        for observer in self._observers:
            observer.update(value)

class CloudUploader:
    def update(self, temp): print(f"Uploaded to cloud: {temp}°C")

class LocalDisplay:
    def update(self, temp): print(f"Screen: {temp:.1f}°C")

# Usage
sensor = TemperatureSensor()
sensor.attach(CloudUploader())
sensor.attach(LocalDisplay())
sensor.temperature = 23.7
# → both observer methods called automatically
```

### Usage (real industry 80/20)
**General:**  
- GUI frameworks  
- Event-driven systems  
**IoT/Embedded:**  
- MQTT on_message callbacks  
- Sensor → dashboard → logger pipelines  
- BLE characteristic notifications

---

## Decorator (Wrapper)

### Problem
You want to add behavior (logging, retry, caching, metrics) to existing objects dynamically and in any combination.  
> Subclassing explodes (Logged + Cached + Retried = 8 classes).  
> Like coffee: you start with espresso, then add milk, sugar, whip — same base, layered features.

### Solution
Create decorator classes that wrap the original component and add behavior before/after delegating.

```python
class ModbusClient:
    def read_holding(self, addr, count=1):
        print(f"Raw read {addr}")
        return [42] * count

class RetryDecorator:
    def __init__(self, client): self.client = client
    def read_holding(self, addr, count=1):
        for i in range(3):
            try:
                return self.client.read_holding(addr, count)
            except Exception:
                if i == 2: raise
                print(f"Retry {i+1}/3")

class LoggingDecorator:
    def __init__(self, client): self.client = client
    def read_holding(self, addr, count=1):
        print(f"→ read_holding({addr}, {count})")
        result = self.client.read_holding(addr, count)
        print(f"← {result}")
        return result

# Stack them freely
client = LoggingDecorator(RetryDecorator(ModbusClient()))
client.read_holding(40001, 5)
```

### Usage (real industry 80/20)
**General:**  
- Middleware stacks (Flask, Django)  
- Logging, metrics, caching  
**IoT/Embedded:**  
- Modbus/TCP retry + timeout + logging  
- BLE connection with reconnection wrapper

---

## Command

### Problem
You need to parameterize objects with actions, queue them, support undo, or transmit them over network.  
> GUI buttons, remote controls, and job queues all represent “do something later”.

### Solution
Turn a request into a standalone object containing everything needed to execute it.

```python
class Switch:
    def on(self):  print("Device ON")
    def off(self): print("Device OFF")

class Command:
    def execute(self): ...

class TurnOn(Command):
    def __init__(self, switch, state):
        self.switch = switch
        self.state = state
    def execute(self):
        if self.state: self.switch.on()
        else: self.switch.off()

class RemoteControl:
    def __init__(self):
        self.commands = [None] * 7
    def set_command(self, slot, cmd):
        self.commands[slot] = cmd
    def press(self, slot):
        if self.commands[slot]:
            self.commands[slot].execute()

# Usage
device = Switch()
remote = RemoteControl()
remote.set_command(0, TurnOn(device, True))
remote.set_command(1, TurnOn(device, False))
remote.press(0)   # Device ON
remote.press(1)   # Device OFF
```

### Usage (real industry 80/20)
**General:**  
- Undo/redo in editors  
- Macro recording  
**IoT/Embedded:**  
- Remote control panels  
- Queued device commands (safe shutdown sequences)

---

## Template Method

### Problem
You have a multi-step algorithm where most steps are identical, but a few vary between implementations.  
> Copy-pasting the whole algorithm for tiny differences is error-prone and hard to maintain.

### Solution
Define the algorithm skeleton in a base class; make varying steps abstract or hooks that subclasses override.

```python
from abc import ABC, abstractmethod

class FirmwareUpdate(ABC):
    def perform_update(self):
        self.connect()
        self.erase_flash()
        self.write_firmware()
        self.verify()
        self.reboot()
        print("Update completed")
    
    def connect(self): print("Connected via bootloader")
    def reboot(self):  print("Device rebooted")
    
    @abstractmethod
    def erase_flash(self): ...
    @abstractmethod
    def write_firmware(self): ...
    @abstractmethod
    def verify(self): ...

class Stm32Update(FirmwareUpdate):
    def erase_flash(self): print("STM32: mass erase")
    def write_firmware(self): print("STM32: page write")
    def verify(self): print("STM32: CRC check")
```

### Usage (real industry 80/20)
**General:**  
- Test frameworks (setup → test → teardown)  
**IoT/Embedded:**  
- Device provisioning flows  
- Communication protocol handlers (common framing, different payloads)
---

