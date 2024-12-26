# Lecture 14: Optimistic Concurrency Control

## Introduction

The Farm system is a research prototype that explores high-performance transactions using Remote Direct Memory Access (RDMA). Motivated by the performance potential of new RDMA network interface cards (NICs), Farm addresses the challenge of achieving high performance while maintaining transaction consistency. Unlike Spanner, which focuses on geographic replication and uses two-phase commit, Farm targets single data center replication using optimistic concurrency control.

## Farm vs. Spanner Comparison

### Spanner
- Geographically distributed
- Uses synchronized time
- Implements two-phase commit
- Features read-only transaction optimization
- Read/write transactions take 10-100 milliseconds

### Farm
- Operates in a single data center
- Utilizes RDMA
- Implements optimistic concurrency control
- Processes simple transactions in 58 microseconds (approximately 100x faster than Spanner)
- CPU time is the primary bottleneck (versus Spanner's speed of light and network delays)

## System Architecture

Farm's architecture consists of several key components:
- Runs in one datacenter with a configuration manager (similar to previous systems) using Zookeeper
- Data is sharded across primary/backup pairs
- Replicas are updated on every change
- Transaction code runs as separate clients, acting as transaction coordinators for two-phase commit
- Performance is enhanced through sharding, RAM-based data storage, NVRAM, RDMA, and kernel bypass

## Non-Volatile RAM (NVRAM)

NVRAM addresses the volatility of RAM through:
- Battery backup for each rack
- Power failure protocol: system copies RAM to SSDs before shutdown
- Data restoration from SSD to RAM upon power restoration
- Limited protection against non-power-failure crashes
- Elimination of persistence writes as a bottleneck

## Network Optimization

### Traditional RPC Limitations
Traditional RPC involves multiple components leading to slow performance:
- User space application
- Kernel
- Socket layer
- TCP stack
- NIC hardware and drivers
- System calls
- Interrupts
- Data copying

### Kernel Bypass
Farm implements kernel bypass with the following characteristics:
- Direct NIC access by applications
- Kernel configures protection for direct access
- Direct DMA to application memory
- No system calls or interrupts
- Requires kernel modifications
- Tools like DPDK enable implementation

## Remote Direct Memory Access (RDMA)

RDMA enables direct memory access between computers:
- Special NICs allow direct memory read/write between computers
- Bypasses target computer's CPU
- Implements reliable, sequenced protocol between NICs
- Enables one-sided RDMA for direct memory operations
- Achieves 10 million small RDMA operations per second
- 5 microseconds latency

## Optimistic Concurrency Control (OCC)

Farm implements OCC to handle RDMA and transaction challenges:
- Reads proceed without locking
- Writes are buffered locally
- Validation occurs at commit time
- Conflicts trigger transaction abort
- Compatible with one-sided RDMA operations

## Farm API

The system provides the following API:
- TX_create(): Initiates new transaction
- TX_read(OID): Reads object by identifier
- TX_write(OID, new_object): Updates object (locally buffered)
- TX_commit(): Validates and commits changes
- Object IDs use compound identifiers with region numbers and memory addresses

## Server Memory Organization

Memory is structured as follows:
- Divided into regions containing objects
- Objects include headers with version numbers and lock flags
- System maintains queues and logs for inter-server communication

## OCC Commit Protocol

The protocol consists of three phases:

### Execute Phase
- Client performs RDMA reads
- Initial version numbers are recorded

### Lock Phase
- Client sends lock messages to primaries
- Messages include object ID, version number, and new value
- Primary processes verify locks and version numbers
- Responses indicate success or failure

### Commit Phase
- Requires all primaries to vote yes
- Client sends commit messages via RDMA
- Primary updates object, increments version, and clears lock bit

## Transaction Scenarios

The system handles various transaction scenarios:
- Concurrent transactions with conflict resolution
- Interleaved transactions with version checking
- Sequential transactions with proper ordering
- Guarantees serializability
- Validates transaction consistency

## Validate Optimization

Optimize read-only transactions through:
- Avoiding log writes
- Direct object header verification
- Version number and lock bit checking
- Efficient handling of read-only operations
