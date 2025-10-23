# 🧠 Quorum

## 🔹 1. What is a Quorum?

**Literal meaning:**
"The minimum number of people required to start a meeting."
— If not enough people show up, the meeting cannot begin.

**In distributed systems:**
Replace people with nodes and meeting with decision-making.
So, a quorum is the minimum number of nodes that must agree for an operation (read/write) to be valid.

## 🔹 2. Why Quorum?

Distributed systems replicate data across multiple nodes to ensure:

- **Consistency:** All nodes agree on the same data value.
- **Fault tolerance:** The system keeps working even if some nodes fail.

**Without quorum:**
- Different nodes might hold different data versions.
- You might read stale or conflicting data.
- System reliability collapses under node or network failures.

Hence, **Quorum = consistency + fault tolerance** (not necessarily availability).

## 🔹 3. The Concept of Quorum in Distributed Systems

A Quorum-based system uses subsets of nodes (quorums) to:
- Approve operations (read/write).
- Maintain consistency and reliability even when some nodes fail.

## 🔹 4. Quorum Size

**Definition:**
Minimum number of nodes required for a decision or operation to be valid.

**Let:**
- N = total number of nodes
- R = read quorum size
- W = write quorum size

**To ensure consistency:**

```
R = (N + 1) / 2

```

**Example:**
If N = 5,
then R = (5 + 1) / 2 = 3.
→ At least 3 nodes must agree on a value for the read to be accepted.

### 🔵 Write Quorum (W):

Minimum number of nodes that must acknowledge a write operation for it to be valid.

Ensures that a majority of nodes store the updated value.

**Formula:**

```
W = ceil(N / 2) + 1
```


**Example:**
If N = 5,
then W = ceil(5 / 2) + 1 = 4.
→ At least 4 nodes must agree before the write is accepted.

## 🔹 6. Quorum Consistency

**Goal:** Maintain a single consistent state across distributed nodes.

**How it works:**
- Every read/write operation must be approved by a quorum of nodes.
- The overlapping quorum ensures that any read operation always sees the latest successful write.

### 🔸 Importance

**Conflict Resolution:**
When multiple nodes attempt to write conflicting updates (say A→B and A→C), quorum ensures that only one update with majority votes is accepted.

**Data Replication:**
Ensures that replicas stay in sync even during concurrent updates. Quorum guarantees that updates propagate correctly to a majority before they're accepted.

## 🔹 7. Quorum and Fault Tolerance

Quorum mechanisms improve fault tolerance because they don't require all nodes to be available.

Even if some nodes fail, as long as enough nodes respond to form a quorum, the system continues working.

**Examples:**

### 🧩 Node Failure:
If some nodes crash or become unresponsive, quorum can still be achieved using the remaining active nodes.

### 🌐 Network Partitioning:
Even when network issues prevent some nodes from communicating, quorum ensures that the system can make progress using reachable nodes only.

## 🔹 8. Challenges / Issues with Quorum Systems

### Reduced Data Availability
If the quorum size isn't met (e.g., not enough nodes agree), operations fail.

**Example:** W = 4 but only 3 nodes respond → write fails → lower availability.

### Latency
Reaching quorum requires communication between multiple nodes.

If nodes are globally distributed (e.g., India ↔ USA), latency increases significantly.

### Complexity
Managing quorum-based systems involves handling node failures, network partitions, and consistency protocols—non-trivial at scale.

## 🔹 9. Example Scenario

**Let's say:**
- N = 5 nodes → {A, B, C, D, E}
- R = 3, W = 4

**Write Operation:**
Needs 4 nodes to acknowledge before success.
If fewer than 4 respond, the write fails.

**Read Operation:**
Needs 3 nodes to return the same value before success.
Ensures that at least one node has the latest written value (since 3 + 4 > 5).

## 🔹 10. Sloppy Quorum

A relaxation of strict quorum rules.

### 🔸 Problem with Strict Quorum:
Traditional quorum can be too rigid.
If even one or two nodes fail, the quorum might not be reachable → operation fails unnecessarily.

### 🔸 Solution — Sloppy Quorum:
Sloppy quorum relaxes the strict requirement of R + W > N by allowing flexibility in quorum size.

Instead of fixed R and W, you define a range:

```
R ∈ [Rmin, Rmax]
W ∈ [Wmin, Wmax]

```


This allows the system to:
- Continue operating even when fewer nodes are available.
- Improve fault tolerance and availability.
- Adapt dynamically to network or node failures.

**Example:**
For N = 9,
- Strict quorum: R = 5, W = 6
- Sloppy quorum: R between 3–6, W between 3–6
→ If at least 3 nodes respond, the operation proceeds (system remains functional).

## 🔹 11. Advantages of Sloppy Quorum

✅ Higher availability under node failures.
✅ More adaptive and fault-tolerant.
✅ Keeps the system partially operational even during network partitions.

## 🔹 12. When to Use Quorum-Based Systems

**Use quorum-based systems when you need:**
- Strong consistency over pure availability.
- Fault tolerance in multi-node distributed databases.
- Coordination among replicas for correctness.

**Common in:**
- DynamoDB
- Cassandra
- Zookeeper
- MongoDB (Replica Sets)

## 🔹 13. Summary Table

| Concept | Purpose | Formula / Logic | Advantage | Trade-off |
|---------|---------|-----------------|-----------|-----------|
| Read Quorum (R) | Ensures reads are consistent | R = (N+1)/2 | Fresh reads | May increase latency |
| Write Quorum (W) | Ensures writes are durable | W = ceil(N/2)+1 | Strong consistency | Lower availability |
| R + W > N | Guarantee overlap between reads & writes | – | Prevents stale reads | Requires majority |
| Sloppy Quorum | Relaxed quorum | Flexible R/W range | Better availability | Possible temporary inconsistency |

## 🔹 14. Real-World Analogy

Think of distributed nodes as committee members in a company:

- For a decision (operation) to be official, a majority (quorum) must agree.
- If only a few people show up, the meeting can't start.
- In sloppy quorum, even if a few people are absent, the rest can continue and decide — temporarily — to keep the system moving.

## 🔹 15. Closing Thoughts

- Quorum ensures consistency and reliability in distributed systems.
- It prevents conflicting data states during replication or concurrent operations.
- While strict quorum ensures correctness, sloppy quorum ensures flexibility and availability.

Every large-scale system — from distributed databases to consensus protocols — relies on the quorum principle.
