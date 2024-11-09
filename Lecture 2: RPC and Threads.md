# Lecture 2: RPC and Threads

## Why Use Go?

- **Course Requirement**: Labs in this course are written in Go.
- **Concurrency Support**: Excellent support for threads, locking, and synchronization.
- **Remote Procedure Calls**: Convenient package for inter-machine communication.
- **Type and Memory Safety**: Prevents memory-related bugs.
- **Garbage Collection**: No need for manual memory management, simplifying threading.
- **Simplicity**: Easier to learn and debug than languages like C++.

---

## Threads (Goroutines)

- **Concurrency in Go**: Threads (goroutines) allow multiple activities to progress simultaneously.
- **Importance**: Concurrency is essential in distributed programming for interacting with multiple computers.
  
### Understanding Threads

- **Single-threaded Program**: Has one execution path in a shared address space.
  - **Image Placeholder**: Serial Program with One Thread.
- **Multi-threaded Program**: Has multiple execution paths (threads) within a shared address space.
  - **Image Placeholder**: Multithreaded Program with Multiple Threads.

### Reasons for Using Threads

1. **IO Concurrency**: Overlaps activities, e.g., waiting for multiple server responses.
2. **Multi-core Parallelism**: Utilizes multiple CPU cores to increase CPU utilization.
3. **Background Tasks**: Enables periodic activities or background processing, like a master server checking worker nodes.

---

## Challenges of Threaded Programming

1. **Shared Data and Races**: Threads sharing address space may lead to data races.
   - **Example**: Multiple threads incrementing a global variable incorrectly.
   - **Solution**: Use **locks** (mutexes) to control thread access.
   - **Image Placeholder**: Race Condition Diagram.
  
2. **Coordination**: Threads often need to interact or wait for each other.
   - **Coordination Techniques**:
     - **Channels**: Transfer data between threads.
     - **Condition Variables**: Signal waiting threads.
     - **Wait Groups**: Synchronize goroutines.

3. **Deadlocks**: Occur when threads are blocked waiting for each other to release resources.
   - **Classic Scenario**: Thread 1 holds lock A and needs lock B, while Thread 2 holds lock B and needs lock A.
   - **Image Placeholder**: Deadlock Diagram.

---

## Web Crawler Examples

### Serial Crawler
- **Method**: Performs depth-first search on the web.
- **Tracking**: Uses a map to avoid cycles.
- **Limitation**: Does not support parallel fetching.

### Concurrent Crawler (Shared Data and Locks)
- **Parallel Fetching**: Spawns a thread for each fetch.
- **Shared Data**: Uses a mutex-protected shared table of fetched URLs.
- **Completion**: Uses a Wait Group to synchronize completion.
- **Drawback**: No limit on thread creation.

### Concurrent Crawler (Channels)
- **Data Flow**: Master thread holds a private table of fetched URLs, receiving URL lists from workers through a channel.
- **No Shared Data**: Workers don’t share data, removing the need for locks.

---

## Additional Notes

- **Detecting Races**: Use the `-race` flag in Go to identify data races.
- **Closures**: Go closures can retain access to variables from the outer function even after it returns.
- **Maps**: Go maps are pointers, passed by reference by default.
- **Select Statement**: Helps avoid blocking on channel reads.
- **Defer Keyword**: Ensures a function (e.g., `done`) runs before the function returns.

---  
