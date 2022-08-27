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
