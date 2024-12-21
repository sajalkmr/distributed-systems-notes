# Lecture 11: Cache Consistency: Frangipani

## Introduction
- Frangipani is a relatively old distributed file system paper, but it presents interesting and useful designs for cache coherence, distributed transactions, and distributed crash recovery.
- It also examines the interactions between these concepts.
- The system is designed as a network file system that allows existing UNIX applications to run without modification.

## Overall Design

### Workstations
- Each user operates from a workstation running an instance of the Frangipani server.
- The majority of the file system logic resides within the Frangipani software on each workstation.

### Petal
- The real storage of the file system data structures (file contents, inodes, directories, etc.) is stored on a shared virtual disk surface called Petal.
  - Petal replicates data across pairs of servers for fault tolerance.
  - Petal acts as a shared disk drive that Frangipani servers access via remote procedure calls.

### Intended Use
- Designed for a research lab environment with approximately 50 users.
  - It was meant to store home directories and shared project files.
  - Assumes a trusted environment, with no security considerations, unlike systems like Athena.

### Performance
- Most users primarily read and write their own files, with less sharing.
- Frangipani uses caching to improve performance.
- Supports write-back caching, where modifications are initially local until other workstations need to see them. This enables rapid local modifications.
- All the file system logic is in the workstation.
- The decentralized architecture means that adding workstations also increases CPU capacity for file system operations, allowing for natural scaling.

## Design Challenges & Solutions

### Challenge 1: Cache Coherence
- **Problem**: When a workstation creates a file in its local cache, other users need to see the new file. The system needs to ensure that readers see the latest modifications, referred to as strong consistency or linearizability.
- **Solution**: Cache Coherence Protocols ensure that when a cached item is modified, all other caches automatically reflect the modification.
- Frangipani uses locks to drive cache coherence.

#### Lock Servers
- A separate lock server maintains a table of named locks, where each file potentially has a lock.
- Workstations track which locks they hold, linking them to their cached data.

#### Lock Modes
- Locks can be either 'busy' (actively used) or 'idle' (held but not actively used).

#### Rules
- No caching of data without holding the corresponding lock.
- Acquire the lock before reading data from Petal.
- Before releasing a lock, write modified data back to Petal.

#### Messages
- **Request**: Workstations ask for a lock.
- **Grant**: Lock server grants a lock.
- **Revoke**: Lock server requests a workstation to release a lock.
- **Release**: Workstation releases a lock.

#### Lazy Lock Release
- Workstations keep locks after use in an idle state to avoid overhead. This is advantageous for frequently accessed files.

#### Optimizations
- Frangipani uses both shared read locks and exclusive write locks.
- Periodic Write-Back: Workstations write back all modified data every 30 seconds to prevent data loss due to crashes.

### Challenge 2: Atomicity
- **Problem**: Operations like file creation involve multiple steps, and these steps should appear atomic to other workstations.
- **Solution**: Frangipani uses a distributed transaction system driven by locks.
  - Workstations acquire all locks required for an operation and release them only after the operation is complete.
  - Reuses the same lock mechanism for both cache coherence (making writes visible) and atomicity (hiding writes during operations).

### Challenge 3: Crash Recovery
- **Problem**: If a workstation crashes while holding locks and modifying data, the system must recover without corrupting the file system.
- **Solution**: Frangipani uses right-ahead logging.

#### Right-Ahead Logging
- Operations are logged before they are applied.
- Each workstation maintains its own log, stored in Petal, unlike most systems that keep logs locally.

#### Log Entries
- Each log entry contains a log sequence number and descriptions of modifications that include a block number, version number, and data to be written.
- Logs store metadata changes, not file content data.

#### Log Storage
- Workstation logs are initially in memory and are written to Petal when a revoke message is received.
  - Avoids writing log entries to Petal unless necessary.
  - If a revoke message is received, the entire log is written to Petal.

#### Crash Scenario
- If a workstation crashes, the lock server will initiate recovery after a timeout. Another live workstation will read the crashed workstation’s log.

#### Recovery Process
- The recovery workstation scans the log, replaying each operation to Petal.
- If the crashed workstation had not written any log entries, then no recovery is needed.
- The recovery workstation plays changes back into Petal.

#### Version Numbers
- Each piece of metadata in Petal has a version number.
- Log entries also include the version number of the data that they modify plus one.
- During recovery, the version number in a log entry is compared with the version number of the data in Petal.
- If the version number in Petal is greater than or equal to the version number in the log entry, that log entry is skipped because a newer version already exists in Petal.
- This mechanism ensures that recovery is selective, replaying only necessary operations, even if modifications were already written to Petal.

## Performance and Legacy
- Frangipani demonstrated reasonable scalability, where adding workstations did not slow down the system.
- While Frangipani introduced interesting techniques, it did not greatly influence the evolution of storage systems.
- The target environment (small workgroups) is not the primary focus of modern distributed storage systems.
- Modern systems tend to use databases for smaller items of data and systems like GFS for big data computation, which differ from Frangipani's focus.
- Frangipani’s emphasis on caching, cache coherence, and locking are not as useful in big data processing, where caching large datasets is not beneficial.
