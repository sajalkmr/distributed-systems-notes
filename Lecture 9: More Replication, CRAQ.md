# Lecture 9: More Replication CRAQ

## I. Zookeeper
- **Overview**: General-purpose service for distributed systems coordination. Based on Raft, fault-tolerant, handles partitions.
- **API Design**: General-purpose API with mini-transactions.
- **Fault Tolerance**: Handles partitions, fault-tolerant.
- **Performance**: Reads at any replica (stale data possible). Writes ordered across replicas. Single-client operations processed in order.

### Use Cases
- **Test-and-Set Service**: Fault-tolerant service for failovers (e.g., VMware FT).
- **Configuration Management**: Publishes configuration info (e.g., master server IP).
- **Master Election**: Elects a master server, handles partitions.
- **State Storage**: Stores master state; ensures recovery after crashes.
- **Worker Registration**: Registers workers; master assigns tasks.
- **Coordination**: Supports multiple services in a data center.

### Znodes
- **Types**:
  - Regular: Permanent.
  - Ephemeral: Deleted when client session expires.
  - Sequential: Unique with increasing numbers.
- **Operations**:
  - Create/Delete: Conditional based on version.
  - Exists: Checks existence; can set watch.
  - Get/Set Data: Retrieves/updates data; can set watch.

## II. Mini-Transactions
- **Concept**: Supports atomic operations on single data items.
- **Counter Example**:
  - Use loop to get value & version, increment, and conditionally set.
  - Retries on failure, exponential backoff reduces load.
- **Atomicity**: Ensures atomic read-modify-write despite retries.

## III. Locks
- **Basic Lock**: Create ephemeral lock file; delete on release. Suffers from herd effect.
- **Scalable Lock**: Use sequential ephemeral files; clients check lowest number and wait for the next file’s deletion. Avoids herd effect.
- **Use Cases**: Recovery, soft locks (e.g., MapReduce), master election.

## IV. Chain Replication
- **Overview**: Data replicated across a chain of servers.
- **Topology**: Head, tail, and intermediate nodes.
- **Operations**:
  - Writes: Start at head, pass down chain. Tail replies to client.
  - Reads: Handled by tail.
- **Failure Recovery**:
  - Head failure: Next node becomes head.
  - Tail failure: Next node becomes tail.
  - Intermediate failure: Drop node; predecessor resends writes.
- **Performance**:
  - Head sends write once, unlike Raft leader.
  - Reads handled by tail, reducing leader load.

### Configuration Manager
- Decides node status to avoid split-brain scenarios.
- Assigns data to chains and manages updates.

### Comparison with Raft
- **Raft**: Leader processes all reads/writes; needs majority response.
- **Chain Replication**: Splits load between head (writes) and tail (reads). Affected by slow replicas.

## V. Zookeeper vs Chain Replication
- **Zookeeper**: Read-any-replica strategy, less consistency.
- **Chain Replication**: Preserves linearizability; efficient load splitting.
- **API**: Zookeeper offers general-purpose tools (e.g., mini-transactions, locks).

## VI. Additional Notes
- **Duplicate Suppression**: All systems handle duplicate requests.
- **Network Issues**: Misconfigurations can cause split-brain scenarios.
