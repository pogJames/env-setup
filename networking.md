## **1. The OSI Model (The 7-Layer Cake)**

The **OSI (Open Systems Interconnection)** model is a conceptual framework used to understand how data moves across a network. While the internet actually runs on a simpler model (TCP/IP), the OSI model is the industry standard for troubleshooting.

* **Layer 1: Physical** – Cables, connectors, and electricity (bits).
* **Layer 2: Data Link** – Moves data between two physically connected nodes. This is where **MAC Addresses** live.
* **Layer 3: Network** – Routes data between different networks. This is where **IP Addresses** live.
* **Layer 4: Transport** – Ensures data gets there reliably. This is where **TCP** and **UDP** live.
* **Layers 5–7: Session, Presentation, Application** – These handle the data logic, encryption, and the final interface (like **HTTP** or **FTP**).


## **2. Router vs. Switch**

In the 80/20 of networking hardware, you just need to know which layer they operate on.

* **Switch (Layer 2):** Connects devices *within* the same local network (LAN). It uses **MAC addresses** to send data to the correct port. Think of it as a power strip for data inside one room.
* **Router (Layer 3):** Connects *different* networks together (e.g., connecting your home LAN to the Internet). It uses **IP addresses** to decide which "neighborhood" the data needs to go to.

> **The Rule of Thumb:** Switches create networks; Routers connect networks.


## **3. Configuring IPv4 vs. IPv6**

Every device needs an address to communicate.

* **IPv4 (The Old Standard):** Uses 32-bit addresses written in decimal (e.g., `192.168.1.1`). Because there are only ~4.3 billion addresses, we ran out years ago.
* **Config:** Requires a **Subnet Mask** (to define network size) and a **Default Gateway** (the router's IP).


* **IPv6 (The New Standard):** Uses 128-bit addresses written in hexadecimal (e.g., `2001:0db8:85a3...`). There are enough addresses for every grain of sand on Earth.
* **Config:** Often uses **SLAAC** (Stateless Address Autoconfiguration), meaning devices can often generate their own IP without a central server.



## **4. Device Discovery**

1. ARP (Address Resolution Protocol)

The Workhorse: This is the most used discovery protocol in existence.

The 80% Use Case: Every single time an IPv4 packet is sent, an ARP check happens first.

Why it’s essential: It’s the bridge between the IP (software) and the MAC (hardware). If ARP fails, nothing else works.

2. mDNS / DNS-SD (Multicast DNS / Bonjour)
   
The "User Friendly" King: This is how your computer finds printer.local, apple-tv.local, or your esp32.local.

The 80% Use Case: Used by almost all consumer IoT, Apple devices, and modern Linux distributions for "ZeroConf" networking.

Why it's essential: It allows discovery without a central server.

3. DHCP (Dynamic Host Configuration Protocol)
   
The Identity Provider: While primarily for giving out IPs, it is a discovery tool because the "Handshake" (DORA) tells the router exactly who just joined the network.

The 80% Use Case: Every device joining a Wi-Fi or Ethernet network.

Why it's essential: It’s the first time a device "announces" its hostname to the network infrastructure.

4. LLDP / CDP (Link Layer Discovery Protocol)
   
The Infrastructure Map: This is what switches and routers use to "talk" to each other.

The 80% Use Case: Professional networking gear (Cisco, Juniper, Aruba) use this to draw those fancy topology maps you see in enterprise software.

Why it's essential: It tells you exactly which physical port a device is plugged into.


## **5. Tools for Topology Discovery**

1. Wireshark (The Gold Standard)
   
Role: Packet Inspection (Layer 2–7).

Why it's in the 20%: It is the ultimate source of truth. When a device says "I sent the data" and the server says "I didn't get it," Wireshark proves who is lying.

Pareto Use Case: 80% of Wireshark use is just looking for TCP Retransmissions, ICMP Unreachable, or mDNS queries.

2. Nmap (The Network Map)
   
Role: Port Scanning & Host Discovery.

Why it's in the 20%: It’s the fastest way to "inventory" a network.

Pareto Use Case: Most engineers just use nmap -sn (to find live IPs) or nmap -sV (to check service versions).

| Command | What it does | Why you use it |
| --- | --- | --- |
| `nmap -sn 192.168.1.0/24` | **Ping Scan** | Finds every live IP without "knocking" on every port (Fast). |
| `nmap -sV 192.168.1.50` | **Version Detection** | Tells you if that port 80 is running Nginx, Apache, or your custom lwIP server. |
| `nmap -O 192.168.1.50` | **OS Fingerprinting** | Guesses the OS (Linux, FreeRTOS, etc.) by looking at how the TCP stack responds. |
| `nmap --script mdns-discovery` | **Scripting Engine** | Uses mDNS to find the "friendly names" (like `sensor.local`) of your IoT devices. |

3. Tcpdump (The CLI Workhorse)
   
Role: Command-line packet capture.

Why it's in the 20%: Since you’re working on an embedded Linux box, you won't have a GUI for Wireshark. tcpdump is the universal tool found on almost every Linux system.

Workflow: You capture a .pcap file on the embedded box using tcpdump, then move it to your laptop to analyze it visually in Wireshark.

4. Iperf / Iperf3 (The Performance Gauge)
   
Role: Bandwidth & Jitter measurement.

Why it's in the 20%: Once you find a device, the next question is always "Why is it slow?" Iperf tells you if the bottleneck is the network throughput or the device's CPU.
