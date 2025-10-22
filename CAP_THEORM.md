CAP Theorem (Consistency, Availability, Partition Tolerance)
Definition

The CAP Theorem, proposed by Eric Brewer, states that in a distributed system, it is impossible to guarantee all three of the following properties simultaneously once a network partition (communication failure between nodes) occurs.

These three properties are:

C — Consistency

A — Availability

P — Partition Tolerance

In simple terms:

When parts of a distributed system can’t talk to each other (due to network issues or node failure), you must choose between consistency and availability.
You can’t have both at the same time.

1. Consistency (C)

Meaning:
Every read operation returns the most recent write or an error.
That means all nodes (servers or databases) in the system always show the same data at any given moment.

Example:
Imagine you withdraw ₹1000 from your bank account using an ATM.
When you immediately check your balance in the banking app or another ATM, you should see the reduced amount — not the old balance.
If the balance doesn’t match everywhere, that would be inconsistency.

How it works internally:
To ensure consistency, the system synchronizes updates across all nodes before confirming a write operation.
Until every node agrees on the new data, the system might block other reads or writes — even rejecting some requests — to avoid serving stale data.

Trade-off:
Maintaining strong consistency requires waiting for all replicas to update, which can:

Increase latency (slower response).

Reduce availability (some requests may fail if any node is unreachable).

In short:

Consistency ensures accuracy, not speed.

2. Availability (A)

Meaning:
Every request gets a response — even if some nodes are down or not communicating.
The response might be slightly outdated, but the system never becomes unresponsive.

Example:
Suppose you open Instagram, and a post shows 10 likes, while your friend’s phone shows 12 likes.
That’s because one node has updated data while another hasn’t synced yet.
Instagram chooses availability — you get to see something instantly, instead of waiting for the system to synchronize likes across all nodes.

Why it matters:
For user-facing applications like social media or streaming, users prefer speed and uptime over strict accuracy.
Seeing a slightly stale count of likes or views is better than seeing an error or blank screen.

Trade-off:
If the system stays available at all costs, it might:

Return stale data (outdated information).

Temporarily violate consistency between nodes.

In short:

Availability ensures responsiveness, not accuracy.

3. Partition Tolerance (P)

Meaning:
The system continues to function correctly even when communication between nodes fails — i.e., when there’s a network partition.

What is a network partition?
It’s when parts of a distributed system can’t talk to each other due to network issues, latency, or node crashes.
Think of it like two data centers — one in Mumbai and another in New York — losing network connectivity.
They are both still running but can’t synchronize data temporarily.

Example:
If the Mumbai data center goes offline or loses connection to New York, users in both regions should still be able to use the service independently.
That’s partition tolerance — the system keeps serving users instead of shutting down.

Why it’s non-negotiable:
In any large distributed setup (like Google, Netflix, or Amazon), network failures are inevitable.
Cables fail, routers crash, latency spikes — partitions will happen.
Hence, every real-world system must tolerate partitions to some degree.

In short:

Partition Tolerance means the system stays operational despite communication failures.

The Core Trade-off

Because network partitions are unavoidable, we can’t drop P.
So, in reality, we can only choose between C (Consistency) and A (Availability) when a partition happens.

That gives us two practical types of systems:

1. CP Systems (Consistency + Partition Tolerance)

Meaning:
System favors correctness (same data everywhere) over uptime.
If nodes can’t talk to each other, some requests are rejected to maintain consistent data.

Example:

Databases: PostgreSQL, MongoDB (with strong consistency), HBase

Use Cases: Banking, financial transactions, inventory systems

Behavior:
If a network failure occurs, the system stops serving certain requests rather than risk inconsistent data.

Scenario:
You try to withdraw ₹1000 during a partition — your request might fail temporarily instead of showing a wrong balance.
Better to reject than to show incorrect money data.

CP = Data accuracy first, uptime second.

2. AP Systems (Availability + Partition Tolerance)

Meaning:
System favors uptime and user experience over strict accuracy.
Even if nodes are out of sync, the system keeps serving requests.

Example:

Databases: Cassandra, DynamoDB, CouchDB

Use Cases: Social media, e-commerce, video streaming

Behavior:
During a network partition, different nodes may have slightly different data.
Once the network heals, they reconcile differences using eventual consistency (data becomes consistent over time).

Scenario:
Two users like a photo on different servers — both see their action immediately, and the servers sync later.

AP = Always on, even if slightly out of sync.

3. CA Systems (Consistency + Availability)

Meaning:
System ensures both accuracy and uptime — but only as long as there are no partitions.
Once communication breaks, one of them must be sacrificed.

Reality Check:
This only works in theoretical or single-node systems (like traditional relational databases running on one machine).
Once you distribute data across a network, P becomes unavoidable.

Challenges in Applying CAP

Trade-off decisions are application-specific:
Each system must decide what matters most — accuracy (C) or availability (A).
Banking needs consistency; social media needs availability.

Design complexity:
Building systems with partial or eventual consistency adds engineering overhead (conflict resolution, replication logic, data reconciliation).

Performance impact:
Synchronizing data across nodes adds latency and replication delays, especially in global systems.

User experience vs. correctness:
You must balance what’s acceptable inconsistency for the user.
(E.g., seeing an old “like count” is fine; seeing the wrong account balance isn’t.)

Conclusion

The CAP Theorem is a fundamental guideline for distributed system design.
It reminds architects and engineers that:

🧠 “When a network partition occurs, you must choose between Consistency and Availability.”

There’s no perfect system, only well-informed trade-offs based on business needs.

Choose CP → When accuracy and correctness are critical.

Choose AP → When uptime and user experience are more important.

Ultimately, CAP helps you design distributed systems intelligently, knowing what you’re giving up — and why.