# 🧭 Raft Consensus Algorithm 

## 🌍 1. Why Do We Need Consensus in Distributed Systems?

Before we touch "Raft," let's first understand the problem it solves.

### 🧩 Imagine:

You have a cluster of servers (nodes) running a distributed system. All nodes are supposed to store the same data (state) — for example, a key-value database, a blockchain ledger, or a Kubernetes control plane.

**But there's a catch:**
- Nodes can fail, restart, or get disconnected (network partition).
- Some nodes might have outdated information.
- Yet the entire system must still behave like a single reliable system.

### 🎯 Goal:
All nodes must agree on the same value / order of operations — even if failures happen.

That agreement is called **consensus**.

## ⚙️ 2. What is Raft?

Raft is a consensus algorithm designed to ensure that multiple nodes in a distributed system agree on a shared state.

### 🧠 Key Idea:
Raft uses a **Leader-Follower model** (one boss, many employees):
- Leader handles all writes and updates.
- Followers replicate leader's data.
- If the leader dies → a new one is democratically elected.

### 💡 Comparison:
Raft was designed as a simpler alternative to another algorithm called **Paxos**, which is conceptually correct but notoriously difficult to understand and implement.

**Raft focuses on:**
- Understandability
- Simplicity of design
- Ease of implementation

## 🧱 3. Core Building Blocks of Raft

There are three main pillars that make Raft work:

1. **Leader Election**
2. **Log Replication**
3. **Safety & Fault Tolerance** (through Heartbeats, Terms, and Commit Rules)

Let's tackle them one by one.

## 🧑‍⚖️ 4. Leader Election — Choosing the Boss

### 🧩 The setup:
At any time, every node is in one of three states:
- **Leader**
- **Follower**
- **Candidate**

### 🪫 Initial State:
All nodes start as followers. If a follower doesn't hear from a leader for some time (no heartbeat), it assumes the leader is dead.

### 🕓 Election Timeout:
Each follower waits for a random election timeout (e.g. 150–300 ms). If no heartbeat is received → it becomes a Candidate.

### 🗳️ Election Process:
1. The Candidate increases its term number (think of it as "election year").
2. It requests votes from all other nodes.
3. Other nodes respond:
   - If they haven't voted yet in this term → they grant their vote.
   - If they already voted for someone else → they reject.
4. When a Candidate receives votes from majority (>50%), it becomes the Leader.

### ❤️ Heartbeats:
- Once elected, the Leader sends periodic heartbeats (empty messages) to all followers.
- Followers reset their election timers upon receiving these.
- This prevents unnecessary elections.
- If followers stop hearing heartbeats → new election begins.

## 🗄️ 5. Log Replication — Keeping Everyone in Sync

Now that we have a leader, it's time to replicate logs.

Each node maintains a **log** (a list of commands that change the system's state).

### 📦 Structure of a Log Entry:
Each log entry contains:
- **Term number** → identifies the term (leader) that created this entry.
- **Command** → actual operation (e.g., "set x = 5").

### 🧭 Process:
1. Client sends a command to the Leader.
2. Leader appends it to its log (uncommitted state).
3. Leader sends AppendEntries RPCs (Remote Procedure Calls) to all followers.
4. Followers append the entry to their logs and reply "ACK."
5. Once the Leader receives ACKs from the majority, it commits the entry.
6. Leader then informs all followers to commit it as well.

✅ The command is now final — the system is consistent.

### 🧩 Why Majority?
Because majority ensures fault tolerance: Even if some nodes crash or disconnect, the system can still reach consensus.

## ❤️‍🔥 6. Heartbeats and Timeouts

- **Heartbeats** are just "I'm alive" signals from the leader.
- Sent at regular intervals.
- Help followers reset their election timers.
- If followers don't get them → they assume leader is dead → election starts.

### ⏱️ Randomized Election Timeout
- Each node has a random timeout value (e.g. 150ms, 200ms, 300ms).
- **Why random?** Because it reduces split votes — if all nodes timed out at the same instant, multiple would declare leadership simultaneously.
- Different timeouts → lower collision probability.

## 🧩 7. Terms — The Logical Timeline

- Raft divides time into **terms**, each term having at most one leader.
- Every term has a unique term number.
- Term numbers strictly increase.
- If a node hears about a higher term → it updates its own term and reverts to follower.

## 🧱 8. Commitment & Safety Rules

Raft ensures **safety** — meaning once a log entry is committed, it cannot be lost or changed, even if failures occur.

### ✅ Commitment Rule:
A log entry is considered committed when:
- It has been replicated on a majority of nodes,
- AND it is stored in the leader's log for the current term.

**Once committed →**
- Leader applies it to the state machine.
- Followers do the same when they learn it's committed.

### 🔒 Log Matching Property:
If two logs contain an entry with the same term number and index, they are identical up to that point. This ensures consistent ordering across nodes.

### 🧠 Leader Completeness Property:
A leader always contains all committed entries from previous terms. This prevents outdated leaders from overwriting good data.

## ⚡ 9. Fault Tolerance Mechanisms

Distributed systems fail — Raft is designed to survive failures gracefully.

### 🧨 (a) Leader Failure:
- Detected via missing heartbeats.
- Election triggered.
- New leader elected → ensures continuity.

### 🧩 (b) Follower Failure:
- If follower is down, leader continues with majority.
- When follower recovers → it catches up by syncing missing logs.

### 🌐 (c) Network Partitions:
- Some nodes may become isolated.
- Majority partition continues operations.
- Minority partition cannot elect a leader (no quorum).
- Once partition heals → followers resync logs automatically.

## 🔍 10. Why Raft Works (and Paxos Is Painful)

| Feature | Raft | Paxos |
|---------|------|-------|
| Model | Leader-based | Multi-proposer |
| Simplicity | Easy to grasp, structured | Conceptually complex |
| Log Handling | Built-in log replication | Must be built separately |
| Understandability | Clear separation of concerns | Hard to reason about |
| Use in real systems | etcd, Consul, TiDB, Kubernetes | Google Chubby, Spanner |

Raft's leader-based approach makes it intuitive and practically implementable without needing a PhD in distributed systems.

## 🧰 11. Real-World Systems Using Raft

| System | Purpose | Why Raft |
|--------|---------|----------|
| etcd | Key-value store used by Kubernetes | Leader election, cluster state consistency |
| Consul | Service discovery, configuration | Consensus on cluster state |
| TiDB / CockroachDB | Distributed SQL databases | Data consistency across replicas |
| Nomad | Job scheduling | Reliable leader coordination |

## ⚠️ 12. Challenges / Limitations of Raft

- **Network Partitions** - Can cause temporary unavailability. Requires careful handling to avoid data divergence.
- **Implementation Complexity** - Simpler than Paxos, yes — but still needs precise handling of edge cases (timeouts, log mismatches, term updates).
- **Single Leader Bottleneck** - All writes go through one leader → limits scalability.
- **Follower Lag** - Followers can lag behind during high load or network delays.

## 🧩 13. Summary — Raft in a Nutshell

| Concept | Purpose |
|---------|---------|
| Consensus | Ensures all nodes agree on same data |
| Leader Election | Choose a single coordinator |
| Log Replication | Copy commands reliably to all nodes |
| Heartbeats | Detect failures and maintain authority |
| Terms | Provide logical ordering of elections |
| Commitment | Guarantee durability and consistency |
| Fault Tolerance | Operate despite node or network failures |

## ✏️ 14. Recommended Figures to Draw

1. Leader-Follower State Diagram (Leader ↔ Candidate ↔ Follower)
2. Log Replication Flow (Client → Leader → Followers → Commit)
3. Timeline showing terms and leadership transitions
4. Majority commit diagram (5 nodes → 3 required for majority)
5. Heartbeat Timeout Illustration

## 🧠 15. Mental Model — How to Remember Raft

Think of Raft as a company:

- **The Leader** = CEO (makes final decisions)
- **Followers** = Employees (execute CEO's commands)
- **Heartbeats** = Daily check-ins ("I'm alive")
- **Election Timeout** = "No word from CEO for 3 days — time to elect new one!"
- **Logs** = Meeting notes replicated across all employees
- **Commit** = Action plan agreed upon by majority
- **Terms** = Financial years (one CEO per year)
