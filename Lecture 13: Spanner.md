# Lecture 13: Spanner

- **Distributed Transactions**: Spanner enables distributed transactions over data separated by large distances, even across the internet and multiple data centers.
- **Key Features**:
  - Transactions simplify programming and ensure data is consistent across replicas.
  - Data distribution enhances fault tolerance and brings data closer to users.
- **Core Concepts**:
  - **Two-phase commit**: Ensures distributed consistency using Paxos-replicated participants.
  - **Synchronized time**: Facilitates efficient, consistent read-only transactions.
- **Impact**:
  - Used by many Google services and offered as a cloud service.
  - Inspired similar systems, such as the open-source CockroachDB.
- **Motivation**:
  - Google’s need to manage sharded data across MySQL and BigTable databases, particularly for advertising systems.
  - Previous systems lacked support for multi-server transactions and optimal data distribution.
- **Workload**:
  - Dominated by read-only transactions requiring strong consistency, including serializable and external consistency.
  - External consistency ensures that transactions commit in a globally correct order.

---

## Spanner's Physical Arrangement

- **Server Distribution**:
  - Servers are located in data centers worldwide.
  - Each piece of data is replicated across multiple data centers for fault tolerance.
- **Data Sharding**:
  - Data is divided by key into shards, each distributed across several servers.
  - Shards are replicated across multiple data centers.
- **Clients**:
  - Data centers host web servers that act as Spanner clients.
  - Users interact with Google services through these clients, ensuring seamless access to distributed data.

---

## Paxos and Replication

- **Replication Protocol**:
  - Managed via Paxos or similar protocols like Raft, with independent leaders for each shard.
  - Leaders coordinate replication of logs containing data changes to follower nodes.
- **Parallelism**:
  - Sharding allows independent Paxos instances, enabling high throughput and parallel processing.
- **Fault Tolerance**:
  - Multiple replicas ensure resilience against data center failures.
  - Majority agreement in Paxos enables the system to continue operating despite slow or failing nodes.

---

## Challenges

- **Local Reads**:
  - Reads from local data centers may return outdated data if replicas lag behind the leader.
- **Consistency**:
  - Maintaining external consistency for transactions across multiple shards and Paxos groups.
- **Transaction Complexity**:
  - Read-write transactions require coordination across shards, while read-only transactions aim for low latency.

---

## Read-Write Transactions

- **Process**:
  1. Client begins a transaction, reads and writes records.
  2. Writes are committed at the end of the transaction.
  3. Paxos groups ensure consistency by replicating logs.
- **Two-phase Commit**:
  - Ensures atomicity and fault tolerance by involving Paxos-replicated servers.
  - Transaction coordinator manages prepare and commit phases without blocking.
- **Locks**:
  - Read locks are maintained at the shard leader during the transaction.
  - Writes are logged and replicated before being committed.
- **Performance**:
  - Delays can occur due to communication across data centers, but sharding mitigates throughput bottlenecks.

---

## Read-Only Transactions

- **Optimized Reads**:
  - Faster than read-write transactions as they avoid locks, two-phase commit, and transaction managers.
  - Read directly from local replicas, reducing latency significantly.
- **Consistency**:
  - Snapshot isolation ensures consistency by reading data as of a specific timestamp.
- **Latency Improvement**:
  - Typically tenfold compared to read-write transactions, with results identical to serial execution.

---

## Snapshot Isolation

- **Concept**:
  - Transactions are assigned timestamps to simulate serial execution order.
  - Each replica stores multiple data versions, each tagged with a transaction’s timestamp.
- **Implementation**:
  - Read-only transactions query data based on their assigned timestamp.
  - Ensures consistency without affecting concurrent read-write transactions.
- **Example**:
  - Transaction T1 commits with timestamp 10, T2 with timestamp 20.
  - Read-only transaction T3 (timestamp 15) reads values committed by T1 but ignores T2’s updates.

---

## External Consistency and Timestamps

- **Definition**:
  - External consistency ensures transactions appear to execute in a globally correct order.
- **Storage Implications**:
  - Multiple data versions are stored temporarily to maintain snapshot isolation.
  - Old versions are discarded based on garbage collection policies.
- **Safe Time**:
  - Read-only transactions ensure consistency by waiting until local replicas are up-to-date with the requested timestamp.

---

## Time Synchronization

- **True Time (TT)**:
  - Uses intervals (earliest and latest times) to account for clock uncertainty.
  - Transactions commit only after their timestamp falls into the past.
- **Synchronization**:
  - Relies on GPS receivers and data center time masters for accurate clock synchronization.
- **Commit Wait**:
  - Read-write transactions must wait until their timestamp is in the past to ensure consistency.

---

## Key Takeaways

- **Innovative Design**:
  - Combines snapshot isolation with synchronized timestamps for serializability and external consistency.
- **Programmer Benefits**:
  - Simplifies development with strong consistency guarantees.
- **Performance**:
  - Read-only transactions achieve low latency by avoiding complex locking mechanisms.
- **Impact**:
  - Spanner demonstrates the feasibility of distributed transactions across data centers, influencing future systems and research.
