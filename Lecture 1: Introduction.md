# Lecture 1: Introduction

## What is a Distributed System?

A distributed system is a collection of computers communicating over a network to complete a shared task. Examples include:
- **Storage systems** for large websites
- **Large data computations** (e.g., MapReduce)
- **Peer-to-peer file sharing**

## Why Build Distributed Systems?

Distributed systems are complex and should be used when simpler solutions are insufficient. They offer:
- **High performance**: Utilizing multiple computers’ combined resources for parallelism.
- **Fault tolerance**: Continuing operation despite some failures.
- **Geographical distribution**: Addressing problems that need distributed resources.
- **Security**: Isolating computations and limiting trust between components.

## Challenges of Distributed Systems

- **Concurrency**: Multiple computers lead to complex interactions and timing issues.
- **Partial failures**: Some components may fail while others keep running.
- **Scaling performance**: Achieving proportional speedups with added resources.

---

## Course Structure

This course focuses on building infrastructure for applications, covering:
- **Lectures**: Two per week, mainly case studies.
- **Papers**: Weekly reading with questions.
- **Exams**: Midterm and final, based on papers and labs.
- **Labs**:
  - **Lab 1**: Implement MapReduce.
  - **Lab 2**: Implement Raft for fault tolerance.
  - **Lab 3**: Build a fault-tolerant key-value server.
  - **Lab 4**: Build a sharded key-value service.
- **Optional Project**: A custom distributed system project can replace Lab 4.

---

## Key Concepts in Distributed Systems

- **Abstractions**: Simplified interfaces to hide distributed complexity.
- **Implementation**: Using tools like RPC(Remote Procedure Call) and threads.
- **Performance**: Scaling performance with additional resources.
- **Fault tolerance**: Ensuring continuity despite failures.
  - **Availability**: System operates through certain failures.
  - **Recoverability**: System restores state post-repair.
- **Consistency**: Keeping data consistent across replicas.
  - **Strong**: Ensures recent writes are read.
  - **Weak**: Allows flexibility to improve performance.

---

## MapReduce: A Case Study

**MapReduce** simplifies large-scale data processing on distributed systems.

### Programmer’s Perspective
Defines two functions:
- **Map**: Processes input, produces key-value pairs.
- **Reduce**: Aggregates values for each key.

### System Components
- **Master Server**: Manages job execution.
- **Worker Servers**: Run map and reduce tasks, storing intermediate data locally.
- **Google File System (GFS)**: Distributed file system for input/output data, automatically partitioning large files.

### Performance Considerations
- **Co-location**: Map tasks are run on servers storing the required input, minimizing network use.
- **Shuffle phase**: Transfers intermediate data between map and reduce workers; often a network bottleneck.
- **Modern networks**: Higher throughput has reduced network bottlenecks, allowing more flexible data placement.

---
