# Load Balancing

## What is Load Balancing?

**Definition:**
Load balancing is a method of distributing workloads or network traffic evenly across a group (pool) of resources, such as servers, to ensure optimal resource utilization, high availability, and reliability.

In simple terms, it means spreading the work evenly so that no single resource is overloaded while others sit idle.

## Real-Life Example: Restaurant Analogy

Imagine you walk into a new restaurant where there's only one waiter.

- When the first customer (you) arrives, that waiter takes your order and serves you.
- Later, another customer arrives. Again, the same waiter handles them.
- This works fine when there are only one or two customers.

But as the restaurant becomes popular, more people arrive simultaneously — say, four groups (P1, P2, P3, P4).
Now one waiter has to serve all four tables, which causes delays because:

- While one table is being served, others have to wait.
- The single waiter becomes a bottleneck.

**Solution:**
Hire more waiters.

- Each waiter now serves fewer tables.
- Customers get faster service.
- Everyone's happier.

That's load balancing in action — distributing the "load" (customers) evenly among multiple "resources" (waiters).

## Technical Analogy

Now translate this to computing.

When you use websites like Facebook, Amazon, or WhatsApp, millions of users are sending requests to the servers at the same time.
A single server cannot handle all those requests — it would crash or slow down.

So, companies use multiple servers.
A load balancer sits in front of them and distributes incoming traffic evenly among these servers.

## Why Do We Need Load Balancing?
**Key Objectives**

- **Even Distribution of Load**
  - Prevents some servers from being overloaded while others are idle.
  - Ensures that every server shares the work fairly.

- **Optimal Resource Utilization**
  - All servers are used efficiently.
  - No resource (server) remains underused — this saves cost and maximizes performance.

- **High Availability and Reliability**
  - If one server goes down, others can still handle requests.
  - The system stays up and responsive.

## How Does Load Balancing Work?

The load balancer is a component (hardware or software) that receives incoming traffic and decides which server should handle each request.

But the key question is:
👉 **How does it decide which server gets which request?**

That's where load balancing strategies come in.

## Load Balancing Strategies

Let's assume:

- There are 5 servers (S1, S2, S3, S4, S5).
- There are 100 incoming requests labeled from 1 to 100.

We'll explore different ways to distribute these requests.

### A. Round Robin

**Idea:**
Assign requests to servers in order — one by one — and then loop back.

**Example:**
```

Request 1 → S1
Request 2 → S2
Request 3 → S3
Request 4 → S4
Request 5 → S5
Request 6 → S1 (start again)

```

**Problem:**
Round Robin doesn't consider the load or request size.

Suppose Request 1 is very large (takes long to process),
while Request 2 is small (finishes quickly).

Even if S2 is free, the next incoming request (say, Request 6) will still go to S1 — which is still busy.

So, some servers stay idle, while others are overloaded.
This leads to inefficiency and delays.

### B. Least Connections

**Idea:**
Assign the new request to the server with the fewest active connections (least load).

**Example:**

```
Server 1 currently has 20 active connections.
Server 2 has 80 active connections.
→ The next request will go to Server 1 (because it's less busy).

```


**Advantage:**
More intelligent than Round Robin — it reacts to current server load.

**Problem:**
It doesn't consider the processing power of servers.

What if Server 1 is weak (1 Gbps) and Server 2 is powerful (100 Gbps)?
Then even with 80 connections, Server 2 can still handle more load.

So, assigning requests purely based on the number of connections can still cause imbalance.

### C. IP Hashing

**Idea:**
Use the client's IP address to determine which server handles its request.
A hash function converts the IP address into a number, and that number maps to a specific server.

**Example:**

```
Hash(IP1) → S1
Hash(IP2) → S3
Hash(IP3) → S1
```


**Advantages:**
- The same user (same IP) always goes to the same server.
- → Useful for maintaining sessions (sticky sessions).

**Problems:**
- Uneven distribution: Some IPs might hash to the same server, causing overload.
- If the number of servers changes (say, one is removed or added), the mapping changes, and many requests need to be reassigned, leading to disruption.

### D. Consistent Hashing (Modern Approach)

**Definition:**
A distributed hashing scheme that operates independently of the number of servers by mapping both servers and requests onto a virtual ring (hash ring).

**How It Works**

1. Imagine a circular ring (0°–360°).
2. Each server is assigned a random position on this ring using a hash function.
3. Each request is also hashed to a position on the ring.
4. A request is handled by the first server encountered while moving clockwise from its position.

**When servers are added or removed:**
- Only a small portion of requests need to be remapped.
- This avoids massive redistributions (unlike IP hashing).

**Benefits of Consistent Hashing**

- **Minimal Redistribution on Server Changes**
  - When you add or remove a server, only a few keys/requests move.
  - No need to reassign everything.

- **Balanced Load Distribution**
  - Requests are spread fairly across all servers.

- **Flexibility & Scalability**
  - Servers can be added or removed dynamically.
  - System adapts automatically without breaking existing mappings.

- **High Availability**
  - If one server goes down, its requests can quickly reroute to the next available one.

## Summary

| Strategy | Logic | Pros | Cons |
|----------|-------|------|------|
| Round Robin | Sequential rotation | Simple, easy to implement | Ignores request size/load |
| Least Connections | Chooses least busy server | Dynamic | Ignores server power |
| IP Hashing | Based on client IP hash | Good for session persistence | Poor adaptability |
| Consistent Hashing | Hashes requests and servers onto a ring | Scalable, minimal remapping, robust | Slightly complex to implement |

## Conclusion

Load balancing ensures that multiple servers handle traffic efficiently, preventing any single point of failure.

Different algorithms (Round Robin, Least Connections, IP Hashing) have trade-offs.

Consistent Hashing is the modern, scalable, and fault-tolerant solution that most large systems (like AWS, Netflix, and Google) use today.
