# Step-by-Step Guide

1. Install Arduino IDE
2. Set-up Environment
   - Open Arduino IDE
   - File -> Preferences -> Aditional boards manager URLs: `https://dl.espressif.com/dl/package_esp32_index.json`
   - Boards Manager -> Install _esp32 by Espressif Systems_
   - Tools -> Board:... -> esp32 -> WEMOS LOLIN32 -> configure to: `WEMOS LOLIN32 on COM`
   - File -> Examples -> 01.Basics -> Blink
4. Start

# MODBUS Key Concepts

## 1. Master/Slave (Client/Server)
- Master initiates conversation, queries slaves, and receives data
- Slaves answer by transmitting data, with unique IDs (1 to 247)

## 2. RS485
MODBUS: client/server data communication protocol in the application layer
> like the words spoken during a call

RS485: physical wiring standard, using electrical connections: D+/A and D-/B
> like wires on a telephone line

**IMPORTANT**: A connects to A, B connects to B, can be found in the device's RS485 section

## 3. Register Map
**Input Registers**: 16-bit words (unsigned integers) -> Read-only
> usually sensor data, like current temperature value or current humidity value

**Holding Registers**: 16-bit words (unsigned integers) -> Read-write
> like a notebook, maybe to set a target temperature or to write a configuration value for a sensor

**Discrete Inputs**: Single bits (boolean) -> Read-only
> used to indicate the status of physical inputs like a door contact or switch

**Coils**: Single bits (boolean) -> Read-write
> typically used for control signals like turning a pump on or off

## 4. Baud Rate
It's the speed at which two devices talk to each other, both devices **must use the same baud rate**, or the data becomes unreadable
> like talking speed; if you speak too fast, I can't keep up; If I talk too slow while you talk fast, we can't understand each other

## 5. Byte Structure
### 5.1. Data bit
Length of actual information, if data bit is 8, then:
```bash
[Start Bit] [8 Data Bits] [Parity Bit] [Stop Bit]
```

### 5.2. Parity Bit
**Purpose**: error detection<br>
- It's a **single extra bit** added to each byte of data
- It helps the receiver know if the byte got corrupted during transmission
> like a **checksum or “spell check”** in your message — it tells the receiver

**Even Parity (E)**: the total number of `1`s in the binary message + parity bit = even
> Data: `101100` -> Parity Bit: `1` -> `1011001` (Total `1`s = 4, even)

**Odd Parity (O)**: the total number of `1`s in the binary message + parity bit = odd
> Data: `101100` -> Parity Bit: `0` -> `1011000` (Total `1`s = 3, odd)

**No Parity (N)**: no parity, no error check

### 5.3. Stop Bit
**Purpose**: mark the end of a byte<br>
- It’s a bit (or two) sent after the data byte to signal that the byte is finished
- It tells the receiver: “You can start reading the next byte now”
> like a **period at the end of a sentence** — it separates one byte from the next so the receiver doesn’t get confused
