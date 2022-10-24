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

# Design A Key-value Store

## Data partition

- There are two challenges while partitioning the data

  - Distribute data across multiple servers evenly
  - Minimize data movement when nodes are added or removed

- Using consistent hashing to partition data has the following advantage
  - Automatic scaling
  - Heterogeneity

## Data replication

- To achieve high availability and reliability, data must be replicated asynchronously over N servers

## Consistency models

- Strong consistency
- Weak consistenc
- Eventual consistency

## Inconsistency resolution: versioning

- Versioning and vector locks are used to solve inconsistency problems. Versioning means treating each data modification as a new immutable version of data

- Highly available writes
  - Versioning and conflict resolution with vector clocks

## Handling failures

- Failure detection
  - all-to-all multicasting is a straightforward solution. However, this is inefficient when many servers are in the system
  - A better solution is to use decentralized failure detection methods like gossip protocol
    - Each node maintains a node membership list, which contains member IDs and heartbeat counters
    - Each node periodically increments its heartbeat counter
    - Each node periodically sends heartbeats to a set of random nodes, which in turn propagate to another set of nodes
    - Once nodes receive heartbeats, membership list is updated to the latest info

## System architecture diagram

- Clients communicate with the key-value store through simple APIs: get(key) and put(key, value)
- A coordinator is a node that acts as a proxy between the client and the key-value store
- Nodes are distributed on a ring using consistent hashing
- The system is completely decentralized so adding and moving nodes can be automatic
- Data is replicated at multiple nodes
- There is no single point of failure as every node has the same set of responsibilities

## Write path

- The write request is persisted on a commit log file
- Data is saved in the memory cache
- When the memory cache is full or reaches a predefined threshold, data is flushed to SSTable on disk

## Read path

- If the data is not in memory, it will be retrieved from the disk instead. We need an efficient way to find out which SSTable contains the key. Bloom filter is commonly used to solve this problem
- 1. The system first checks if data is in memory. If not, go to step 2
- 2. If data is not in memory, the system checks the bloom filter
- 3. If data is not in memory, the system checks the bloom filter.
- 4. SSTables return the result of the data set
- 5. The result of the data set is returned to the client

# Design A Unique ID Generator In Distributed Systems

- Multiple options can be used to generate unique IDs in distributed systems
  - Multi-master replication
  - Universally unique identifier (UUID)
  - Ticket server
  - Twitter snowflake approach

# Design A URL Shortener

- API design
- data model
- hash function
- URL shortening
- URL redirecting

# Design A Web Crawler

- DFS vs BFS

  - DFS is usually not a good choice because the depth of DFS can be very deep
  - BFS is commonly used by web crawlers and is implemented by a first-in-first-out (FIFO) queue

- Robustness
  - Consistent hashing
  - Save crawl states and data
  - Exception handling
  - Data validation

# Design A Notification System

- high-level design
  - Reliability
  - Security
  - Tracking and monitoring
  - Respect user settings
  - Rate limiting

# Design A News Feed System

## Fanout service

- Fanout on write
  - With this approach, news feed is pre-computed during write time. A new post is delivered to friends’ cache immediately after it is published
  - Pros
    - The news feed is generated in real-time and can be pushed to friends immediately
    - Fetching news feed is fast because the news feed is pre-computed during write time
  - Cons
    - If a user has many friends, fetching the friend list and generating news feeds for all of them are slow and time consuming. It is called hotkey problem
    - For inactive users or those rarely log in, pre-computing news feeds waste computing resources
- Fanout on read
  - The news feed is generated during read time. This is an on-demand model. Recent posts are pulled when a user loads her home page
  - Pros
    - For inactive users or those who rarely log in, fanout on read works better because it will not waste computing resources on them
    - Data is not pushed to friends so there is no hotkey problem
  - Cons
    - Fetching the news feed is slow as the news feed is not pre-compute

## Cache architecture

- Cache is extremely important for a news feed system. We divide the cache tier into 5 layers

  - News Feed: It stores IDs of news feeds
  - Content: It stores every post data. Popular content is stored in hot cache
  - Social Graph: It stores user relationship data
  - Action: It stores info about whether a user liked a post, replied a post, or took other actions on a post
  - Counters: It stores counters for like, reply, follower, following, etc

  # Design A Chat System

  - Storage

    - Selecting the correct storage system that supports all of our use cases is crucial. We recommend key-value stores for the following reasons
      - Key-value stores allow easy horizontal scaling
      - Key-value stores provide very low latency to access data
      - Relational databases do not handle long tail of data well. When the indexes grow large, random access is expensive
      - Key-value stores are adopted by other proven reliable chat applications

    # Design A Search Autocomplete System

    - Trie data structure
      - Fetching the top 5 search queries from a relational database is inefficient. The data structure trie (prefix tree) is used to overcome the problem.
      - A trie is a tree-like data structure
      - The root represents an empty string
      - Each node stores a character and has 26 children, one for each possible character
      - Each tree node represents a single word or a prefix string
    - How does autocomplete work with trie? Before diving into the algorithm, let us define some terms
      - p: length of a prefix
      - n: total number of nodes in a trie
      - c: number of children of a given node
    - Steps to get top k most searched queries
      1. Find the prefix. Time complexity: O(p)
      2. Traverse the subtree from the prefix node to get all valid children. A child is valid if it can form a valid query string. Time complexity: O(c)
      3. Sort the children and get top k. Time complexity: O(clogc)
    - It is too slow because we need to traverse the entire trie to get top k results in the worst-case scenario. Below are two optimizations

    1. Limit the max length of a prefix
    2. Cache top search queries at each node

  ## Data gathering service

  - Aggregated Data
    - Workers
      - Workers are a set of servers that perform asynchronous jobs at regular intervals. They build the trie data structure and store it in Trie DB
    - Trie Cache
      - rie Cache is a distributed cache system that keeps trie in memory for fast read. It takes a weekly snapshot of the DB
    - Trie DB
      - Trie DB is the persistent storage
        - Document store: Since a new trie is built weekly, we can periodically take a snapshot of it, serialize it, and store the serialized data in the database. Document stores like MongoDB are good fits for serialized data
        - Key-value store: A trie can be represented in a hash table form by applying the following logic
          - Every prefix in the trie is mapped to a key in a hash table
          - Data on each trie node is mapped to a value in a hash table

  ## Query service

  - Query service requires lightning-fast speed. We propose the following optimizations
    - AJAX request
      - The main benefit of AJAX is that sending/receiving a request/response does not refresh the whole web page
    - Browser caching
    - Data sampling
      - For a large-scale system, logging every search query requires a lot of processing power and storage. Data sampling is important. For instance, only 1 out of every N requests is logged by the system

  ## Trie operations

  - Create
    - Trie is created by workers using aggregated data. The source of data is from Analytics Log/DB
  - Update
    1. Update the trie weekly. Once a new trie is created, the new trie replaces the old one
    2. Update individual trie node directly. We try to avoid this operation because it is slow. However, if the size of the trie is small, it is an acceptable solution. When we update a trie node, its ancestors all the way up to the root must be updated because ancestors store top queries of children
  - Delete

    - We have to remove hateful, violent, sexually explicit, or dangerous autocomplete suggestions. We add a filter layer (Figure 14) in front of the Trie Cache to filter out unwanted suggestions. Having a filter layer gives us the flexibility of removing results based on different filter rules. Unwanted suggestions are removed physically from the database asynchronically so the correct data set will be used to build trie in the next update cycle

  - Building a real-time search autocomplete is complicated and is beyond the scope of this course so we will only give a few ideas
    - Reduce the working data set by sharding
    - Change the ranking model and assign more weight to recent search queries

# Design YouTube

## Video streaming flow

- streaming protocol

  - MPEG–DASH. MPEG stands for **_Moving Picture Experts Group_** and DASH stands for **_Dynamic Adaptive Streaming over HTTP_**

  - Apple HLS. HLS stands for **_HTTP Live Streaming_**
  - Microsoft Smooth Streaming
  - Adobe HTTP Dynamic Streaming (HDS)

## Video transcoding

- Video transcoding is important for the following reasons
  - Raw video consumes large amounts of storage space. An hour-long high definition video recorded at 60 frames per second can take up a few hundred GB of space
  - Many devices and browsers only support certain types of video formats. Thus, it is important to encode a video to different formats for compatibility reasons
  - To ensure users watch high-quality videos while maintaining smooth playback, it is a good idea to deliver higher resolution video to users who have high network bandwidth and lower resolution video to users who have low bandwidth
  - Network conditions can change, especially on mobile devices. To ensure a video is played continuously, switching video quality automatically or manually based on network conditions is essential for smooth user experience
- Many types of encoding formats are available; however, most of them contain two parts
  - Container
    - This is like a basket that contains the video file, audio, and metadata. You can tell the container format by the file extension, such as .avi, .mov, or .mp4
  - Codecs
    - These are compression and decompression algorithms aim to reduce the video size while preserving the video quality. The most used video codecs are H.264, VP9, and HEVC

## Directed acyclic graph (DAG) model

- To support different video processing pipelines and maintain high parallelism, it is important to add some level of abstraction and let client programmers define what tasks to execute. For example, Facebook’s streaming video engine uses a directed acyclic graph (DAG) programming model, which defines tasks in stages so they can be executed sequentially or parallelly

## Video transcoding architecture

- Preprocessor
  - The preprocessor has 4 responsibilities
  1. Video splitting. Video stream is split or further split into smaller Group of Pictures (GOP) alignment. GOP is a group/chunk of frames arranged in a specific order. Each chunk is an independently playable unit, usually a few seconds in length
  2. Some old mobile devices or browsers might not support video splitting. Preprocessor split videos by GOP alignment for old clients
  3. DAG generation
  4. Cache data
- DAG scheduler

  - The DAG scheduler splits a DAG graph into stages of tasks and puts them in the task queue in the resource manager

- Resource manager
  - The resource manager is responsible for managing the efficiency of resource allocation. It contains 3 queues and a task scheduler
    - Task queue: It is a priority queue that contains tasks to be executed
    - Worker queue: It is a priority queue that contains worker utilization info
    - Running queue: It contains info about the currently running tasks and workers running the tasks
  - Task scheduler: It picks the optimal task/worker, and instructs the chosen task worker to execute the job
  - The resource manager works as follows
    - The task scheduler gets the highest priority task from the task queue
    - The task scheduler gets the optimal task worker to run the task from the worker queue
    - The task scheduler instructs the chosen task worker to run the task
    - The task scheduler binds the task/worker info and puts it in the running queue
    - The task scheduler removes the job from the running queue once the job is done
- Task workers
  - Task workers run the tasks which are defined in the DAG
- Temporary storage
- Encoded video
  - Encoded video is the final output of the encoding pipeline

## System optimizations

- Speed optimization

  - parallelize video uploading
    - The job of splitting a video file by GOP can be implemented by the client to improve the upload speed
  - place upload centers close to users
  - parallelism everywhere
    - Before the message queue is introduced, the encoding module must wait for the output of the download module
    - After the message queue is introduced, the encoding module does not need to wait for the output of the download module anymore. If there are events in the message queue, the encoding module can execute those jobs in parallel

- Safety optimization
  - pre-signed upload URL
    - The client makes a HTTP request to API servers to fetch the pre-signed URL, which gives the access permission to the object identified in the URL. The term pre-signed URL is used by uploading files to Amazon S3. Other cloud service providers might use a different name. For instance, Microsoft Azure blob storage supports the same feature, but call it **_Shared Access Signature_**
    - API servers respond with a pre-signed URL
    - Once the client receives the response, it uploads the video using the pre-signed URL
  - protect your videos
    - Digital rights management (DRM) systems
    - AES encryption
    - Visual watermarking
- Cost-saving optimization
  - Only serve the most popular videos from CDN and other videos from our high capacity storage video servers
  - For less popular content, we may not need to store many encoded video versions. Short videos can be encoded on-demand
  - Some videos are popular only in certain regions. There is no need to distribute these videos to other regions
  - Build your own CDN like Netflix and partner with Internet Service Providers (ISPs). Building your CDN is a giant project; however, this could make sense for large streaming companies. An ISP can be Comcast, AT&T, Verizon, or other internet providers. ISPs are located all around the world and are close to users. By partnering with ISPs, you can improve the viewing experience and reduce the bandwidth charges

## Error handling

- To build a highly fault-tolerant system, we must handle errors gracefully and recover from them fast. Two types of errors exist
  - Recoverable error. For recoverable errors such as video segment fails to transcode, the general idea is to retry the operation a few times. If the task continues to fail and the system believes it is not recoverable, it returns a proper error code to the client
  - Non-recoverable error. For non-recoverable errors such as malformed video format, the system stops the running tasks associated with the video and returns the proper error code to the client

# Design Google Drive

## Save storage space

- Storage space can be filled up quickly with frequent backups of all file revisions. Three techniques are proposed to reduce storage costs
  - De-duplicate data blocks. Eliminating redundant blocks at the account level is an easy way to save space. Two blocks are identical if they have the same hash value
  - Adopt an intelligent data backup strategy. Two optimization strategies can be applied
    - Set a limit
    - Keep valuable versions only
    - Moving infrequently used data to cold storage

# Proximity Service

- The system comprises two parts
  - Location-based service (LBS)
    - The LBS service is the core part of the system which finds nearby businesses for a given radius and location
      - It is a read-heavy service with no write requests
      - QPS is high, especially during peak hours in dense areas
      - This service is stateless so it’s easy to scale horizontally
  - Business service
    - Business owners create, update, or delete businesses. Those requests are mainly write operations, and the QPS is not high
    - Customers view detailed information about a business. QPS is high during peak hours

## Algorithms to fetch nearby businesses

- Two-dimensional search
  - Hash: even grid, geohash, cartesian tiers , etc.
  - Tree: quadtree, Google S2, RTree , etc
  - geohash, quadtree, and Google S2 are most widely used in real-world applications
- Evenly divided grid
- Geohash
  - Geohash is better than the evenly divided grid option. It works by reducing the two-dimensional longitude and latitude data into a one-dimensional string of letters and digits. Geohash algorithms work by recursively dividing the world into smaller and smaller grids with each additional bit
- Quadtree
  - A quadtree is a data structure that is commonly used to partition a two-dimensional space by recursively subdividing it into four quadrants (grids) until the contents of the grids meet certain criteria
  - quadtree is an in-memory data structure and it is not a database solution.
- Google S2

  - it is an in-memory solution. It maps a sphere to a 1D index based on the Hilbert curve (a space-filling curve)
  - S2 is great for geofencing because it can cover arbitrary areas with varying levels (Figure 17). According to Wikipedia, **_A geofence is a virtual perimeter for a real-world geographic area. A geo-fence could be dynamically generated—as in a radius around a point location, or a geo-fence can be a predefined set of boundaries (such as school zones or neighborhood boundaries)_**
  - Another advantage of S2 is its Region Cover algorithm . Instead of having a fixed level (precision) as in geohash, we can specify min level, max level, and max cells in S2

- Geohash vs quadtree
  - Geohash
    - Easy to use and implement. No need to build a tree
    - Supports returning businesses within a specified radius
    - When the precision (level) of geohash is fixed, the size of the grid is fixed as well. It cannot dynamically adjust the grid size, based on population density. More complex logic is needed to support this
    - Updating the index is easy. For example, to remove a business from the index, we just need to remove it from the corresponding row with the same geohash and business_id
  - Quadtree
    - Slightly harder to implement because it needs to build the tree
    - Supports fetching k-nearest businesses
    - Updating the index is more complicated than geohash

## Scale the database

- Business table
  - The easiest approach is to shard everything by business ID
- Geospatial index table
  - Option 1: For each geohash key, there is a JSON array of business IDs in a single row This means all business IDs within a geohash are stored in one row
  - Option 2: If there are multiple businesses in the same geohash, there will be multiple rows, one for each business. This means different business IDs within a geohash are stored in different rows
  - Recommendation: Option2
    - if we have two columns with a compound key of (geohash, business_id), the addition and removal of a business are very simple. There would be no need to lock anything

# Nearby Friends

- WebSocket: real-time communication between clients and the server
- Redis: fast read and write of location data
- Redis pub/sub: routing layer to direct location updates from one user to all the online friends
- Alternative to Redis pub/sub
  - Erlang

# Google Maps

- Location service
- Navigation service
- Map rendering

# Distributed Message Queue

- What benefits do message queues bring

  - Decoupling
    - Message queues eliminate the tight coupling between components so they can be updated independently.
  - Improved scalability
    - We can scale producers and consumers independently based on traffic load. For example, during peak hours, more consumers can be added to handle the increased traffic
  - Increased availability
    - If one part of the system goes offline, the other components can continue to interact with the queue
  - Better performance
    - Message queues make asynchronous communication easy. Producers can add messages to a queue without waiting for the response and consumers consume messages whenever they are available. They don’t need to wait for each other
  - Message queues vs event streaming platforms

- the basic functionalities of a message queue
  - Producer sends messages to a message queue
  - Consumer subscribes to a queue and consumes the subscribed messages
  - Message queue is a service in the middle that decouples the producers from the consumers, allowing each of them to operate and scale independently
  - Both producer and consumer are clients in the client/server model, while the message queue is the server. The clients and servers communicate over the network

# Metrics Monitoring and Alerting System

- A metrics monitoring and alerting system generally contains five components

  - Data collection
    - collect metric data from different sources
  - Data transmission
    - transfer data from sources to the metrics monitoring system
  - Data storage
    - organize and store incoming data
  - Alerting
    - analyze incoming data, detect anomalies, and generate alerts. The system must be able to send alerts to different communication channels
  - Visualization
    - present data in graphs, charts, etc. Engineers are better at identifying patterns, trends, or problems when data is presented visually, so we need visualization functionality

- According to DB-engines , the two most popular time-series databases are **_InfluxDB_** and **_Prometheus_**, which are designed to store large volumes of time-series data and quickly perform real-time analysis on that data. Both of them primarily rely on an in-memory cache and on-disk storage

## Metrics collection

- Pull vs push models

# Ad Click Event Aggregation

## Aggregation service

- The MapReduce framework is a good option to aggregate ad click events.The directed acyclic graph (DAG) is a good model for it
- The DAG model represents the well-known MapReduce paradigm. It is designed to take big data and use parallel distributed computing to turn big data into little- or regular-sized data

## Kappa architecture V.S lambda architecture

- A disadvantage of lambda architecture is that you have two processing paths, meaning there are two codebases to maintain
- Kappa architecture, which combines the batch and streaming in one processing path, solves the problem. The key idea is to handle both real-time data processing and continuous data reprocessing using a single stream processing engine

# Hotel Reservation System

- Race conditions issue

  - Pessimistic locking
    - prevents simultaneous updates by placing a lock on a record as soon as one user starts to update it. Other users who attempt to update the record have to wait until the first user has released the lock
    - Pros
      - Prevents applications from updating data that is being – or has been – changed
      - It is easy to implement and it avoids conflict by serializing updates. Pessimistic locking is useful when data contention is heavy
    - Cons
      - Deadlocks may occur when multiple resources are locked. Writing deadlock-free application code could be challenging
      - This approach is not scalable. If a transaction is locked for too long, other transactions cannot access the resource. This has a significant impact on database performance, especially when transactions are long-lived or involve a lot of entities
  - Optimistic locking

    - allows multiple concurrent users to attempt to update the same resource
    - Optimistic locking is usually faster than pessimistic locking because we do not lock the database. However, the performance of optimistic locking drops dramatically when concurrency is high
    - Pros

      - It prevents applications from editing stale data
      - We don’t need to lock the database resource. There's actually no locking from the database point of view. It's entirely up to the application to handle the logic with the version number
      - Optimistic locking is generally used when the data contention is low. When conflicts are rare, transactions can complete without the expense of managing locks

    - Cons
      - Performance is poor when data contention is heavy

  - Database constraints
    - Pros
      - Easy to implement
      - It works well when data contention is minimal
    - Cons
      - Similar to optimistic locking, when data contention is heavy, it can result in a high volume of failures. Users could see there are rooms available, but when they try to book one, they get the “no rooms available” response. This experience can be frustrating to users
      - The database constraints cannot be version-controlled easily like the application code
      - Not all databases support constraints. It might cause problems when we migrate from one database solution to another

## Scalability

- Database sharding
- Caching
  - Pros
    - Reduced database load. Since read queries are answered by the cache layer, database load is significantly reduced
    - High performance. Read queries are very fast because results are fetched from memory
  - Cons
    - Maintaining data consistency between the database and cache is hard. We need to think carefully about how this inconsistency affects user experience

## Data consistency among services

- To address the data inconsistency, here is a high-level summary of industry-proven techniques
  - Two-phase commit (2PC) . 2PC is a database protocol used to guarantee atomic transaction commit across multiple nodes, i.e., either all nodes succeeded or all nodes failed. Because 2PC is a blocking protocol, a single node failure blocks the progress until the node has recovered. It’s not performant
  - Saga. A saga is a sequence of local transactions. Each transaction updates and publishes a message to trigger the next transaction step. If a step fails, the saga executes compensating transactions to undo the changes that were made by preceding transactions. 2PC works as a single commit to perform ACID transactions while Saga consists of multiple steps and relies on eventual consistency

# Distributed Email Service

- Email protocols
  - SMTP

## Choosing the right database

- A single column can be a single-digit of MB
- Strong data consistency
- Designed to reduce disk I/O
- It should be highly available and fault-tolerant
- It should be easy to create incremental backups

# S3-like Object Storage

- Uploading an object
- Downloading an object
- Object versioning and listing objects in a bucket

# Real-time Gaming Leaderboard

## Redis solution

- redis has a specific data type called sorted sets that are ideal for solving leaderboard system design problems
  - a sorted set is implemented by two data structures
    - hash table
    - skip list

### Implementation using Redis sorted sets

- ZADD: insert the user into the set if they don’t yet exist. Otherwise, update the score for the user. It takes O(log(n)) to execute
- ZINCRBY: increment the score of the user by the specified increment. If the user doesn’t exist in the set, then it assumes the score starts at 0. It takes O(log(n)) to execute
- ZRANGE/ZREVRANGE: fetch a range of users sorted by the score. We can specify the order (range vs. revrange), the number of entries, and the position to start from. This takes O(log(n)+m) to execute, where m is the number of entries to fetch (which is usually small in our case), and n is the number of entries in the sorted set
- ZRANK/ZREVRANK: Fetch the position of any user sorting in ascending/descending order in logarithmic time

## Alternative solution: NoSQL

- Ideally, we want to choose a NoSQL that has the following properties
  - Optimized for writes
  - Efficiently sort items within the same partition by score

# Payment System

# Digital Wallet

## Distributed transaction: two-phase commit

## Distributed transaction: Try-Confirm/Cancel

- TC/C is a type of compensating transaction that has two steps

  1. In the first phase, the coordinator asks all databases to reserve resources for the transaction
  2. In the second phase, the coordinator collects replies from all databases

     - If all databases reply with "yes", the coordinator asks all databases to confirm the operation, which is the Try-Confirm process.

     - If any database replies with "no", the coordinator asks all databases to cancel the operation, which is the Try-Cancel process

## Distributed transaction: Saga

- Saga is the de-facto standard in a microservice architecture. The idea of Saga is simple:
  - All operations are ordered in a sequence. Each operation is an independent transaction on its own database
  - Operations are executed from the first to the last. When one operation has finished, the next operation is triggered
