# Lecture 16: Cache Consistency: Memcached at Facebook

## Introduction
- This paper discusses Facebook's use of Memcached to manage large-scale data efficiently.
- It highlights the challenges of scaling and the trade-offs between performance and consistency.
- The paper illustrates the fundamental tension between achieving high performance and maintaining consistency, especially with replication.

## Early Website Architecture
- Websites typically start small, focusing on features rather than high-performance infrastructure.
- Facebook used PHP and MySQL as the foundation for its early architecture.
  - MySQL was chosen for its SQL query capabilities, power, and ACID compliance.
- As traffic increased, PHP scripts began consuming too much CPU, creating performance bottlenecks.

## Scaling the Architecture
- **Architecture 2**: To handle more load, multiple front-end servers were added, all accessing a single MySQL database server.
  - A single database server is simpler than multiple servers, avoiding complications like distributed transactions.
- **Architecture 3**: As the database server struggled, data was sharded across multiple servers to distribute the load.
  - Sharding splits data into chunks, increasing capacity but also introducing complexities in database access and software changes.
- Database servers like MySQL are not fast enough for read-heavy applications like websites, requiring additional solutions.

## Introduction of Caching with Memcached
- Rather than adding more database servers, Facebook introduced Memcached to act as a cache layer.
  - Memcached is significantly faster for read operations than MySQL.
- **Architecture 4**: Facebook introduced front-end servers, database servers, and a Memcached cache layer.
  - Front-end servers first query Memcached; if a miss occurs, they fetch data from the database and store it in the cache.
- Writes are still sent to the database, ensuring durability, while reads are served by the cache for improved performance.
- Memcached operates as a lookaside cache, which means it is independent of the database and unaware of its contents.
  - The cache does not know the database structure and only stores the processed data.

## Facebook's Specific Design
- Facebook prioritizes presenting data quickly to users, and data freshness isn't always critical.
  - Stale data a few seconds old is acceptable, but stale data older than that is avoided.
  - Consistency is important when users update their own data.
- Facebook operates in multiple data centers for redundancy and better performance:
  - The primary data center is on the West Coast, and a secondary one is on the East Coast.
  - Each data center has a full replica of the data.
  - Writes are sent to the primary data center and asynchronously replicated to the secondary center.
  - Reads are served locally from the closest data center, which improves speed.

## Caching Details
- **Reads**: Front-end servers first check Memcached for cached data.
  - If a cache hit occurs, data is returned from Memcached; if not, it queries the database and caches the result.
- **Writes**: New data is sent to the database and simultaneously a delete command is sent to Memcached to invalidate outdated cache entries.
  - The cache invalidation ensures that outdated data isn’t returned on subsequent requests.

## Performance Optimization Techniques
- **Parallelization**: Facebook achieves high performance through partitioning (sharding) and replication.
  - **Partitioning**: Data is divided into segments (partitions) across multiple servers to optimize memory usage, though it doesn't address hot keys effectively.
  - Front-end servers may need to query multiple partitions for data.
  - **Replication**: Data is duplicated across servers for popular keys, but caching becomes less effective due to the increased data volume.
- Memcached combines both partitioning and replication to optimize its performance.
  - Data is split using hashing, where each key is mapped to a specific Memcached server.
- **Regions**: Facebook uses regional data centers to replicate data for redundancy and minimize latency.
  - Each region has a complete replica of the data, and users are served by the nearest data center.
  - Front-end servers rely on local Memcached servers for faster access to cached data.
- **Clusters**: Within each region, Memcached servers are grouped into clusters.
  - Each cluster has its own set of front-end servers and Memcached instances.
  - This reduces "n-squared" communication overhead between front-end servers and Memcached.
- **Cold Start**: New clusters enter "cold start" mode where front-end servers query warmer clusters to avoid overwhelming the database.
  - This phase lasts a couple of hours, after which the cold start mode is disabled.
- **Thundering Herd**: When a popular item is deleted from the cache, multiple front-end servers try to retrieve the data at the same time.
  - Facebook mitigates this with a lease mechanism that grants only one front-end server the permission to fetch and refresh the cache. Other servers must wait for the lease to expire.
- **Gutter Servers**: In case of a Memcached server failure, gutter servers temporarily take over and reduce database load.
  - Front-end servers use the hashed key to determine which gutter server to interact with.
  - Gutter servers delete outdated cache keys rapidly, without waiting for explicit deletes.

## Consistency Challenges and Solutions
- The system involves many copies of data, leading to concurrency issues and stale data.
  - Multiple front-end servers may cause conflicts or data inconsistency.
- **Update Races**: Slow reads can result in outdated data being written back to Memcached, causing stale data to persist.
- **Lease Mechanism**: The lease mechanism, also applied to the Thundering Herd problem, is extended to solve stale data issues:
  - When a cache miss occurs, a lease is granted to the front-end requesting the data.
  - If a delete occurs, the lease is invalidated.
  - A new "set" operation with an invalid lease is ignored, and the data is not updated in the cache.
  - This ensures that the next read fetches fresh data from the database instead of stale cached data.
