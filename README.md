# Scale From Zero To Millions Of Users

## Database

- Relational databases represent and store data in tables and rows MySQL, Oracle database, PostgreSQL, etc.

- NoSQL databases. Popular ones are CouchDB, Neo4j, Cassandra, HBase, Amazon DynamoDB, etc. These databases are grouped into four categories: key-value stores, graph stores, column stores, and document stores.(Join opertations are generally not supported in non-relational databases.)

- Non-relational databases might be the right choice if:

  - Your application requires super-low latency.

  - Your data are unstructured, or you do not have any relational data.

  - You only need to serialize and deserialize data (JSON, XML, YAML, etc.).

  - You need to store a massive amount of data.

## Vertical scaling vs horizontal scaling

- Vertical scaling, referred to as **_scale up_** adding more power (CPU, RAM, etc.) to your servers

- Horizontal scaling, referred to as **_scale-out_** adding more servers into your pool of resources.

- vertical scaling comes with serious limitations
  - Vertical scaling has a hard limit. It is impossible to add unlimited CPU and memory to a single server.
  - Vertical scaling does not have failover and redundancy. If one server goes down, the website/app goes down with it completely.

## Load balancer

- The load balancer communicates with web servers through private IPs.

## Database replication

- Advantages of database replication
  - Better performance
    - In the master-slave model, all writes and updates happen in master nodes; whereas, read operations are distributed across slave nodes. This model improves performance because it allows more queries to be processed in parallel.
  - Reliability
    - If one of your database servers is destroyed by a natural disaster, such as a typhoon or an earthquake, data is still preserved. You do not need to worry about data loss because data is replicated across multiple locations.
  - High availability
    - By replicating data across different locations, your website remains in operation even if a database is offline as you can access data stored in another database server.

## Cache

- A cache is a temporary storage area that stores the result of expensive responses or frequently accessed data **_in memory_** so that subsequent requests are served more quickly.

## Stateless web tier

- Each web server in the cluster can access state data from databases

### Stateful architecture

- A stateful server remembers client data (state) from one request to the next. A stateless server keeps no state information.

- In this stateless architecture, HTTP requests from users can be sent to any web servers, which fetch state data from a shared data store. State data is stored in a shared data store and kept out of web servers.

## Data centers

- Several technical challenges must be resolved to achieve multi-data center setup
  - Traffic redirection
  - Data synchronization
  - Test and deployment

## Message queue

- A message queue is a durable component, stored in memory, that supports asynchronous communication.

## Database scaling

- vertical scaling comes with some serious drawbacks
  - You can add more CPU, RAM, etc. to your database server, but there are hardware limits. If you have a large user base, a single server is not enough.
  - Greater risk of single point of failures.
  - The overall cost of vertical scaling is high. Powerful servers are much more expensive.

## Summary

- Keep web tier stateless
- Build redundancy at every tier
- Cache data as much as you can
- Support multiple data centers
- Host static assets in CDN
- Scale your data tier by sharding
- Split tiers into individual services
- Monitor your system and use automation tools

# Back-of-the-envelope Estimation

- Memory is fast but the disk is slow.
- Avoid disk seeks if possible.
- Simple compression algorithms are fast.
- Compress data before sending it over the internet if possible.
- Data centers are usually in different regions, and it takes time to send data between them.

## Back-of-the-envelope estimation is all about the process. Solving the problem is more important than obtaining results.

- Rounding and Approximation
- Write down your assumptions
- Label your units.

# A Framework For System Design Interviews

- A 4-step process for effective system design interview

  - Step 1 - Understand the problem and establish design scope
  - Step 2 - Propose high-level design and get buy-in
  - Step 3 - Design deep dive
  - Step 4 - Wrap up

  # Design A Rate Limiter

  ## Algorithms for rate limiting

  - Token bucket
    - A token bucket is a container that has pre-defined capacity. Tokens are put in the bucket at preset rates periodically. Once the bucket is full, no more tokens are added.
    - The token bucket algorithm takes two parameters
      - Bucket size: the maximum number of tokens allowed in the bucket
      - Refill rate: number of tokens put into the bucket every second
      - Pros
        - The algorithm is easy to implement
        - Memory efficient
        - Token bucket allows a burst of traffic for short periods. A request can go through as long as there are tokens left.
      - Cons
        - Two parameters in the algorithm are bucket size and token refill rate. However, it might be challenging to tune them properly.
  - Leaking bucket

    - Leaking bucket algorithm takes the following two parameters
      - Bucket size: it is equal to the queue size. The queue holds the requests to be processed at a fixed rate.
      - Outflow rate: it defines how many requests can be processed at a fixed rate, usually in seconds.
      - Pros
        - Memory efficient given the limited queue size
        - Requests are processed at a fixed rate therefore it is suitable for use cases that a stable outflow rate is needed
      - Cons
        - A burst of traffic fills up the queue with old requests, and if they are not processed in time, recent requests will be rate limited
      - There are two parameters in the algorithm. It might not be easy to tune them properly

  - Fixed window counter
    - The algorithm divides the timeline into fix-sized time windows and assign a counter for each window
    - Each request increments the counter by one
    - Once the counter reaches the pre-defined threshold, new requests are dropped until a new time window starts
    - Pros
      - Memory efficient
      - Easy to understand
      - Resetting available quota at the end of a unit time window fits certain use cases
    - Cons
      - pike in traffic at the edges of a window could cause more requests than the allowed quota to go through
  - Sliding window log

    - the fixed window counter algorithm has a major issue: it allows more requests to go through at the edges of a window. The sliding window log algorithm fixes the issue
    - Pros
      - Rate limiting implemented by this algorithm is very accurate. In any rolling window, requests will not exceed the rate limit
    - Cons
      - The algorithm consumes a lot of memory because even if a request is rejected, its timestamp might still be stored in memory.

  - Sliding window counter
    - Pros
      - It smooths out spikes in traffic because the rate is based on the average rate of the previous window
      - Memory efficient
    - Cons
      - It only works for not-so-strict look back window. It is an approximation of the actual rate because it assumes requests in the previous window are evenly distributed.

## Rate limiter in a distributed environment

- Race condition
  - Locks are the most obvious solution for solving race condition. However, locks will significantly slow down the system. Two strategies are commonly used to solve the problem: Lua script and sorted sets data structure in Redis
- Synchronization issue
  - When multiple rate limiter servers are used, synchronization is required. For example, on the left side of Figure 15, client 1 sends requests to rate limiter 1, and client 2 sends requests to rate limiter 2. As the web tier is stateless, clients can send requests to a different rate limiter as shown on the right side of Figure 15. If no synchronization happens, rate limiter 1 does not contain any data about client 2. Thus, the rate limiter cannot work properly
  - One possible solution is to use sticky sessions that allow a client to send traffic to the same rate limiter. This solution is not advisable because it is neither scalable nor flexible. A better approach is to use centralized data stores like Redis.

# Design Consistent Hashing

- Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on average, where k is the number of keys, and n is the number of slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be remapped

- Map servers and keys on to the ring using a uniformly distributed hash function
- To find out which server a key is mapped to, go clockwise from the key position until the first server on the ring is found.

- The benefits of consistent hashing include
  - Minimized keys are redistributed when servers are added or removed
  - It is easy to scale horizontally because data are more evenly distributed
  - Mitigate hotspot key problem. Excessive access to a specific shard could cause server overload
