# Lecture 7: Fault Tolerance: Raft (2)

## Fault Tolerance in Distributed Systems
- Fault tolerance is the ability of a system to continue functioning properly despite component failures.
- **Importance**:
  - Distributed systems are more prone to failures than single-machine systems.
  - Fault tolerance ensures reliability and consistency in such environments.

## Introduction to Raft
- Raft is a **consensus algorithm** designed to be simple and easy to implement.
- Ensures that a set of servers can agree on a series of commands even in the presence of failures.

---

## Log Replication and Consistency
- Raft uses **log replication**:
  - Each server maintains a **log of commands**.
  - The **leader replicates its log** to followers, which apply the commands to their state.

### Persistence
- Logs must be stored in **non-volatile storage** (e.g., disk) to recover from crashes.
- **Key information to persist**:
  - Log entries.
  - Current term.
  - Voted-for information.

### Detecting Discrepancies
- The leader ensures consistency using **AppendEntries RPCs**:
  - Contains metadata about the previous log entry.
  - If followers' logs don't match, the leader decrements the `nextIndex` and retries.

### Handling Uncommitted Entries
- Log entries not committed by a majority can be erased safely:
  - Uncommitted entries are considered tentative.
  - Clients resend requests if no response is received.

---

## Leader Election in Raft
- **Election Rules**:
  - A server votes for a candidate if:
    1. The candidate has a **higher term** in its last log entry.
    2. Or, the candidate has the **same term** but a log length greater than or equal to the voter's log length.
  - These restrictions ensure the elected leader has the most up-to-date log.

---

## Log Compaction and Snapshots
- Raft addresses **log growth** through **log compaction** and **snapshots**:
  - Prevents performance degradation caused by replaying large logs on restart.

### Snapshots
- A snapshot is a **compact representation of the application state** at a specific log index.
- **Process**:
  1. Raft requests the application to create a snapshot when the log grows too large.
  2. The snapshot and subsequent log entries are persisted to disk.
  3. Older log entries (before the snapshot point) are discarded.

### Recovery with Snapshots
- On restart:
  - Raft retrieves the **latest snapshot** and log entries from disk.
  - Initializes the application state using the snapshot.

---

## InstallSnapshot RPC
- Used when a follower's log ends before the leader's log starts.
- The leader sends:
  1. Its current **snapshot**.
  2. An **AppendEntries RPC** containing subsequent log entries.

---

## Linearizability
- A correctness condition for replicated services.
- Requires:
  1. A **total order of operations** that matches the **real-time order** for non-concurrent requests.
  2. Reads see the value of the most recent preceding write.

---

## Key Points
1. **Fault Tolerance**:
   - Essential for distributed systems to ensure reliability.
2. **Raft Consensus**:
   - Simplifies consensus with clear rules for leader election and log replication.
3. **Log Replication**:
   - Ensures consistency and persistence across servers.
4. **Leader Election**:
   - Prioritizes leaders with the most up-to-date logs.
5. **Log Compaction**:
   - Addresses log growth through snapshots and efficient recovery mechanisms.
6. **Linearizability**:
   - Defines correctness for replicated services.
