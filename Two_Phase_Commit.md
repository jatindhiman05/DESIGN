# 🌐 Topic: Two-Phase Commit (2PC)

## 🚩 1. Introduction

Let's start with the why.

In distributed systems, data is stored across multiple nodes (servers/databases). Suppose a piece of data D is replicated on all nodes:
```
Node1: D
Node2: D
Node3: D

```

Now, an update happens — D → D1. Ideally, all nodes should now hold D1.

But what if one node crashes or becomes unreachable while others update?

Then we end up with:
```
Node1: D1
Node2: D1
Node3: D ← failed to update

```


This means inconsistency — one node shows the old data, another shows the new one. That's unacceptable for any system where correctness matters (like banking, payments, etc.).

So, we need a protocol that guarantees:

**Either all nodes commit the change or none of them do.**

That's exactly what Two-Phase Commit (2PC) does.

## ⚙️ 2. Definition

**Two-Phase Commit (2PC)** is a distributed transaction protocol used to ensure atomicity and consistency across multiple nodes in a distributed system.

It ensures that:

- A transaction is either fully committed on all nodes,
- or fully rolled back on all nodes — no partial updates, no inconsistency.

It uses two kinds of entities:

- **Coordinator (Master)** — the one who initiates and controls the transaction.
- **Participants (Slaves or Cohorts)** — nodes that actually perform the operation.

Together, they vote and agree on the final outcome.

## 🧩 3. Why Two-Phase Commit?

Because distributed transactions are hard. When multiple databases or nodes are involved, several things can go wrong:

- A node might fail mid-transaction.
- Network partitions may isolate nodes.
- Some nodes might commit while others haven't yet.

We need a coordination mechanism that ensures:

- All nodes agree on whether to commit or abort.
- The final state is consistent — even in case of failures.

That coordination is what 2PC provides.

## 🧱 4. Key Idea: "All or Nothing"

Two-Phase Commit enforces atomicity across the entire distributed system.

Either:
- Commit everywhere,
- Or Rollback everywhere.

There's no in-between.

## 🧠 5. Components

| Component | Role |
|-----------|------|
| Coordinator | Initiates and manages the distributed transaction. Collects votes and decides final outcome. |
| Participants (Cohorts) | Execute transaction locally when asked, vote "YES" (ready to commit) or "NO" (abort). |

## 🔄 6. The Two Phases

Let's break it down:

### Phase 1: Voting Phase (Prepare Phase)

- The Coordinator sends a "Prepare to Commit?" request to all participants.
- Each Participant performs all necessary checks (resources, constraints, etc.) and responds:
  - **YES** → ready to commit
  - **NO** → cannot commit (due to failure, validation error, etc.)
- The coordinator collects all votes.

**In short:** everyone votes on whether they're ready to commit.

### Phase 2: Commit or Rollback Phase

Based on the votes:

- **If all participants voted YES** →
  - The Coordinator sends a COMMIT message to all.
  - All participants commit the transaction permanently.
  - Each sends an acknowledgment back ("Done").

- **If any participant voted NO** →
  - The Coordinator sends a ROLLBACK message to all.
  - All participants abort the transaction and revert any changes.

**Everyone either commits or rolls back. No partial updates.**

## 🧮 7. Example: Updating Data Across Nodes

**Step 1** — Coordinator updates local data: Data changes from D → D1.

**Step 2** — Coordinator asks others: Are you ready to commit D1?

**Step 3** — Responses:
- Node1 → YES
- Node2 → YES
- Node3 → NO (failure or unavailable)

**Step 4** — Coordinator decides:

Since not all said YES → Coordinator sends ROLLBACK to everyone.

No node commits the change. Data remains consistent (all have D).

*If everyone said YES → Coordinator sends COMMIT to everyone. All update to D1.*

## 🧭 8. Visual Summary
```text
          ┌──────────────┐
          │ Coordinator  │
          └──────┬───────┘
                 │
     Phase 1: Prepare (Voting)
                 │
   ┌─────────────┴─────────────┐
   │           │               │
 Node1       Node2           Node3
  YES          YES             NO
   │           │               │
   └─────────────┬─────────────┘
                 │
     Phase 2: Decision (Commit or Rollback)
                 │
          ┌──────┴──────┐
          │ Rollback All│
          └─────────────┘
```


## 💪 9. Advantages

| Advantage | Description |
|-----------|-------------|
| Atomicity Guaranteed | Either all nodes commit or none do. Prevents partial updates. |
| Data Integrity | Keeps distributed data consistent across nodes. |
| Reliability | Provides fault tolerance against minor communication issues. |
| Synchronization | Ensures all nodes are in sync about transaction state. |

## ⚠️ 10. Limitations / Disadvantages

| Problem | Explanation |
|---------|-------------|
| Blocking Problem | If a participant or coordinator crashes mid-transaction, other nodes may have to wait indefinitely (system "blocks"). |
| Single Point of Failure | The coordinator is central. If it fails, the entire transaction can hang or fail. |
| Performance Overhead | Requires multiple network round-trips and acknowledgments, slowing down performance. |
| Scalability Issues | As number of participants increases, coordination overhead grows exponentially. |

## 🧩 11. Real-World Applications

Two-Phase Commit is used in systems that require strict consistency and atomic updates across distributed components.

**Examples:**

- Distributed Databases (Oracle, MySQL, PostgreSQL clusters)
- Banking Systems (account debits/credits)
- Payment Gateways
- E-commerce Transactions (inventory + payment coordination)
- Stock Exchanges / Financial Trading
- Booking Systems (ensuring single seat/ticket allocation)

## 🔁 12. Summary Table

| Aspect | 2PC Behavior |
|--------|--------------|
| Purpose | Maintain atomicity and consistency across distributed nodes |
| Phases | 1️⃣ Voting (Prepare) 2️⃣ Commit/Rollback |
| Coordinator Role | Collect votes, decide outcome |
| Participant Role | Vote YES/NO, execute commit or rollback |
| Guarantees | Atomicity & Consistency |
| Drawbacks | Blocking, single point of failure, performance cost |
| Common Use | Financial systems, distributed databases |

## 🧠 13. Conceptual Analogy

Think of it like a group decision:

- You're the coordinator planning a group trip.
- You ask all friends, "Ready to book tickets?"
- If everyone says YES, you book for everyone (commit).
- If anyone says NO, you cancel the plan (rollback).

Nobody ends up in a half-committed state where half the group has tickets and half doesn't.

That's 2PC in human terms.

## ⚡ 14. Drawbacks → Why 3PC Exists

Because 2PC can block indefinitely if coordinator or participant crashes. So, **Three-Phase Commit (3PC)** adds an extra phase to make it non-blocking and more fault-tolerant.

But 3PC comes at an even higher communication and performance cost.

## ✅ 15. Final Takeaway

Two-Phase Commit ensures distributed atomicity — a transaction across multiple systems either succeeds completely or fails completely.

It's the backbone of consistent distributed databases but comes at the cost of speed and fault-tolerance.

**Use 2PC when correctness matters more than performance.** Use alternative strategies (like eventual consistency or 3PC) when scalability and uptime are top priorities.
