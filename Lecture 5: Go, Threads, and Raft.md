# Lecture 5: Go, Threads, and Raft



## The Go Memory Model
- Reading material emphasizes writing correct threaded code in Go with examples of pitfalls.
- Focus on practical application over deep theoretical understanding of the happens-before relation.
- Threads should aid clarity; complex optimizations (e.g., fine-grained locking) are discouraged.
- Avoid identifier capture issues by passing the loop variable as an argument in goroutines.

## Go Concurrency Primitives
- **Closures**: Useful for goroutines to reference and mutate enclosing scope variables.
- **Periodic Tasks**: Use infinite loops with `time.Sleep` for intervals.
- **Controlling Goroutine Lifetime**: Shared variables (e.g., "done") control lifecycle, and mutexes ensure thread safety.
- **RF.killed** method in Raft simplifies managing periodic goroutines.
- **Synchronization Primitives**: Channels and locks ensure proper cross-thread communication and updates to shared variables.
- **Mutexes**:
  - Mutual exclusion to prevent simultaneous execution in critical sections.
  - Atomic updates avoid data corruption.
  - Use `defer` to unlock mutexes, ensuring they are released at function end.
  - Protect invariants, ensuring properties of shared data remain true even with concurrent modifications.
- **Condition Variables**:
  - Allow threads to wait for specific conditions on shared data.
  - Pattern:
    - Use `cond.Broadcast()` to notify changes, `cond.Wait()` for waiting in a loop.
    - `cond.Wait()` releases the mutex and reacquires it upon returning.
    - Prefer `cond.Broadcast()` over `cond.Signal()` for simplicity.

## Channels in Go
- Synchronous communication between goroutines; sends/receives block until both sender and receiver are ready.
- **Unbuffered Channels**: Enforce strict synchronization.
- **Buffered Channels**: Offer storage but can add complexity; use only when needed.
- **Use Cases**:
  - Producer-consumer patterns.
  - Wait-group-like functionality for waiting on multiple goroutines.
- **Advice on Channels**:
  - Avoid single-goroutine channel use to prevent deadlocks.
  - Consider mutexes/condition variables if channels add complexity.
  - Use buffered channels sparingly.

## Debugging Tips
- **DPrintf**: Helpful for debugging concurrent code, allowing toggled logging.
- **Strategic Placement**: Narrow down bug scope by placing DPrintf statements.
- **SIGQUIT Signal**: Triggers a stack trace, useful for identifying deadlocks.
- **-race Flag**: Detects race conditions in tests.
- **Race Detector Output**: Provides clues for resolving race issues.
- **Understanding Test Scenarios**: Essential for effective debugging, especially for Raft.
- **Parallel Test Scripts**: Helpful for exposing concurrency bugs.

---

