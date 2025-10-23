# 🔁 Consistent Hashing — The Backbone of Scalable Load Balancing

## 🔹 Introduction

When designing distributed systems or load balancers, one common challenge is how to distribute incoming requests or data evenly across multiple servers — and how to handle what happens when servers are added or removed.

Traditional load balancing techniques like Round Robin, Least Connections, or IP Hashing work fine — but each comes with a fatal flaw when the system scales up or changes dynamically.

This is where **Consistent Hashing** shines. It provides a stable, efficient, and scalable way to distribute load that minimizes redistribution when servers are added or removed.

## ⚙️ The Problem with Traditional Load Balancing

Let's quickly recall what happens in typical methods:

| Method | Approach | Major Limitation |
|--------|----------|------------------|
| Round Robin | Cyclically assigns requests to servers | Doesn't consider server load or capacity |
| Least Connections | Chooses server with fewest active connections | Ignores differences in server capacity; may overload weaker nodes |
| IP Hashing | Uses client's IP hash to assign a server | If a server is added/removed, all mappings change (major rehashing needed) |

All of these methods depend on the number of servers.
When a new server joins or an existing one fails, the entire mapping of requests-to-servers changes, causing cache invalidation and load redistribution chaos.

## 🔹 Enter Consistent Hashing

Consistent Hashing is a distributed hashing technique that:

- Works independently of the number of servers
- Minimizes data movement when servers are added or removed
- Ensures balanced load distribution using virtual replicas

In short, it's hashing — but made consistent and scalable.

## 🌀 The Hash Ring (Abstract Circle)

Imagine all possible hash values arranged in a circular space — called a hash ring or abstract circle.

The circle represents the range of hash values (e.g., 0 to 2³² - 1, or in our example, 0–99).

Both servers and keys/requests are mapped onto this ring using a hash function.

**Example:**

```
Hash(Server1) = 10
Hash(Server2) = 40
Hash(Server3) = 60
Hash(Server4) = 80

```

**Visually:**

```
0 → 10 → 40 → 60 → 80 → (back to 0)

 ```



## 🧭 Assigning Requests to Servers

When a request or key comes in:

1. Compute its hash using the same function (e.g., hash(request) % 100).
2. Locate that position on the hash ring.
3. Move clockwise until you find the next server on the ring.
4. Assign the request to that server.

**Example:**

```
Request with hash 15 → next server clockwise is Server2 (at 40)
Request with hash 85 → next server clockwise is Server1 (at 10)

```

This approach ensures deterministic and consistent mapping.

## ⚠️ What Happens When a Server Fails?

Suppose Server2 (at 40) goes down.

In traditional hashing (mod n), all keys might be remapped because n changes.

In consistent hashing, only the keys mapped between Server1 (10) and Server2 (40) move — and only to the next clockwise server (Server3).

So, only a small fraction of keys are remapped.
That's the key advantage.

## 🧩 The Skew Problem — and Virtual Nodes (Replicas)

But there's a subtle issue:
If servers' hash positions are unevenly spaced, load can become skewed — some servers handle far more requests than others.

To fix this, consistent hashing introduces **virtual nodes** (or replicas).

Each physical server is assigned multiple positions (replicas) on the ring using slightly varied names:

```
Server1#1, Server1#2, Server1#3
Server2#1, Server2#2, Server2#3
...

```


These replicas:
- Spread each server's presence around the ring.
- Distribute requests more evenly.
- Make the system robust against skew.

## 🆕 Adding or Removing Servers

Let's say a new server S5 joins.

1. Compute its hash → say hash(S5) = 90.
2. Place it on the ring at position 90.
3. Only keys that fall between the previous server (S4 at 80) and S5 (90) move to the new server.
4. Everything else stays put.

Similarly, if a server fails, only keys in its segment move to the next clockwise server.

**Result:** Minimum data reshuffling.

## 🧮 Why It's Called "Consistent"

The term **consistent** comes from the idea that:

The distribution of keys remains mostly consistent (unchanged) even when servers are added or removed.

Unlike simple modulo-based hashing (hash % num_servers), which causes complete remapping, consistent hashing preserves most key-server assignments.

## ✅ Advantages of Consistent Hashing

| Advantage | Explanation |
|-----------|-------------|
| Scalability | Works seamlessly as servers scale up or down |
| Minimal Rehashing | Only a small portion of keys are reassigned on topology changes |
| Even Load Distribution | Virtual nodes prevent hotspots |
| Fault Tolerance | Easy redistribution when nodes fail |
| Cache Efficiency | Reduces cache invalidation and data movement |

## ⚔️ Comparison: Modulo Hashing vs. Consistent Hashing

| Scenario | Modulo Hashing | Consistent Hashing |
|----------|----------------|-------------------|
| Add a new server | Most keys remapped | Only a small fraction remapped |
| Remove a server | Most keys remapped | Only affected segment remapped |
| Load balance | Can be skewed | Even (with virtual nodes) |
| Scalability | Poor | Excellent |

## 💡 Real-World Uses

Consistent hashing is used in:

- Distributed caching (e.g., Memcached, Redis Cluster)
- CDNs (Content Delivery Networks)
- Load balancers
- Distributed databases (like Cassandra, DynamoDB)
- Peer-to-peer systems (like Chord DHT)

## 🧠 Summary — The Core Idea

1. Map servers and keys on a circular hash space (hash ring).
2. Each key is assigned to the next server clockwise.
3. When servers are added/removed, only nearby keys move.
4. Virtual nodes ensure even distribution.

That's it.
That's consistent hashing — simple, elegant, and foundational to scalable distributed systems.

## 🏁 Key Takeaways

- It's a distributed hashing scheme.
- It operates independently of the number of servers.
- It uses an abstract circular hash space (hash ring).
- It uses virtual replicas (nodes) to avoid load imbalance.
- It ensures minimal disruption when servers change.
