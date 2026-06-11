# 🌐 Distributed Key-Value Store using Consistent Hashing

<p align="center">
  <img src="https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" />
  <img src="https://img.shields.io/badge/Distributed%20Systems-0052CC?style=for-the-badge&logo=apachekafka&logoColor=white" />
  <img src="https://img.shields.io/badge/TCP%20Sockets-239120?style=for-the-badge&logo=socket.io&logoColor=white" />
  <img src="https://img.shields.io/badge/Multithreading-FF6F00?style=for-the-badge&logo=java&logoColor=white" />
  <img src="https://img.shields.io/badge/Course-CSCI%204780%2F6780-6A0DAD?style=for-the-badge" />
</p>

<p align="center">
  <b>A fully working distributed naming service</b> — multiple servers talk to each other over TCP sockets,
  automatically share data when nodes join or leave, and route queries across the network using the
  Consistent Hashing algorithm (the same technique used by Amazon DynamoDB and Apache Cassandra).
</p>

---

## 🧠 What Did I Build?

Think of this like a **distributed dictionary** — you can store, find, and delete words (key-value pairs) — but instead of one dictionary, the data is spread across multiple servers running on a network.

The servers form a **virtual ring**. Each server is responsible for a slice of the data. When you look up a key, the system automatically routes the request across the ring until it finds the right server.

```
                        [ Bootstrap Server : 0 ]
                       /   owns keys 701 – 1023   \
                      ↑                             ↓
        [ NServer : 700 ]               [ NServer : 468 ]
         owns keys 469–700               owns keys 1–468
```

> 🔁 Requests always travel **clockwise** around the ring until they reach the responsible server.

---

## ✅ Key Highlights

| Concept | What's Implemented |
|---|---|
| 📡 **Distributed Systems** | Multiple independent servers communicate over a real TCP network |
| 🔁 **Consistent Hashing** | Keys are distributed across servers using a ring-based algorithm |
| 🔌 **Java Socket Programming** | All inter-server communication is built from scratch using Java Sockets |
| 🧵 **Multithreading** | Each server runs a Listener Thread + a User Interaction Thread simultaneously |
| 🔄 **Dynamic Node Join** | A new server can enter the ring, claim its key range, and receive data automatically |
| 🚪 **Graceful Node Exit** | A server can leave the ring and hand off all its data to its successor — zero data loss |
| 🗂️ **Data Redistribution** | When topology changes, only the affected slice of data moves — not everything |
| 🗺️ **Request Routing** | Every lookup/insert/delete prints the exact path it took across the ring |

---

## 🛠️ Tech Stack

```
Language      →  Java (JDK 8+)
Networking    →  Java TCP Sockets (java.net.Socket / ServerSocket)
Concurrency   →  Java Threads (java.lang.Thread)
Data Storage  →  HashMap<Integer, String> per server
Algorithm     →  Consistent Hashing on a [0, 1023] key ring
Communication →  Plain-text message protocol over TCP
```

---

## 📁 Project Structure

```
Project-4/
│
├── BootstrapServer.java    ← The anchor server (always running, ID = 0)
├── NServer.java            ← Any additional name server (dynamic, joinable/leavable)
│
├── bnConfigFile.txt        ← Config for Bootstrap Server
├── nsConfigFile.txt        ← Config for Name Server 1
├── nsConfigFile2.txt       ← Config for Name Server 2
│
└── README.md
```

---

## ▶️ How to Run It

### Step 1 — Compile
```bash
javac BootstrapServer.java
javac NServer.java
```

### Step 2 — Start Bootstrap Server *(Terminal 1 — always first)*
```bash
java BootstrapServer bnConfigFile.txt
```

### Step 3 — Start Name Servers *(one per terminal)*
```bash
java NServer nsConfigFile.txt     # Terminal 2
java NServer nsConfigFile2.txt    # Terminal 3
```

### Step 4 — Join the ring *(in each Name Server terminal)*
```
NameServer> enter
```

---

## 🖥️ Real Output — What It Actually Looks Like

### Name Server Joining the Ring
```
NameServer> enter
Successful Entry!!
Predecessor ID: 0
Successor ID: 700
Keys that will be managed by this server:
  12  : Cabbage       26  : Okra         47  : Cucumber
  62  : Potato        89  : Apricot      109 : Carrot
  143 : Broccoli      152 : Peas         187 : Fig
  264 : Grape         288 : Cherry       325 : Beetroot
  410 : Fennel        434 : Lemon        502 : Apple
  572 : Cantaloupe    616 : Guava        618 : Lettuce
```
> ✅ Server 468 joined the ring, automatically claimed keys 1–468, and received 18 key-value pairs from its successor.

---

### Looking Up Keys (from Bootstrap Server)
```bash
# 1-hop lookup — key 12 lives at Server 468
BootstrapServer> lookup 12
Key Found !!
Servers Traversed: bootstrap: 0 > server: 468 > Cabbage

# 2-hop lookup — key 616 passes through 468, lands at Server 700
BootstrapServer> lookup 616
Key Found !!
Servers Traversed: bootstrap: 0 > server: 468 > server: 700 > Guava

# Bootstrap-local lookup — key 1016 lives at Bootstrap itself
BootstrapServer> lookup 1016
Key found at Bootstrap server: Zucchini
```
> ✅ The system correctly routes queries across the ring and shows the exact traversal path.

---

## 💬 All Supported Commands

### On the Bootstrap Server
| Command | What It Does |
|---|---|
| `lookup <key>` | Finds and returns the value — shows which servers were visited |
| `Insert <key> <value>` | Stores a new key-value pair in the correct server |
| `delete <key>` | Removes a key-value pair from the correct server |

### On Each Name Server
| Command | What It Does |
|---|---|
| `enter` | Joins the ring, claims its key range, receives data from successor |
| `exit` | Leaves the ring gracefully, hands all data to successor |

---

## ⚙️ Config File Format

**Bootstrap** (`bnConfigFile.txt`)
```
0               ← Server ID (always 0 for Bootstrap)
3768            ← Port number
12 Cabbage      ← Pre-loaded key-value pairs
26 Okra
...
```

**Name Server** (`nsConfigFile.txt`)
```
468             ← This server's unique ID
4768            ← Port number
localhost 3768  ← Bootstrap server address
```

---

## 🔍 How Consistent Hashing Works (Simple Version)

1. All servers are placed on a ring numbered 0 to 1023
2. Each server owns all keys **between itself and its predecessor**
3. When a key needs to be stored or found, the request travels **clockwise** until it reaches the right server
4. When a new server joins, it only takes keys from its immediate neighbor — not the whole ring
5. When a server leaves, its keys go to the next server — no data is lost

> This is exactly how **Amazon DynamoDB**, **Apache Cassandra**, and **Discord's message routing** work at scale.

---

## 👨‍💻 Authors

| Name | University |
|---|---|
| Abhishek Patwardhan | University of Georgia |
| \<Partner Name\> | University of Georgia |

---

## ✅ Academic Integrity

> *This project was done in its entirety by **Abhishek Patwardhan** and **\<Partner Name\>**.
> We hereby state that we have not received unauthorized help of any form.*

---

<p align="center">
  Built with ☕ Java &nbsp;•&nbsp; CSCI 4780/6780 Distributed Computing Systems &nbsp;•&nbsp; University of Georgia
</p>