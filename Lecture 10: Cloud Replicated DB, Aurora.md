# Lecture 10: Cloud Replicated DB, Aurora

## Introduction  
- Case study on Aurora, a high-performance cloud database service by Amazon.  
- Aurora achieves a **35x speedup** in transaction throughput compared to certain systems.  

## Evolution of AWS Database Infrastructure  
### Early Days: EC2 and Local Storage  
- EC2 VMs with locally attached disks.  
- Issues: Data loss on hardware failure, periodic backups via S3.  

### EBS: Elastic Block Store  
- Fault-tolerant storage with replicated data across servers.  
- Challenges: High network traffic and limited fault tolerance to data center failures.  

### RDS: Replicated Database Service  
- Databases replicated across availability zones using EC2 and EBS.  
- Issues: Expensive and slower due to network overheads.  

## Aurora's Design Principles  
### Custom Software and Storage  
- Uses Amazon's custom database server and storage.  
- Sends only log records over the network (smaller than data pages).  
- Storage is optimized for MySQL logs, unlike general-purpose block storage.  

### Quorum-Based Writes  
- Requires acknowledgment from 4 out of 6 replicas (quorum).  
- Allows better fault tolerance and performance.  

### Performance Gains  
- Log-based design and quorum system enable a **35x throughput improvement**.  

## Quorum Details  
- **Fault Tolerance Goals**: Operates with a dead availability zone and slow replicas.  
- **Classic Quorum**: Ensures consistency using overlapping read/write quorums.  
- **Aurora’s Quorum**: `n=6`, `w=4`, `r=3`, tolerates up to 3 dead servers for reads.  

## Storage Server Details  
- Stores data pages and logs of modifications.  
- Updates pages on read requests by applying log records.  
- Tracks log progress for optimized reads.  
- Crash recovery via quorum reads and undo operations.  

## Scalability and Sharding  
- Data split into 10GB **protection groups**, each with 6 replicas.  
- Parallel log distribution and recovery for faster replication.  

## Read Replicas  
- Supports up to 15 read-only replicas for read-heavy workloads.  
- Read replicas use log entries for updates, lagging slightly behind the main server.  
- Uses **mini-transactions** for atomic updates visible to replicas.  

## Key Takeaways  
- **Transaction Databases**: Aurora integrates database and storage layers for performance.  
- **Quorums**: Ensure fault tolerance and consistency.  
- **Cloud Infrastructure**: Handles failures, slowness, and network bottlenecks effectively.  
- **Network Bottleneck**: Minimized by reducing data sent over the network.  
- **Tradeoffs**: Prioritized storage work over network use; CPU is cheaper than bandwidth.  
