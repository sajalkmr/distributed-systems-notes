# Lecture 3: GFS

## Big Storage Systems and the Challenges They Face

### Overview
- Storage is a foundational abstraction in distributed systems, essential for many other systems.
- **Main Challenge**: Balancing high performance with consistency, especially when handling faults.
- **Sharding**:
  - Sharding data across servers enables parallel reads.
  - However, sharding introduces fault tolerance challenges, risking data loss if servers fail.
- **Replication**:
  - Provides fault tolerance but may lead to inconsistencies if replicas diverge.
  - Strong consistency (behaving like a single server) is desirable but can impact performance.

---

## Challenges with Replication

- **Simple Replication Design**:
  - Sending writes to multiple servers without order can cause inconsistencies.
  - Clients may read different values for the same data, which breaks the consistency expectation.
  - Directing reads to specific servers can still result in unexpected data changes during failures.

---

## Google File System (GFS): Addressing the Challenges

### Goals of GFS
- **Scalability**: Supports vast datasets and high client loads.
- **Global File System**: Accessible by various Google applications.
- **Automatic Failure Recovery**: Self-heals without human intervention.

### Non-Goals of GFS
- **Geographic Distribution**: Designed for a single data center.
- **External Service**: Intended for Google’s internal use.
- **Small Random Access**: Optimized for large sequential reads/writes.
- **Low Latency**: Prioritizes high throughput over response time.

### GFS Architecture
- **Single Master Architecture**:
  - The master handles metadata and coordinates data access on chunk servers.
- **Data Storage**:
  - Files are divided into 64 MB chunks, each chunk replicated across multiple servers.
- **Metadata Management**:
  - **File Table**: Maps file names to chunk IDs.
  - **Chunk Table**: Maintains chunk details, replica locations, version numbers, and primary server info.
- **Data Persistence**:
  - Master stores metadata on disk and in memory.
  - Changes recorded in a log; state checkpointed periodically.

---

## GFS Reads

1. **Client Request**: Sends file name and offset to the master.
2. **Master Lookup**: Identifies the chunk and returns its handle and server locations to the client.
3. **Client Caching**: Caches chunk location info for future reads.
4. **Chunk Server Request**: Sends handle and offset to a chunk server (preferably on the same rack).
5. **Data Retrieval**: Chunk server reads data from disk and sends it to the client.

---

## GFS Record Appends

1. **Client Request**: Sends file name and data to be appended to the master.
2. **Master Lookup**: Identifies the last chunk and checks for a primary server.
3. **Primary Selection**:
   - If no primary exists, the master assigns one with the latest chunk version.
   - Increments version number and informs primary and secondaries.
4. **Data Distribution**: Client sends data to all replicas.
5. **Append Execution**:
   - Primary assigns an offset, instructs replicas to append data at that offset.
   - Replicas acknowledge the write.
6. **Response**:
   - Success: Primary confirms all replicas succeeded.
   - Failure: Primary reports failure, and client retries.

---

## GFS Consistency Model

- **Relaxed Consistency**:
  - Allows temporary inconsistencies among replicas.
  - Successful appends eventually align all replicas at the same offset.
  - Failed appends may cause discrepancies, requiring applications to manage inconsistencies with checksums and record boundaries.

---

## Challenges and Limitations

- **Single Master Limitations**:
  - **Scalability**: Limited by master’s memory as files/chunks increase.
  - **Performance Bottleneck**: Thousands of client requests can overwhelm the master.
- **Lack of Strong Consistency**: Some applications struggle with GFS’s relaxed consistency.
- **Manual Master Failover**: Recovery from master crash needs human intervention, leading to downtime.

---

## Achieving Stronger Consistency (Hypothetical)

- **Duplicate Request Detection**: Avoids redundant writes.
- **Enforced Secondary Execution**: Ensures secondaries execute primary’s instructions reliably.
- **Secondary Failure Handling**: Quickly removes faulty secondaries.
- **Two-Phase Commit**: Coordinates writes for consistent commit/abort across replicas.
- **Primary Crash Recovery**: Allows new primary to sync with secondaries post-crash.
- **Read Path Management**: Routes reads through the primary or uses a lease system for secondaries.

Note: These modifications improve consistency but add complexity and communication overhead, impacting performance.
