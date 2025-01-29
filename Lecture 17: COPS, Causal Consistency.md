# Lecture 17: COPS, Causal Consistency

## Overview
The lecture focuses on causal consistency in distributed systems, using the COPS paper as a case study. It explores large websites with data replicated across multiple data centers, aiming for local reads and writes to improve performance and fault tolerance. The lecture covers various consistency models and their impact on application development.

## Background: Distributed Data Storage
- **Sharding**: Data is distributed across multiple data centers, with each data center managing a subset of the data on different servers.
- **Goals**: Fast reads, functional writes, and consistency.
  
### Examples of Systems:
- **Spanner**: Uses Paxos for consistent writes across data centers, but sacrifices speed. Reads are faster using a true time scheme.
- **MemcacheD**: Has a primary site for writes and local memcache servers for fast reads, with writes involving cross-data center communication.

## Strawman Designs for Consistency

### Design 1: Local Reads & Asynchronous Writes (Eventual Consistency)
- **Concept**: Clients read from local replicas, and writes are acknowledged immediately by the local shard server. The write is then asynchronously streamed to other data centers.
- **Strengths**: Improves performance by enabling local reads and writes.
- **Weaknesses**:
  - Weak consistency: No guarantee of write order across data centers.
  - Potential anomalies (e.g., showing a non-existent photo in a list).
  - Requires defensive programming (e.g., retrying operations if data is missing).

### Version Numbers and Time
- **Wall Clock Time**: Initially used to assign version numbers to writes, but may lead to issues due to unsynchronized clocks.
- **Lamport Clocks**: 
  - Solves clock synchronization issues by maintaining a version number (`Tmax`) that increases with each operation.
  - Ensures that new version numbers are always higher than previous ones.

### Concurrent Writes & Conflict Resolution
- **Last Writer Wins**: The write with the higher timestamp wins.
- **Challenges**: This approach is deterministic but may not be suitable for all cases (e.g., counters).
- **Mini-transactions**: Some systems use mini-transactions for atomic operations to handle conflicts.

### Design 2: Eventual Consistency with Barriers (Sync)
- **Sync Operator**: Introduces a sync operator to enforce order. It waits until all data centers are updated to at least a specified version number.
- **How It Works**: A client inserts a photo and uses sync to ensure consistency across data centers before proceeding with another operation.
- **Weaknesses**:
  - Slow synchronization due to cross-data-center communication.
  - Not fault-tolerant: A data center being down can block the sync call.

### Quorums
- **Real-world implementations** use quorums to avoid blocking during synchronization.

### Logging Approach
- **Log Servers**: Each data center has a designated log server that appends writes and sends them to other data centers in order.
- **Advantages**: Preserves write order.
- **Disadvantages**: The log server becomes a bottleneck.

## Causal Consistency: The COPS Approach
- **Goal**: Enable local reads and writes while ensuring consistency through causal dependencies.
- **Client Context**: Clients accumulate information about the order of operations (dependencies) and send it with each operation.
  
### How It Works:
1. **Client Operations**: When a client performs a put, the context (containing dependencies) is sent to the local shard server.
2. **Shard Server Behavior**: 
   - Assigns a new version number.
   - Forwards the put with dependencies to other data centers.
   - Remote servers delay the write until all dependencies are met in the local data center.

### Causal Dependencies
- **Causal Consistency**: If a client reads an operation `B` that depends on `A`, the client must also see `A`.
- **Transitivity**: Dependencies are transitive, meaning if `A -> B` and `B -> C`, then `A -> C`.
- **Independence of Operations**: Operations that are not causally related can be performed in parallel, enabling better performance.

### Optimizing Client Context
- **Post-Put**: After a put, the client context is replaced by the version number of that put, which is sent to remote data centers along with the dependency.
- **Reduced Overhead**: Remote data centers ensure all dependencies are satisfied before applying the write, reducing the need for maintaining complex client context.

## Limitations of Causal Consistency
- **Limited Tracking**: Causal consistency only tracks dependencies it knows about, so external dependencies (e.g., phone calls, emails) are not managed.
- **Anomalies**: Anomalous behavior can still occur in cases where the system is unaware of causal relationships (e.g., access control lists).
- **Challenges**: Causal consistency is not powerful enough for more complex use cases, such as multi-step transactional operations.

### COPS GT (Causal Consistency with Global Transactions)
- **Improvement**: Returns the full set of dependencies back to the client, allowing the client to fetch the correct data if inconsistencies are detected.

## Real-World Adoption of Causal Consistency
- **Challenges**:
  - Tracking per-client causality is difficult in large systems.
  - Limited support for transactions.
  - Significant overhead for tracking and storing dependency information.
  
- **Status**: While promising, causal consistency has not been widely adopted in production systems due to these challenges.

## Conclusion
Causal consistency provides a good balance between high performance and consistency by tracking causal relationships between operations. However, its adoption in real-world systems is limited due to complexity, tracking overhead, and the difficulty of supporting more complex transactions.
