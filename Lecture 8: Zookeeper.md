# Lecture 8: Zookeeper

## Linearizability

- **Definition**: Linearizability is a standard definition for strong consistency in storage systems.
- **Purpose**: Helps determine if a sequence of operations in a system is acceptable and how systems might fall short of strong consistency.
- **Checking Linearizability**:
  1. Find a total order of operations that matches real-time and where each read gets the value of the most recent preceding write.
  2. Identify a cycle in the "comes before" graph, which proves non-linearizability.
- **Real-Time Rule**: If a request starts after another request finishes, it must occur after the first request in the total order.

### Example 1: Linearizable History
- **Scenario**:
  - Client writes X=0.
  - Another client writes X=1.
  - Concurrent client writes X=2.
  - Reads occur: first read gets 2, second read gets 1.
- **Linearizable Order**: `write X=0, write X=2, read X=2, write X=1, read X=1`.
- Matches real-time and ensures each read observes the most recent preceding write.

### Example 2: Non-Linearizable History
- **Scenario**:
  - Client 1 reads X and sees 2, then reads X again and sees 1.
  - Client 2 reads X and sees 1, then reads X again and sees 2.
- **Analysis**:
  - Different clients observe conflicting orders of writes.
  - A "comes before" graph creates a cycle, proving non-linearizability.
  - Problem arises in replicated systems with inconsistent data copies.
- **Solution**: Linearizability ensures the system behaves as if there is only one copy of the data.

### Example 3: Stale Data
- **Scenario**:
  - Client 1 writes X=1.
  - Client 2 writes X=2.
  - Client 3 reads X and gets 1.
- **Analysis**: This is not linearizable because the read gets stale data.

### Example 4: Client Retransmissions
- **Scenario**: A client sends a read request, gets no response, and resends the request.
- **Server Responsibility**: Filter out duplicate requests and return the original reply.
- **Analysis**: A stale value is acceptable if the read is concurrent with a write.
- **Implication**: Applications must tolerate delayed responses in linearizable systems.

## Zookeeper

- **Overview**: Zookeeper is a successful, real-world, open-source coordination service used in many software systems.
- **Comparison**:
  - Unlike Raft, which is a library, Zookeeper is a standalone service.
  - Uses the Zab protocol, similar to Raft, for replication.
- **Performance**:
  - Adding more servers may not improve performance as the leader is a bottleneck.
  - The leader processes all requests and sends them to replicas.

### Read Performance in Zookeeper
- **Optimization**: Read requests can be sent directly to replicas.
- **Tradeoff**: Leads to stale data.
- **Linearizability**: Not provided for reads to allow improved read performance.

### Zookeeper's Consistency Guarantees
- **Linearizable Writes**: Behaves as if writes are executed one at a time in some order, obeying real-time ordering.
- **FIFO Client Order**:
  - Client operations execute in the order specified by the client.
  - Applies to both writes and reads.
  - Writes must follow client-specified order, even asynchronously.
  - Reads must observe points in the log that do not regress.
- **Mechanisms**:
  - Each log entry is tagged by the leader with a ZXID.
  - Replica responses include the highest ZXID seen to ensure consistency.
  - If a replica lags behind the client’s ZXID, it waits or rejects the read.
  - Reads observe their own client’s writes.
- **Stale Reads**: While Zookeeper does not guarantee fresh data for reads, a client’s reads will observe its own writes.

### Programming with Zookeeper's Guarantees
- **Guarantees**: Not as strong as linearizability but sufficient for many applications.
- **Sync Operation**: Ensures that a subsequent read observes data at least as up-to-date as the sync operation in the log.
- **Ready File Example**:
  - **Master Updates**:
    - Deletes a "ready" file.
    - Writes configuration files.
    - Recreates the "ready" file to indicate the configuration is ready.
  - **Worker Behavior**:
    - Checks for the "ready" file before reading the configuration.
    - FIFO client order ensures that a recreated "ready" file guarantees reading the most recent configuration.
  - **Problem**: Clients might read a mix of old and new configuration data if they act prematurely.
  - **Solution**: Use "watches" to notify clients of changes.
    - Clients set a watch on a file to get notifications of changes before reading.
    - Replica maintains a watch table indexed by filename.
    - Notifications ensure consistent reads.
- **Handling Crashes**: If a replica crashes, clients must reset their watches and are informed through notifications.
