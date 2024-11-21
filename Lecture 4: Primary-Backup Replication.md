# Lecture 4: Primary-Backup Replication

## Fault Tolerance
- Ensures system continues operation despite component failures.

## Replication
- Maintains multiple copies for fault tolerance, handling fail-stop failures (e.g., power loss, network disconnection).
- Does not handle software bugs, hardware defects, or correlated failures (e.g., shared defects, natural disasters).
- Value of replication depends on the cost of failure.

## Replication Approaches
- **State Transfer**: Primary sends full state to backup.
- **Replicated State Machine**: Sends only events that affect state; preferred due to smaller data size.

## Key Questions in Replication
- What to replicate?
- Synchronization level between primary and backup.
- Client switchover method when primary fails.
- Handling anomalies during switchover.
- Creating new replica on failure.

## VMware FT (Fault Tolerance)
- Replicates full machine state (memory, registers) on separate physical machines.
- Clients connect to primary; backup activated if primary stops logging entries.

## VMware FT Challenges & Solutions
- **Non-deterministic events**: Logged and synchronized with special CPU support.
- **Backup lagging primary**: Backup waits for event buffer before proceeding.
- **Direct Memory Access (DMA)**: Data copied to private memory, emulating NIC interrupts.
- **Output rule**: Primary waits for backup acknowledgment before sending output.
- **Synchronous waits**: Output rule impacts performance, especially over distances.
- **Split-brain problem**: Uses an external test-and-set service as an arbitrator.

---
