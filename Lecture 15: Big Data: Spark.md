# Lecture 15: Big Data: Spark

## Introduction

- Spark is presented as a successor to MapReduce, representing an evolutionary step in data processing. It is widely used for data center computations due to its utility and popularity.
- A key difference from MapReduce is Spark's generalization of the two-stage (map and reduce) approach to a more flexible, multi-step data flow.
- Spark supports iterative applications more efficiently than MapReduce, which may require multiple applications chained together.

## PageRank Example

- The PageRank algorithm is used to demonstrate Spark’s capabilities.
  - PageRank is described as an algorithm used by Google to calculate the importance of web search results.
  - It is noted that PageRank does not work very well with MapReduce because it requires multiple distinct steps and iterations.
- The input for PageRank consists of lines of links between URLs, which is typically derived from crawling the web.
- The goal is to estimate the importance of a page based on links from other important pages, modeling the probability a user will land on a page by following links.
  - A user has an 85% chance of following a link and a 15% chance of switching to a random page.
- The algorithm simulates user clicks repeatedly to update page ranks, which are tracked for every URL.
- In MapReduce, implementing PageRank is cumbersome due to the iterative nature of the algorithm, which requires multiple calls to MapReduce applications and involves file I/O with GFS.

## Spark Programming Model

- The Spark programming model is introduced through an example of running PageRank code.
  - It emphasizes that downloading and running Spark is straightforward.
- The Spark shell allows for interactive execution of code line by line, which helps in understanding the programming model.
- When Spark reads a file, it reads from a distributed file system like HDFS, splitting the file into chunks over multiple servers.
  - Each worker may be responsible for multiple partitions of the input file.

## Lineage Graph

- Instead of immediately processing data, Spark builds a lineage graph (a recipe for the computation).
  - The actual computations only begin when an action, such as `collect`, is called.
  - The lineage graph represents the transformations that need to be performed.
- The `collect` action triggers Spark to fetch data, compile the lineage graph into bytecodes, and send those bytecodes to worker machines.
- Transformations are operations that produce new RDDs (Resilient Distributed Datasets) from existing ones. Examples include:
  - `map`: Applies a function to each element in the input, operating on each line of input independently, allowing for parallel execution.
  - `distinct`: Removes duplicate records, requiring communication to bring identical items to the same worker.
  - `groupByKey`: Groups records by a key, bringing together all links from a given page.
- The data is shuffled so that items with the same key end up on the same worker.
- Data that will be used repeatedly should be cached or persisted to avoid recomputing it from scratch.
  - Modern Spark uses the `cache` function for this purpose.
  - The `persist` call provides more general control over where the data is stored such as saving it to HDFS.

## Execution Details

- When `collect` is called, Spark figures out where the data is on HDFS, picks a set of workers, compiles transformations into bytecodes, sends them to workers, and then fetches the data back.
- The execution of `map` can happen in parallel across workers since it operates independently on each line.
- Transformations like `distinct` require communication and data shuffling between workers.
- Spark uses hashing or sorting to distribute data for operations that require shuffling.
- The system is designed so that workers perform as many narrow stages as possible before moving on to wide transformations.
- Narrow dependencies are transformations that operate on each record independently and are local to the worker.
- Wide dependencies involve transformations that require data from all partitions, which includes shuffles, and require data movement over the network.

## Iteration and Optimization

- In iterative processes, such as PageRank, each iteration is handled by repeatedly using the same links data and updating page ranks. Spark doesn't actually change the values but creates new transformations.
- The lineage graph represents the sequence of processing stages, including repeated joins and reductions.
- Spark optimizes performance by leaving data in memory between transformations and by optimizing shuffles for wide transformations.
- Spark can optimize execution by avoiding shuffles when data is already partitioned correctly for the next operation.

## Fault Tolerance

- Spark assumes that the input data on HDFS is fault-tolerant, with multiple copies stored on different servers.
- If a worker fails, Spark recomputes the lost partitions on another worker.
- For narrow dependencies, this process is relatively simple, just requiring the recomputation of a partition.
- For wide dependencies, if a worker fails, all the workers may have discarded the intermediate results from prior transformations which may require re-executing those transformations.
- Checkpoints can be set to save intermediate results to HDFS so that they don't have to be recomputed from scratch in case of failure.
  - Checkpointing can be done periodically in long iterative processes.
  - The `persist` call can be used to save the output of transformations to HDFS.
  - `cache` may also save data in memory, but it is not a guarantee that data will be available in the event of a failure.
- The deterministic nature of transformations and the immutability of RDDs allows Spark to recover from failure by recomputing partitions.

## Limitations

- Spark is designed for batch processing of large datasets and is not suitable for transaction processing or online applications.
- The original Spark described in the paper is not optimized for stream processing, but Spark Streaming is an extension of Spark geared to processing data as it arrives in a stream.

