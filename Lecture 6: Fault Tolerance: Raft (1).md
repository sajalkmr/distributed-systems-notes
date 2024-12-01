# Lecture 6: Fault Tolerance: Raft (1)

## Fault Tolerance in Distributed Systems

### Single Master Issue
- Single master simplifies decisions but introduces a **single point of failure**.
- **Goal**: Avoid **split-brain** (conflicting decisions by partitions).
- **Example**: Two clients acquiring the same lock due to network partition.

### Early Solutions
- Relied on:
  - Expensive, non-failing networks.
  - Human intervention.

### Modern Solutions
- Use **majority voting** with odd-numbered servers:
  - Ensures only one partition can form a majority.
  - Progress requires a majority agreement.
  - **2F+1 servers** tolerate **F failures**.
- **Quorum systems** is another term for majority voting.

### Historical Context
- Paxos and Viewstamp Replication (~1990) introduced majority voting.
- Raft builds upon these ideas, similar to Viewstamp Replication.

---

## Overview of Raft

### Components
- **Application Code**: Service-specific logic.
- **Application State**: Replicated data managed by Raft.
- **Raft Layer**: Handles replication and maintains logs.

### Client Interaction
- Clients send requests to the leader:
  - Leader commits operations through majority agreement.
  - Leader applies the operation and sends the response back.

### Fault Tolerance
- Operations are committed after being present in a **majority** of logs.
- Replicas ensure consistent state by replaying committed logs.

---

## Logs in Raft

### Roles of Logs
- **Order Operations**: Ensures consistent application across replicas.
- **Handle Uncommitted Entries**: Stores tentative operations.
- **Retransmit Data**: Leaders resend missed operations after outages.
- **Persistence**: Allows recovery after crashes.

### Challenges
- **Log Growth**: Unbounded growth if leader outpaces followers.
- **Solution**: Implement flow control to throttle the leader.

---

## Recovery from Crashes

### Process
- Servers read persisted logs but wait for the leader to identify the committed point.
- Leader synchronizes logs across replicas.

### Efficiency
- Replaying logs is costly; **checkpoints** provide an optimized solution.

---

## Leader Election in Raft

### Term-Based Leadership
- **Term Numbers**: Distinguish leaders and track progress.
- Only one leader per term.

### Election Process
- Servers start elections when no leader communication is detected.
- Candidates require a **majority vote** to win.
- **Randomized election timeouts** prevent split votes.

### Handling Old Leaders
- Old leaders in minority partitions cannot execute requests.
- Leaders rely on **heartbeat responses** to confirm activity.

---

## Log Divergence and Consistency

### Divergence
- Logs may diverge due to:
  - Leader crashes.
  - Network disruptions.

### Consistency
- Leader ensures all logs are consistent before execution.
- Non-majority entries are discarded.

---

## Key Scenarios in Raft

1. **Scenario 1**:
   - Leader sends a command; some replicas don't receive it before leader crashes.
   - New leader ensures consistency across replicas.

2. **Scenario 2**:
   - Leader sends a command to some replicas; crashes before completing.
   - New leader resolves the commitment.

3. **Scenario 3**:
   - Conflicting entries across replicas.
   - Raft resolves conflicts by retaining entries with majority support.

---

## Election Timeout Tuning

1. **Minimum Timeout**:
   - Must exceed the heartbeat interval to avoid premature elections.
2. **Maximum Timeout**:
   - Shorter timeouts enable faster recovery but may cause unnecessary elections.
3. **Timer Gap**:
   - Must accommodate vote round-trip times for smooth leader selection.
