# CREATIONAL PATTERNS

## Singleton
### Problem
Ensures a class has only one instance (e.g., to manage shared resources like a database connection or configuration settings) while providing global access to it
> Without it, multiple instances could lead to inconsistencies or resource waste...\
> Like a government, A country can have only one official government and everybody must talk to the same one

### Solution
The class controls its own instantiation: 
- a private constructor
- a static variable to hold the single instance
- a static method to `get` the instance (creating it if needed)
> Thread-safety is often added for concurrent environments

### Structure
|Singleton|
|:---|
|- instance: Singleton|
|- Singleton()|
|+ getInstance(): Singleton|
```bash
# getInstance()
if (instance == null) {
   instance = new Singleton()
}
return instance
```

### Code Examples
#### Python
```python
class ModbusMaster:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            # init serial port, etc.
        return cls._instance
```
#### Rust
```rust
use once_cell::sync::Lazy;
use std::sync::Mutex;

static CONFIG: Lazy<Mutex<AppConfig>> = Lazy::new(|| {
    Mutex::new(AppConfig { debug: false, name: "gateway".into() })
});

#[derive(Debug)]
struct AppConfig {
    debug: bool,
    name: String,
}

// Usage anywhere
let mut cfg = CONFIG.lock().unwrap();
cfg.debug = true;
```

### Usage
**General**
- Global logger database
- Configuration manager
  
**Technical**
- One Modbus Master per process
- Shared MQTT client or cloud connection pool

---

## Builder

### Problem
Object has many optional parameters - telescoping constructors or long argument lists - hard to read and error-prone

### Solution
Separate construction with a fluent builder - chainable methods - final `build()` - immutable result

### Structure
| Builder |
|:---|
| + with_xxx() |
| + build() |

```bash
obj = Builder()
        .host("192.168.1.10")
        .port(502)
        .timeout(1000)
        .build()
```

### Code Examples

#### Python
```python
class Request:
    def __init__(self, host, port, timeout):
        self.host = host
        self.port = port
        self.timeout = timeout

class RequestBuilder:
    def __init__(self):
        self.host = "localhost"
        self.port = 8080
        self.timeout = 5000

    def host(self, h):    self.host = h;    return self
    def port(self, p):    self.port = p;    return self
    def timeout(self, t): self.timeout = t; return self
    def build(self):      return Request(self.host, self.port, self.timeout)

# Usage
req = RequestBuilder().host("api.com").port(443).timeout(2000).build()
```

#### Rust
```rust
#[derive(Debug)]
struct Request { host: String, port: u16, timeout: u64 }

struct RequestBuilder {
    host: String, port: u16, timeout: u64,
}

impl RequestBuilder {
    fn new() -> Self {
        Self { host: "localhost".into(), port: 8080, timeout: 5000 }
    }
    fn host(mut self, h: &str) -> Self { self.host = h.into(); self }
    fn port(mut self, p: u16) -> Self { self.port = p; self }
    fn timeout(mut self, t: u64) -> Self { self.timeout = t; self }
    fn build(self) -> Request { Request { host: self.host, port: self.port, timeout: self.timeout } }
}

// Usage
let req = RequestBuilder::new()
    .host("192.168.1.10")
    .port(502)
    .timeout(1000)
    .build();
```

### Usage
**General**
- Complex immutable objects
- API clients with many options

**Technical**
- Building Modbus requests
- Configuring serial/TCP connections
- Creating telemetry payloads

---

## Prototype

### Problem
Creating similar objects from scratch is expensive (config parsing, heavy init)

### Solution
Clone an existing fully-configured object (prototype) and modify only what differs

### Structure
| Prototype |
|:---|
| + clone() |

```bash
new_obj = template.clone()
new_obj.id = 42
```

### Code Examples

#### Python
```python
import copy

class Device:
    def __init__(self):
        self.id = 0
        self.name = "sensor"
        self.enabled = True

    def clone(self):
        return copy.deepcopy(self)

# Template
template = Device()

dev1 = template.clone()
dev1.id = 101

dev2 = template.clone()
dev2.id = 102
```

#### Rust
```rust
#[derive(Clone, Debug)]
struct Device {
    id: u32,
    name: String,
    enabled: bool,
}

// Template
let template = Device { id: 0, name: "sensor".into(), enabled: true };

let dev1 = template.clone();
let dev2 = template.clone();
```

### Usage
**General**
- Expensive object creation
- Template-based instances

**Technical**
- Provisioning many identical Modbus devices
- Fast boot from pre-configured template
- Test fixtures

---

## Factory Method

### Problem
You know you need an object, but exact type depends on subclass or config

### Solution
Define creation method in base class - let subclasses decide concrete type

### Structure
| Creator |
|:---|
| + factory_method() |

```bash
product = creator.factory_method()
```

### Code Examples

#### Python
```python
from abc import ABC, abstractmethod

class Logger(ABC):
    @abstractmethod
    def log(self, msg): pass

class ConsoleLogger(Logger):
    def log(self, msg): print(msg)

class FileLogger(Logger):
    def log(self, msg): pass  # write to file

class Application(ABC):
    def create_logger(self): raise NotImplementedError
    def start(self):
        self.create_logger().log("started")

class DevApp(Application):
    def create_logger(self): return ConsoleLogger()

class ProdApp(Application):
    def create_logger(self): return FileLogger()
```

#### Rust
```rust
trait Logger { fn log(&self, msg: &str); }

struct ConsoleLogger;
struct FileLogger;

impl Logger for ConsoleLogger { fn log(&self, msg: &str) { println!("{}", msg); } }
impl Logger for FileLogger    { fn log(&self, _: &str) { /* file */ } }

trait Application {
    fn logger(&self) -> Box<dyn Logger>;
    fn start(&self) { self.logger().log("App started"); }
}

struct DevApp;
struct ProdApp;

impl Application for DevApp  { fn logger(&self) -> Box<dyn Logger> { Box::new(ConsoleLogger) } }
impl Application for ProdApp { fn logger(&self) -> Box<dyn Logger> { Box::new(FileLogger) } }
```

### Usage
**General**
- Framework extension points
- Runtime type selection

**Technical**
- Choose RTU vs TCP transport
- Create correct parser per device type
- Master vs Slave role

---

## Abstract Factory

### Problem
Need families of related objects that must work together (e.g. UI components, device drivers)

### Solution
Factory interface that creates entire product families - concrete factories per family

### Structure
| AbstractFactory |
|:---|
| + create_product_a() |
| + create_product_b() |

```bash
factory = WindowsFactory()
button = factory.create_button()
checkbox = factory.create_checkbox()
```

### Code Examples

#### Python
```python
class Button:  def render(self): pass
class Checkbox: def check(self): pass

class WinButton(Button):  def render(self): print("Win button")
class WinCheckbox(Checkbox): def check(self): print("Win check")
class MacButton(Button):  def render(self): print("Mac button")
class MacCheckbox(Checkbox): def check(self): print("Mac check")

class GUIFactory:
    def create_button(self): raise NotImplementedError
    def create_checkbox(self): raise NotImplementedError

class WindowsFactory(GUIFactory):
    def create_button(self): return WinButton()
    def create_checkbox(self): return WinCheckbox()

class MacFactory(GUIFactory):
    def create_button(self): return MacButton()
    def create_checkbox(self): return MacCheckbox()
```

#### Rust
```rust
trait Button   { fn render(&self); }
trait Checkbox { fn check(&self); }

struct WinButton;   struct WinCheckbox;
struct MacButton;   struct MacCheckbox;

impl Button for WinButton { fn render(&self) { println!("Win button"); } }
impl Button for MacButton { fn render(&self) { println!("Mac button"); } }
impl Checkbox for WinCheckbox { fn check(&self) { println!("Win check"); } }
impl Checkbox for MacCheckbox { fn check(&self) { println!("Mac check"); } }

trait GuiFactory {
    fn button(&self) -> Box<dyn Button>;
    fn checkbox(&self) -> Box<dyn Checkbox>;
}

struct WindowsFactory;
struct MacFactory;

impl GuiFactory for WindowsFactory {
    fn button(&self) -> Box<dyn Button> { Box::new(WinButton) }
    fn checkbox(&self) -> Box<dyn Checkbox> { Box::new(WinCheckbox) }
}

impl GuiFactory for MacFactory {
    fn button(&self) -> Box<dyn Button> { Box::new(MacButton) }
    fn checkbox(&self) -> Box<dyn Checkbox> { Box::new(MacCheckbox) }
}
```

### Usage
**General**
- GUI themes
- Cross-platform widgets

**Technical**
- Vendor-specific device families (Schneider vs ABB)
- Multi-protocol support
- Test vs production drivers (mock vs real)
