# Partitioning vs. Sharding

## Partitioning — Splitting data within a single database

### Definition

Partitioning means breaking a large table into smaller, more manageable chunks called partitions, all within the same database instance.

Each partition stores a subset of the total data but follows the same schema (same columns and structure).
To your application, it still looks like one big table — but internally, the database knows which partition to look into for faster access.

### Why Partition?

Databases partition large tables mainly to:

- Improve performance — queries scan less data (fewer rows = faster results).
- Improve manageability — backups, indexing, and maintenance can be done on smaller chunks.
- Enable parallelism — different partitions can be queried or processed in parallel, using multiple CPU cores.

### Example

Imagine a transactions table that stores all payments since 2015. Over time, it's grown to billions of rows.

Instead of one massive table:

You divide it by year →
- `transactions_2022`
- `transactions_2023`
- `transactions_2024`
Now, a query like
```sql
SELECT * FROM transactions WHERE year = 2024;
```
will only scan the 2024 partition — not all years combined. This drastically reduces I/O and speeds up results.

### Common Partitioning Strategies

| Type | How it works | Example |
|------|--------------|---------|
| Range Partitioning | Divide rows by numeric or date ranges | Transactions before 2024 vs. after 2024 |
| List Partitioning | Divide by a list of predefined values | Orders by region: Asia, Europe, US |
| Hash Partitioning | Use a hash function on a key to spread rows evenly | `hash(user_id) % 4` → 4 partitions |
| Composite Partitioning | Combine methods (e.g., range + hash) | Range by year, hash by user_id inside each |

🟢 **Key idea:**
Partitioning optimizes performance within a single database instance.
It does not increase total system capacity beyond that single database's limits.

## Sharding — Splitting data across multiple databases

### Definition

Sharding is like partitioning on steroids.
It's the process of splitting your data horizontally across multiple database servers, where each shard is a completely independent database — with its own CPU, memory, and storage.

Each shard holds a distinct subset of the total data.

### Why Shard?

Sharding becomes essential when:

- The data grows beyond what one machine can handle.
- The number of requests exceeds a single DB's processing capacity.
- You want to reduce latency by storing data close to the users (geo-distribution).
- You need fault isolation — if one shard fails, others keep serving.

### Example (Facebook)

Facebook stores users' data on regional shards:

- Users in Australia → Australian shard (Sydney data center).
- Users in the US → US shard (Virginia data center).

This setup ensures:

- Low latency (nearby servers respond faster).
- If one region goes down, others remain unaffected.
- Workload distribution across the globe.

### Shard Key

A shard key decides where each record goes — which shard will store it.

Common shard key choices:

- User ID hash → balances data evenly.
- Region/Country → groups users geographically (good for latency, risky for uneven load).

⚠️ **Bad shard key = "hot shard"**
If one shard gets way more requests than others, it becomes a bottleneck (e.g., all Indian users on one shard).

### Types of Sharding

| Type | Meaning | Example |
|------|---------|---------|
| Horizontal Sharding | Split rows across multiple DBs | Users A–M → DB1, Users N–Z → DB2 |
| Vertical Sharding | Split columns (attributes) across DBs | User profile data → DB1, User posts → DB2 |

## Fundamental Difference

| Feature | Partitioning | Sharding |
|---------|--------------|----------|
| Scope | Within one database | Across multiple databases/servers |
| Goal | Improve query speed & manageability | Improve scalability & fault isolation |
| Key used | Partition key (e.g., date, ID range) | Shard key (e.g., user ID, region) |
| Complexity | Easier to maintain | More complex (routing, balancing, replication) |
| Failure impact | Whole DB still affected | One shard fails → rest unaffected |

## When to Use Which

| Use Case | Choose |
|----------|--------|
| Large tables slowing down queries | Partitioning |
| Need better backup/archiving | Partitioning |
| Database can't fit on one machine anymore | Sharding |
| Global users with high latency | Sharding |
| Need high throughput & fault isolation | Sharding |

## In Short

- **Partitioning** = Split data within one database to make it faster and easier to manage.
- **Sharding** = Split data across multiple databases to scale horizontally and handle global load.

Or, even simpler:

- Partitioning improves query performance.
- Sharding improves system scalability.

## Analogy (for intuition)

Imagine a library:

- **Partitioning** = Dividing one huge library into labeled sections (Fiction, History, Science) — still one building, just better organized.
- **Sharding** = Building multiple libraries across different cities — each with its own subset of books and staff.
