## [Sairyss](https://github.com/Sairyss/system-design-patterns)

- [System Design Patterns](#system-design-patterns)
  - [Distributed systems integration](#distributed-systems-integration)
    - [Synchronous communication](#synchronous-communication)
    - [Asynchronous communication](#asynchronous-communication)
    - [Data serialization](#data-serialization)
    - [Api Gateway](#api-gateway)
  - [Scalability](#scalability)
    - [Performance and availability](#performance-and-availability)
      - [Creating redundancy](#creating-redundancy)
      - [Autoscaling](#autoscaling)
      - [Load balancing](#load-balancing)
        - [Transport (Layer 4) load balancing](#transport-layer-4-load-balancing)
        - [Application (Layer 7) load balancing](#application-layer-7-load-balancing)
    - [Databases](#databases)
      - [Indexing](#indexing)
      - [Replication](#replication)
      - [Partitioning](#partitioning)
      - [Hot, Warm and Cold data](#hot-warm-and-cold-data)
      - [Database Federation](#database-federation)
      - [Denormalization](#denormalization)
      - [Materialized views](#materialized-views)
      - [Multitenancy](#multitenancy)
      - [SQL vs NoSQL](#sql-vs-nosql)
    - [Caching](#caching)
      - [Caching strategies](#caching-strategies)
      - [Distributed caches](#distributed-caches)
      - [CDNs](#cdns)
    - [Coupling](#coupling)
      - [Location coupling](#location-coupling)
        - [Broker pattern](#broker-pattern)
      - [Temporal coupling](#temporal-coupling)
        - [Message queues](#message-queues)
      - [Business data and logic coupling](#business-data-and-logic-coupling)
      - [Shared-nothing architecture](#shared-nothing-architecture)
      - [Selective data replication](#selective-data-replication)
      - [Event-driven architecture](#event-driven-architecture)
        - [Event-driven end-to-end](#event-driven-end-to-end)
        - [Designing around dataflow](#designing-around-dataflow)
  - [Consistency](#consistency)
    - [Different views on data](#different-views-on-data)
    - [Distributed transactions](#distributed-transactions)
      - [Two-phase commit (2PC)](#two-phase-commit-2pc)
      - [Sagas](#sagas)
        - [Orchestration](#orchestration)
        - [Choreography](#choreography)
        - [Process manager](#process-manager)
    - [Workflow Engines](#workflow-engines)
    - [Dual writes problem](#dual-writes-problem)
      - [The Outbox Pattern](#the-outbox-pattern)
      - [Retrying failed steps](#retrying-failed-steps)
    - [Idempotent consumer](#idempotent-consumer)
    - [Change Data Capture](#change-data-capture)
    - [Time inconsistencies](#time-inconsistencies)
      - [Lamport timestamps](#lamport-timestamps)
      - [Vector clocks](#vector-clocks)
    - [Conflict-free replicated data type (CRDT)](#conflict-free-replicated-data-type-crdt)
    - [Consensus algorithms](#consensus-algorithms)
      - [Byzantine generals problem](#byzantine-generals-problem)
      - [Raft](#raft)
      - [Paxos](#paxos)
  - [Other topics/patterns](#other-topicspatterns)
    - [Event Sourcing](#event-sourcing)
      - [CQRS](#cqrs)
      - [Projections](#projections)
    - [Actor Model](#actor-model)
    - [API Federation](#api-federation)
    - [Chaos Engineering](#chaos-engineering)
    - [Distributed tracing](#distributed-tracing)
    - [Bulkhead pattern](#bulkhead-pattern)
    - [Local-first software](#local-first-software)
    - [Consistent Hashing](#consistent-hashing)
      - [Hash ring](#hash-ring)
    - [Peer-to-peer](#peer-to-peer)
      - [Gossip protocol](#gossip-protocol)
      - [Distributed Hash Table (DHT)](#distributed-hash-table-dht)
        - [Kademlia](#kademlia)
        - [Chord](#chord)
    - [OSI Model](#osi-model)
  - [More resources](#more-resources)
    - [Books](#books)
    - [Articles](#articles)
    - [Blogs](#blogs)
    - [Websites](#websites)
    - [Github Repositories](#github-repositories)
    - [Videos](#videos)

## Distributed systems integration

### Synchronous communication

In synchronous communication a client depends on an answer. In this form of communication the client sends a request and waits for a response before continuing.

Examples of a synchronous communication are [Remote Procedure Calls (RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call) over [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), [gRPC](https://en.wikipedia.org/wiki/GRPC), [Apache Thrift](https://en.wikipedia.org/wiki/Apache_Thrift) etc. Also [RabbitMQ supports RPC pattern](https://www.rabbitmq.com/tutorials/tutorial-six-javascript.html).

Synchronous calls can be a good choice for:

- Communication between your client (like web UI / mobile app) and your backend server (HTTP).
- Communication with internal shared/common services, like email service or a service to process some data (RPC or HTTP calls).
- Communication with external third-party services.

Synchronous calls between distributed systems is not always a good idea. It creates coupling, system becomes fragile and slow. For example, in microservices world synchronous communication between microservices can transform them to a distributed monolith. Try to avoid that as much as possible.

### Asynchronous communication

In async communication, client or message sender doesn't wait for a response. It just sends a message which is stored until the receiver takes it (fire and forget).

Examples: message brokers developed on top of [AMQP protocol](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) (like [RabbitMQ](https://www.rabbitmq.com/)) or event streams like [Kafka](https://en.wikipedia.org/wiki/Apache_Kafka).

When changes occur, you need some way to reconcile changes across the different systems. One solution is eventual consistency and event-driven communication based on asynchronous messaging.

Ideally, you should try to minimize the communication between the distributed systems. But it is not always possible.
In microservices world, prefer asynchronous communication since it is better suited for communication between microservices than synchronous. The goal of each microservice is to be autonomous. If one of the microservices is down that shouldn't affect other microservices. Asynchronous communication via messaging can remove dependencies on the availability of other services. This topic is discussed more in details below in [coupling](#coupling) section.

References:

- [Asynchronous message-based communication](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/asynchronous-message-based-communication)
- [RabbitMQ publish/subscribe pattern](https://www.rabbitmq.com/tutorials/tutorial-three-javascript.html)
- [Microservice architecture with RabbitMQ, why?](https://larionov.pro/en/articles/2019/msa-rabbitmq-why/)

### Data serialization

We need to address the issue of multi-language microservices communication. A possible solution is to describe our data models in some language-agnostic way so we can generate the same models for each language. This requires the use of an IDL, or [interface definition language](https://en.wikipedia.org/wiki/Interface_description_language).

One of the most common formats for that lately is [JSON](https://en.wikipedia.org/wiki/JSON). It is simple and human readable. For most systems out there this format is a great choice.

But when your system is growing, there are better choices.

[Protocol buffers](https://en.wikipedia.org/wiki/Protocol_Buffers), [Apache Avro](https://en.wikipedia.org/wiki/Apache_Avro), [Apache Thrift](https://en.wikipedia.org/wiki/Apache_Thrift) provide tools for cross platform and language neutral data formats to serialize data. Those formats are useful for services that communicate over a network or for storing data.

Some pros for using a schema and serializing data include:

- Messages are serialized to binary format which is compact (binary messages occupy about twice less storage than JSON)
- Provides interoperability with other languages (language agnostic).
- Provides forward and backward-compatible schema for your messages/events

References:

- [Why Use gRPC and Thrift for Remote Procedure Calls](https://dzone.com/articles/why-use-grpc-and-thrift-for-remote-procedure-calls?fromrel=true)
- [The best serialization strategy for Event Sourcing](https://blog.softwaremill.com/the-best-serialization-strategy-for-event-sourcing-9321c299632b)

### Api Gateway

Your clients need to communicate to your system somehow. Here is where Api Gateway comes in.

> An API gateway is an API management tool that sits between a client and a collection of backend services. An API gateway acts as a reverse proxy to accept all application programming interface (API) calls, aggregate the various services required to fulfill them, and return the appropriate result.
> <small>Quote from [What does an API gateway do?](https://www.redhat.com/en/topics/api/what-does-an-api-gateway-do#:~:text=An%20API%20gateway%20is%20an,and%20return%20the%20appropriate%20result.)</small>

You can implement your own API Gateway if needed, or you could use existing solutions like [Kong](https://konghq.com/kong/) or [nginx](https://www.nginx.com/).

Api Gateway can also handle:

- [Gateway Routing pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/gateway-routing) can help with routing requests to multiple services using a single endpoint
- [Load balancing](#load-balancing) to evenly distribute load across your nodes
- [Rate limiting](https://github.com/Sairyss/backend-best-practices#rate-limiting) to protect from [DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack) and [Brute Force](https://en.wikipedia.org/wiki/Brute-force_attack) attacks
- [Authentication](https://en.wikipedia.org/wiki/Authentication) and [Authorization](https://en.wikipedia.org/wiki/Authorization) to allow only trusted clients to access your APIs
- [Circuit Breaker](https://microservices.io/patterns/reliability/circuit-breaker.html) to prevent an application from repeatedly trying to execute an operation that's likely to fail
- [Gateway Aggregation pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/gateway-aggregation) can help to Compose data from multiple microservices for your client
- Analytics and monitoring for your API calls
- and much more

References:

- [Pattern: API Gateway / Backends for Frontends](https://microservices.io/patterns/apigateway.html)
- [Building Microservices: Using an API Gateway](https://www.nginx.com/blog/building-microservices-using-an-api-gateway/)
- [My experiences with API gateways](https://mahesh-mahadevan.medium.com/my-experiences-with-api-gateways-8a93ad17c4c4)
- [The API gateway pattern versus the Direct client-to-microservice communication](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern)
- [Gateway Offloading pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/gateway-offloading)

## Scalability

Scalability is the capability to handle the increased workload by repeatedly applying a cost-effective strategy for extending a system’s capacity.

**Vertical scaling** means scaling by adding more power (CPU, RAM, Storage, etc.) to your existing servers.

**Horizontal scaling** means adding more servers into your pool of resources.

Below are discussed some problems with scalability and patterns to solve those problems.

### Performance and availability

Components of distributed systems communicate through a network. Networks are unreliable and slow compared to communication in a single process node. Ensuring good performance and availability in these conditions is an important aspect of a distributed system.

**[Performance](https://en.wikipedia.org/wiki/Computer_performance)** is the amount of useful work accomplished by a computer system at reasonable time.

**[Availability](https://www.techtarget.com/searchdatacenter/definition/high-availability)** is often quantified by uptime (or downtime) as a percentage of time the service is available. For example, around 8 hours of downtime per year may be considered as a 99.9% available system.

Below we will discuss some techniques that can help with system performance and availability.

#### Creating redundancy

[Redundancy](<https://en.wikipedia.org/wiki/Redundancy_(engineering)>) means duplication of critical components like data, nodes and running processes, etc.
Redundancy can improve flexibility, reliability, availability, scalability and performance of your system, failure resistance and recovery. Below we discuss techniques and patterns for creating redundancy.

#### Autoscaling

[Autoscaling](https://en.wikipedia.org/wiki/Autoscaling) is a method used in cloud computing that dynamically adjusts the amount of computational resources in a server based on the load.

References:

- [What is Autoscaling? How Does It Work In the Cloud – Simply Explained](https://www.scaleyourapp.com/what-is-autoscaling-how-does-it-work-in-the-cloud-simply-explained/)
- [Kubernetes Autoscaling: 3 Methods and How to Make Them Great](https://spot.io/resources/kubernetes-autoscaling-3-methods-and-how-to-make-them-great/)

#### Load balancing

High-traffic websites must serve thousands or even millions of concurrent requests to their clients. To scale to these amounts of traffic usually requires adding more servers with multiple instances/nodes of a backend application running.

To distribute traffic between multiple servers and instances/nodes, you will need load balancing tools sitting between the client and the backend server.
[Load balancing](<https://en.wikipedia.org/wiki/Load_balancing_(computing)>) is a technique to distribute network or application traffic across a number of servers.

Load balancers can operate at one of the two levels described below.

##### Transport (Layer 4) load balancing

Layer 4 load balancers operate at the transport level and make their routing decisions based on address information extracted from the first few packets in the TCP stream and do not inspect packet content.

##### Application (Layer 7) load balancing

Level 7 load balancing deals with the actual content of each message enabling the load balancer to make smarter load balancing decisions, can apply optimizations and changes to the content (such as compression and encryption) and make decisions based on the message content. Layer 7 gives a lot of interesting benefits, but is less performant.

**Note**: Layers 4 and 7 are a part of [OSI Model](#osi-model).

Load balancing can be software and hardware based.

Software Load balancing can be done by an [API gateway](#api-gateway) or a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy).

There are multiple algorithms for implementing load balancing: round-robin, least connection, hashing based, etc.

References:

- [System Design — Load Balancing](https://medium.com/must-know-computer-science/system-design-load-balancing-1c2e7675fc27)
- [What Is Layer 4 Load Balancing?](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)
- [What Is Layer 7 Load Balancing?](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)
- [System Design — Proxies](https://medium.com/must-know-computer-science/system-design-proxies-ef5f2c2676f2)
- [Round Robin Load Balancing Definition](https://avinetworks.com/glossary/round-robin-load-balancing/)

### Databases

Database can be a bottleneck of the entire application. Below are some patterns that can help scale databases.

#### Indexing

[Database index](https://en.wikipedia.org/wiki/Database_index) is a data structure that is used for improving db querying speed.

Creating indexes is one of the first things you should consider when you need to improve database read performance. To be able to create effective indexes, here are a few important things to consider:

- You need to know what index data structure works best for your particular use case. Most common index data structure is a [B-Tree](https://en.wikipedia.org/wiki/B-tree), it works well for most cases. But in some cases (like full text search) you may need something else, like [GIN (Generalized Inverted Index)](https://pgpedia.info/g/gin.html).
- Your query may need a full index, but in some cases a [partial index](https://www.postgresql.org/docs/current/indexes-partial.html) can be a better choice
- When creating an index, an order of columns in an index may play a big role on how this index is going to perform. It is even possible to make query slower by creating a suboptimal index.
- While indexes can improve read performance, they slow down write performance since every time you insert something into a database each index has to be updated too. Create only minimum amount of indices that you will actually use, and remove all unused ones.

Indexing is a complex topic. Creating effective indexes requires deep understanding of how they work internally. Check out the references below for more in-depth guides.

References:

- [[YouTube] Things every developer absolutely, positively needs to know about database indexing](https://youtu.be/HubezKbFL7E)
- [Using Postgres CREATE INDEX](https://pganalyze.com/blog/postgres-create-index)
- [Index Advisor for Postgres](https://pganalyze.com/index-advisor) - a tool for PostgreSQL that can suggest indexes to make querying more efficient

#### Replication

[Data replication](<https://en.wikipedia.org/wiki/Replication_(computing)>) is when the same data is intentionally stored in more than one node.

Having multiple copies of data can improve read performance, availability, prevent disasters in cases when one of the databases loses its data.

References:

- [Read Consistency with Database Replicas](https://shopify.engineering/read-consistency-database-replicas)

#### Partitioning

[Partitioning](<https://en.wikipedia.org/wiki/Partition_(database)>) is a division of a database into distinct independent parts. When you have large amounts of data that don't fit into one database partitioning is a logical next step to improve scalability.

References:

- [Data partitioning guidance](https://docs.microsoft.com/en-us/azure/architecture/best-practices/data-partitioning)
- [Sharding pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/sharding)

#### Hot, Warm and Cold data

You can think about data as hot, cold or warm.

- **Hot storage** is data that is accessed frequently. For example blog posts created in the last couple of days will probably have a lot more attention than old posts.
- **Warm storage** is data that is accessed less frequently. For example blog posts that are created more than a week ago, but still get some attention from time to time.
- **Cold storage** is data that is rarely accessed. For example blog posts created long time ago that are rarely accessed nowadays.

When deciding what data to store where, it’s important to consider the access patterns. Hot data can be stored in a fast and expensive storage (SSDs), warm data can be stored in a cheaper storage (HDDs), and cold data can be stored on the cheapest storage.

References:

- [The Differences Between Cold, Warm, and Hot Storage](https://www.ctera.com/company/blog/differences-hot-warm-cold-file-storage/)
- [[YouTube] Data Partitioning! Don't let growth SLOW you down!](https://youtu.be/_zRXJuW7W98)

#### Database Federation

[Database Federation](https://en.wikipedia.org/wiki/Federated_database_system) allows multiple databases to function as one, abstracting underlying complexity by providing a uniform API that can store or retrieve data from multiple databases using a single query.

For instance, using federated approach you can take data from multiple sources and combine it into a single piece, without your clients even knowing that this data comes from multiple sources.

A simple example can be a functional partitioning, when we split databases by some particular function or domain. For example, imagine we have "users", "wallets" and "transactions" tables. In a federated approach this could be 3 different databases. This can increase performance since load will be split across 3 databases instead of 1.

Pros:

- Federated databases can be more performant and scalable
- Easier to store massive volumes of data
- Single source of truth for your data, meaning there is one place where it is stored using a single schema/format (unlike in [denormalized](#denormalization) databases)

But it also has its downsides:

- You'll need to decide manually which database to connect to.
- Joining data from multiple databases will be much harder.
- More complexity in code and infrastructure.
- Consistency is harder to achieve since you lose [ACID](https://en.wikipedia.org/wiki/ACID) transactions

References:

- [What is a Data Federation?](https://www.tibco.com/reference-center/what-is-a-data-federation)
- [Scaling up to your first 10 million users](https://youtu.be/kKjm4ehYiMs?t=2928) - federation discussed briefly starting at 48:48

#### Denormalization

[Denormalization](https://en.wikipedia.org/wiki/Denormalization) is the process of trying to improve the read performance of a database by sacrificing some write performance by adding redundant copies of data.

For example, instead of having separate "users" and "wallets" tables that must be joined, you put them in a single document that doesn't need any joins. This way querying database will be faster, and scaling this database into multiple partitions will be easier since there is no need to join anymore.

Pros:

- Reads are faster since there are fewer joins (or none at all)
- Since there is no joins it is easier to partition and scale
- Queries can be simpler to write

Cons:

- Writes (inserts and updates) are more expensive
- More redundancy requires more storage
- Data can be inconsistent. Since we have multiple copies of the same data, some of those copies may be outdated (or in some cases never updated at all, which is dangerous and must be handled by ensuring eventual [consistency](#consistency))

References:

- [Denormalization in Databases](https://www.geeksforgeeks.org/denormalization-in-databases/)
- [Building robust distributed systems](https://kislayverma.com/software-architecture/building-robust-distributed-systems/) - denormalization discussed briefly

#### Materialized views

[Materialized view](https://en.wikipedia.org/wiki/Materialized_view) is a pre-computed data set derived from a query specification and stored for later use.

Materialized view can be just a result of a `SELECT` statement:

```sql
CREATE MATERIALIZED VIEW my_view AS SELECT * FROM some_table;
```

Result of this `SELECT` statement will be stored as a view so every time you query for it there is no need to compute the result again.

Materialized views can improve performance, but data may be outdated since refreshing a view can be an expensive operation that is only done periodically (for example once a day). This is great for some things like daily statistics, but not good enough for real time systems.

References:

- [Working with Materialized Views](https://docs.snowflake.com/en/user-guide/views-materialized.html#:~:text=A%20materialized%20view%20is%20a,base%20table%20of%20the%20view.)
- [Materialized View pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/materialized-view)

#### Multitenancy

[Multitenancy](https://en.wikipedia.org/wiki/Multitenancy) is a software architecture in which a single instance of software runs on a server and serves multiple tenants.

There are multiple tenancy models for databases:

- Single federated database for all tenants
- One database per tenant
- Sharded multi-tenant database
- And others

Which model to choose depends on a scale of your application.

References:

- [Multi-tenant SaaS database tenancy patterns](https://docs.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns?view=azuresql)

#### SQL vs NoSQL

**SQL**, or relational, databases are good for structured data.

SQL databases are great for transaction-oriented systems that need strong [consistency](#consistency) because SQL Databases are [ACID](https://en.wikipedia.org/wiki/ACID) compliant.

Relational databases can be very efficient at accessing well structured data, as it is placed in predictable memory locations.

Relational databases are usually scaled vertically. Because in relational databases data is structured and can have connections between tables it is harder to partition / scale horizontally.

Relational databases are a better choice as a main database for most projects, especially new ones, and can be good enough up to a few millions of active users, depending on a use case.

SQL databases include: MySQL, PostgreSQL, Microsoft SQL server, Oracle, MariaDB and many more. Cloud providers like AWS, GCP and Azure include their own relational database solutions.

**NoSQL** is a good choice for unstructured/non-relational data. Because unstructured data is usually self-contained and consists of independent objects with no relations, it is easier to partition, thus it is easier to scale horizontally.

NoSQL / Non-Relational databases can be a good choice for large scale projects with more than few millions of active users, but since NoSQL databases lack ACID properties (NoSQL are usually eventually consistent) and relational structures, they are not the best choice as a main database for most projects on a low scale. Consider all pros and cons and choose wisely when picking a database.

NoSQL databases include: MongoDB, Cassandra, Redis and many more. Cloud providers like AWS, GCP and Azure include their own non-relational database solutions.

References:

- [SQL vs. NoSQL – what’s the best option for your database needs?](https://www.thorntech.com/sql-vs-nosql/)
- [[YouTube] How do NoSQL databases work? Simply Explained!](https://www.youtube.com/watch?v=0buKQHokLK8)

### Caching

A [cache](<https://en.wikipedia.org/wiki/Cache_(computing)>) is a high-speed data storage layer which stores a subset of data, typically temporary, so that future requests for that data are served up faster than by accessing the data’s primary storage.

For caching, you can use tools like [Redis](https://redis.io/), [Memcached](https://memcached.org/), or other key-value stores.

#### Caching strategies

There are several caching strategies worth knowing. Most common are:

- [Cache-aside](https://docs.microsoft.com/en-us/azure/architecture/patterns/cache-aside) (lazy loading)
- Write-through
- Write-behind
- Refresh-ahead

References:

- [Caching guidance](https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching)
- [Database Caching Strategies Using Redis](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/database-challenges.html)
- [Architecture Patterns: Caching (Part-1)](https://kislayverma.com/software-architecture/architecture-patterns-caching-part-1/)

#### Distributed caches

A [distributed cache](https://en.wikipedia.org/wiki/Distributed_cache#:~:text=In%20computing%2C%20a%20distributed%20cache,database%20and%20web%20session%20data.) is an extension of the cache from a single server to multiple servers. A distributed cache can improve the performance and scalability of the system.

References:

- [Distributed cache system design](https://medium.com/system-design-concepts/distributed-cache-system-design-9560f7dd07f2)
- [Architecture Patterns: Caching (Part-2)](https://kislayverma.com/software-architecture/architecture-patterns-caching-part-2/)

#### CDNs

[Content delivery network](https://en.wikipedia.org/wiki/Content_delivery_network) is geographically distributed group of servers which work together to provide fast delivery of Internet content.

A CDN allows for the quick transfer of assets needed for loading Internet content including HTML pages, javascript files, stylesheets, images, and videos.

CDN can increase content availability, redundancy, improve website load times.

References:

- [What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)

### Coupling

One reason why distributed systems are so hard to design is coupling. One of the ways you can scale your system is by reducing it.
Below are some topics and techniques that can help prevent coupling.

#### Location coupling

When a program assumes that something is available at a known location, we can call that a Location coupling.
For example, when one service knows the location of another service (lets say knows its URL).

It is difficult to horizontally scale location coupled systems. Location can change, or there may be multiple locations if there are multiple nodes running. Also systems can assume that accessing those locations may be fast, but usually in distributed systems a remote location can be on a entirely different server in a different country, thus a network trip to may be slow.

Location coupling is a problem when considering horizontal scalability because it directly prevents resources from being added dynamically at different locations.

To break location coupling you can abstract the specifics of accessing another part of the system. Your application should be agnostic to the called system.

This can be implemented differently in different scenarios:

- At the network layer, we can use DNS to mask the specific IP addresses of remote servers
- Load balancers can hide that there are multiple instances of some particular system are running to service high workloads
- Clients can hide the details of sharded database clusters
- When using microservices, you can use message broker (described below).

Abstracting the specifics of accessing another part of the system makes application depend only on that abstraction instead of keeping bunch of locations it needs. This lets application scale better. For example, this lets you dynamically add more services and instances to the network when you need it (during peak hours) without changing anything.

##### Broker pattern

[Broker pattern](https://en.wikipedia.org/wiki/Broker_pattern) is used to structure distributed systems with decoupled components. These components can interact with each other by remote service invocations. A broker component is responsible for the coordination of communication among components.

In application that consists of microservices, your services shouldn't know the location of each other. Instead, you can publish your commands/messages/events to a common place (like a message broker) and let other services pick it up. Now all of your services will know about a common location (message broker), but not about each other. Prefer [Asynchronous communication](#asynchronous-communication) when using message brokers.

References:

- [Broker pattern](https://medium.com/@lalosaimi/broker-pattern-297ac3cff6c5)

#### Temporal coupling

Temporal coupling usually happens in synchronously communicated systems (a part of a system expects all other parts on which it depends to serve its needs instantaneously). If one part fails, all its dependent systems will also fail, creating a chain of failures.

The most common approach to breaking temporal coupling is the use of message queues. Instead of invoking other parts of a system synchronously (calling and waiting for the response), the caller instead puts a request on a queue and the other system consumes this request when it is available.

##### Message queues

A [message queue](https://en.wikipedia.org/wiki/Message_queue) is a component used for [Inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication). In distributed systems typically used for [asynchronous service-to-service communication](#asynchronous-communication). Messages are stored on the queue until they are processed by a consumer.

Each message should be processed only once by a single consumer ([idempotent consumer](#idempotent-consumer)).

Software examples: [RabbitMQ](https://www.rabbitmq.com/) and [Kafka](https://en.wikipedia.org/wiki/Apache_Kafka).

References:

- [System Design — Message Queues](https://medium.com/must-know-computer-science/system-design-message-queues-245612428a22)
- [Queue-Based Load Leveling pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling)

#### Business data and logic coupling

To be autonomous, each component of a system must own its domain data and logic (this is called data ownership).

Try not to share business data between components. Otherwise you may end up with a distributed monolith where every change can impact and break the whole system. This is especially relevant in microservices world.

> Regardless of the method we use to share data across service boundaries we’ll end up with services that are not autonomous. Autonomous services can evolve independently from each other, non-autonomous ones cannot. They are coupled, they won’t be able to evolve without breaking each other, causing evolution to stop or causing friction at development time, deployment time, and run time.
>
> As soon as we allow cross-service data sharing, for example by allowing services to request data from another service, we end up with the worst of two worlds: a monolith suddenly turns into a distributed monolith with all distributed system problems on top of a monolithic one.

Exception to this rule:

- Sharing IDs. For example an "order" microservice may need to know what user made an order. In this case you can save a user ID in both services, but not the entire user object with all its properties.
- Some data, lets say email address, can be needed in multiple services, for example authentication service (to login your user) and a marketing service (to send marketing emails). In those cases you may need to duplicate emails between those services. Duplication is better than calling one service from another, but keep in mind that when updating email you'll need to guarantee tis update in both services. We discuss this further in a section below.

Your components must be independently deployable. In a microservices world, if deploying one microservice you have to change other microservices this means you have a tightly coupled architecture.

#### Shared-nothing architecture

[Shared-Nothing Architecture](https://en.wikipedia.org/wiki/Shared-nothing_architecture) is a distributed computing architecture that consists of multiple separated nodes that don’t share resources.

This architecture reduces coupling, scales better, simplifies upgrades preventing downtime, eliminates single points of failure, allowing the overall system to continue operating despite failures in individual nodes, etc.

References:

- [Shared Nothing Architecture Explained](https://phoenixnap.com/kb/shared-nothing-architecture)

#### Selective data replication

Selective data replication is creating a copy of the data needed from other remote system (external api, microservice etc) into the database of our system.

This approach reduces coupling, runtime dependency, improve latency and makes system more scalable and reliable. Even if external source of data is unaccessible, you still have this data replicated in your own database.

Data that is frequently accessed but changes regularly can be cached temporarily with periodic cache refreshes. Data that changes less frequently or never can be stored in our system's database directly.

References:

- [Querying data across microservices](https://medium.com/@john_freeman/querying-data-across-microservices-8d7a4667668a)

#### Event-driven architecture

An event is a broadcast by a software system about something which has happened within its boundary. For example, when the system successfully create an order, it can publish an `ORDER_CREATED` event.

Using asynchronous communication and event-driven architecture ensures loose-coupled components in a system.

TODO

References:

- [Why Microservices Should Be Event Driven: Autonomy vs Authority](https://www.google.com/search?client=firefox-b-d&q=consistent+hashing)
- [Using Events to build evolutionary architectures](https://kislayverma.com/software-architecture/using-events-to-build-evolutionary-architectures/)
- [[YouTube] Moving to event-driven architectures](https://youtu.be/h46IquqjF3E)

##### Event-driven end-to-end

In typical web pages, when you load a page and data on the server changes afterwards, the browser does not know about this change until you reload the page. The state displayed on a page is stale cache that is not updated until you explicitly poll or query for changes.

More recent protocols have moved beyond simple request/response pattern. [WebSockets](https://en.wikipedia.org/wiki/WebSocket) provide communication channels which allow your browser to establish a TCP connection to your server. Your server can push messages to your browser as long as it remains connected. This provides an ability for the server to actively inform end-users about any state changes on the server.

Recent tools for developing client applications, like React, Redux, Flux etc. already manage client-side state by subscribing to a stream of changes that represent responses from a server. It would be very natural to extend this model to also allow server to push state-changing events to a client. This way changes and events could flow end-to-end, from the interaction of one device that triggers an event, to the server and event bus, to processing this event, storing it in a database, and back to the UI client that is observing the state changes on another device.

The idea of subscribing for changes instead of querying opens a bunch of new possibilities for more responsive user interfaces. It is already used in some places, like online games or messaging apps.

For typical data centric or CRUD services there may be not a lot of benefit designing an entire application using this approach, but some parts of your application can definitely benefit from it.

References:

- [6 Event-Driven Architecture Patterns — Part 1](https://medium.com/wix-engineering/6-event-driven-architecture-patterns-part-1-93758b253f47)

##### Designing around dataflow

Some microservices example:

- In regular approach, when purchasing a product, your microservice can query an exchange rate service to get currency exchange rate.
- In a dataflow approach, the code that processes purchases would subscribe to a stream of exchange rate updates ahead of time and record the current rate in a local database whenever it changes. When processing a purchase it only needs to query its local database ([Selective data replication](#selective-data-replication)).

Same as changing a single number in a spreadsheet column will change the output of a formula that depends on this value.

> Subscribing to a stream of changes, rather than querying the current state when needed, brings up closer to a spredsheet-like model of computation: when some piece of data changes, any derived data that depends on it can swiftly be updated. - <small>Martin Kleppman</small>

By combining techniques like [Event-driven Architecture](#event-driven-architecture) and [Selective Data Replication](#selective-data-replication) we can achieve interesting results.

References:

- [Designing Data-Intensive Applications](https://dataintensive.net/) chapter: "Designing applications around dataflow"

## Consistency

Distributed systems are made up of parts which interact relatively slowly and unreliably. Asynchronous networks can duplicate data, drop packets, reorder messages, have delays, etc. Data can be spread across multiple data stores so managing and maintaining consistency in this environment is an important aspect of the system.

- **Weak consistency** - After a write, there is no guarantee that reads will see it. Works well in real time use cases such as voice or video chat, and realtime multiplayer games, etc.
- **Strong consistency** - After a write, there is a guarantee that reads will se it. Data is replicated synchronously, usually using some kind of [transaction](https://en.wikipedia.org/wiki/Database_transaction).
- **Eventual consistency** - After a write, readers will eventually see written data (usually within seconds) and data is replicated asynchronously. Eventual consistency works best for distributed systems at a large scale.

Below we will discuss where and why we need consistency, how to achieve it and some associated problems.

References:

- [Data Consistency Primer](<https://docs.microsoft.com/en-us/previous-versions/msp-n-p/dn589800(v=pandp.10)>)

### Different views on data

Sometimes there is no single piece of software that satisfies all your requirements. One database may not be enough for all your needs. Application wants to view data as [OLTP](https://en.wikipedia.org/wiki/Online_transaction_processing), analytics team want to view it as [OLAP](https://en.wikipedia.org/wiki/Online_transaction_processing). With high loads, you may also need to give users an ability to access data very fast using [cache](<https://en.wikipedia.org/wiki/Cache_(computing)>) or [full-text search](https://en.wikipedia.org/wiki/Full-text_search). There also may be benefit in creating connections between different data using a [graph database](https://en.wikipedia.org/wiki/Graph_database).

Typically you may need to use:

- Relational databases like [MySQL](https://www.mysql.com/), [PostgreSQL](https://www.postgresql.org/) etc
- Document databases like [MongoDB](https://www.mongodb.com/), [Couchbase](https://www.couchbase.com/) etc
- Graph databases for data with complex relations like [Neo4j](https://neo4j.com/developer/graph-database/), [dgraph](https://dgraph.io/), [ArangoDB](https://www.arangodb.com/) etc.
- For caching you can use [Redis](https://redis.io/) or [Memcached](https://memcached.org/)
- Search engines like [Elasticsearch](https://www.elastic.co/), [Solr](https://solr.apache.org/) etc.

When there is no single storage solution that satisfies all your needs, you may end up composing your services out of multiple software solutions that work together tied by your code. Same data may end up in multiple different storages to satisfy everyone's needs while querying, thus creating a lot of redundancy and data duplication, but providing performance benefits and ability to analyze and query data from different perspectives.

Keeping writes to several storage systems in sync is necessary. Just writing to multiple databases without any additional layer that ensures consistency is a naive approach that can lead to dual-write problems and inconsistencies. We will discuss techniques and patterns that can help reliably replicate data to multiple storages in the next sections.

References:

- [Online analytical processing (OLAP)](https://docs.microsoft.com/en-us/azure/architecture/data-guide/relational-data/online-analytical-processing)

### Distributed transactions

Another reason why distributed systems are hard is consistency problems.

In a typical non-distributed application, you can use database transactions to achieve strong consistency pretty easily. But in a distributed world, when databases can be on a different servers, simple transactions won't work anymore so we need to find workarounds. And with those workarounds we usually can only achieve weak or eventual consistency, because strong consistency usually means low performance and coupling, which is unacceptable when you distribute.

#### Two-phase commit (2PC)

[Two-phase commit (2PC)](https://en.wikipedia.org/wiki/Two-phase_commit_protocol) is one of the ways how to achieve strong consistency in a distributed world. Using 2PC, each server participating in a transaction starts a local transaction, and they all commit only when other participants finish executing the transaction.

2PC can guarantee strong consistency, but it is rarely used because:

- It is blocking. If the coordinator fails permanently, some participants will never resolve their transactions
- It is slow and affects performance of participating systems
- It creates coupling between systems

References:

- [Two Phase Commit](https://martinfowler.com/articles/patterns-of-distributed-systems/two-phase-commit.html)

#### Sagas

In modern distributed systems, eventual consistency is preferred because it scales better. But how do we achieve it?

One of the patterns for this is Sagas. A saga is a sequence of local transactions, but unlike 2PC, each transaction commits immediately, without waiting for other participants to finish. This way, all transactions will be committed asynchronously(eventually). Each participant must have a compensating mechanism to revert a saga, so if one of the participants fails, saga should start a revert process to undo all the changes.

References:

- [Pattern: Saga](https://microservices.io/patterns/data/saga.html)
- [Saga distributed transactions](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)
- [Compensating Transaction pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction)

##### Orchestration

Orchestration is a centralized approach. Parts of the system that are participating in a workflow are controlled by a single entity called Orchestrator, which is responsible for executing each step in a flow and ensuring consistency of the entire operation.

Orchestrated flows are explicit. It works nicely for complex workflows with a lot of steps since it is easy to keep track of an entire flow in one place.

##### Choreography

Choreography is a decentralized peer-to-peer approach. Unlike orchestration, when using choreography there is no central place that coordinates workflows. Each part of the system is responsible for subscribing to events it needs. This way data just flows through the system from one part of the system to another.

Choreography is flexible, allows a high degree of independence and autonomy. However, it poses a challenge because many business activities require a logical explicit chain of events. In choreography, flow of events is implicit. When you have complex workflows with a lot of steps, it will be hard to track, understand, maintain, monitor and troubleshoot large-scale flow of activities that are happening across the system. One event may trigger another one, then another one, and so on. To track the entire workflow you'll have to go multiple places and search for an event handler for each step. In this case, using an orchestration might be a preferred approach compared to choreography since you will have explicit workflows gathered in one place.

Choreography in some cases may be harder to scale with growing business needs and complexities. Pub/sub model works for simple flows, but has some issues when the complexity grows:

- Process flows are “embedded” within the code of multiple applications
- Often there is tight coupling and assumptions around input/output, [Service Level Agreements (SLA)](https://en.wikipedia.org/wiki/Service-level_agreement), event schemas, etc., making it harder to adapt to changing needs.
- You lose sight of the larger scale flow. It is hard to track steps and overall status of the workflow

To solve some of these problems you could use a [workflow engine](#workflow-engines) reading all the events in a flow and check if they can be correlated to a tracking flow.

A lot of systems use both orchestration and choreography depending on the use case, you don't necessarily need to choose just one. Pick the right tools for the job.

References:

- [Microservice Orchestration Vs Choreography](https://softobiz.com/microservice-orchestration-vs-choreography/)

##### Process manager

[Process manager](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html) is used to maintain the state of the sequence and determine the next processing step based on intermediate results.

Process manager is very similar to Orchestrator. In fact, a lot of people use the terms like "Saga" and "Orchestration" when in reality they are talking about the process manager.

Process managers are state machines that store some state, while sagas usually don't have a state and operate only on data that they receive in events.

- [Saga vs. Process Manager](https://blog.devarchive.net/2015/11/saga-vs-process-manager.html?m=1)

### Workflow Engines

Workflow engines are [workflow management systems](https://en.wikipedia.org/wiki/Workflow_management_system) that facilitate orchestration of microservices or data flows (like [ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load)). They provide APIs to define a set of tasks that need to be executed and gives tooling that can help visualize flows (as diagrams and/or [Directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)) and handle workflow monitoring, logging, retries, handling failures, scheduling, etc.

Workflow engines:

- [Apache Airflow](https://github.com/apache/airflow) - is a platform created by AirBnb to programmatically author, schedule, and monitor workflows.
- [Google cloud workflows](https://cloud.google.com/workflows) - easily build reliable applications, process automation, data and machine learning pipelines on Google Cloud.
- [Camunda](https://camunda.com) - The Universal Process Orchestrator
- [Prefect](https://www.prefect.io/) - Python based. Build, run, and monitor data pipelines at scale
- [Conductor](https://netflix.github.io/conductor/) - Workflow Orchestration engine created by Netflix that runs in the cloud.

You can build [Business Process Model and Notation (BPMN)](https://en.wikipedia.org/wiki/Business_Process_Model_and_Notation) diagrams for your workflows using tooling provided by <https://bpmn.io/>

References:

- [The Microservices Workflow Automation Cheat Sheet](https://blog.bernd-ruecker.com/the-microservice-workflow-automation-cheat-sheet-fc0a80dc25aa?gi=b977a74ee087)
- [Architecture options to run a workflow engine](https://blog.bernd-ruecker.com/architecture-options-to-run-a-workflow-engine-6c2419902d91)
- [Microservices and Workflow Engines](https://dzone.com/refcardz/microservices-and-workflow-engines)
- [[YouTube] Prefect Tutorial | Indestructible Python Code](https://youtu.be/0IcN117E4Xo)
- [Awesome workflow engines](https://github.com/meirwah/awesome-workflow-engines)

### Dual writes problem

A dual write describes the situation when you change data in 2 systems, e.g., a database and Apache Kafka, without an additional layer that ensures data consistency over both services.

For example:

```javascript
await database.save(user);
await eventBus.publish(userCreatedEvent);
```

Something can happen in between those operations (a network lag, process restart, event broker crash, etc.) so the second operation can fail resulting in a loss of data. This problem is called "dual writes problem". Dual writes are unsafe and can lead to data inconsistencies. This scenario can happen if you have multiple asynchronous operations executing one by one in a single place, so you should design your system to execute only one such operation at a time, or have some process that guarantees consistency.

There are multiple techniques to avoid dual writes, like Outbox Pattern and Retrying failed steps. We describe them in details below.

References:

- [Dual Writes – The Unknown Cause of Data Inconsistencies](https://thorben-janssen.com/dual-writes/)

#### The Outbox Pattern

Subscribing to an event from a different part of the system and creating your own view of that event (saving it to the database in a way that your system needs) is one of the ways we introduce redundancy. But saving some data and publishing an event that notifies other parts of the system about what happened can face dual write problems.

_The Outbox Pattern_ can help with consistency when writing data to a database and publishing an event to a message broker.

What you have to do is save your message/event as a part of database transaction, together with a result of an operation that is being executed. For example, if you are creating a user and sending an event that notifies other services that a user was created, you should save a user object and a message that you want to publish in a single database transaction. User goes to a "user" table and a message goes to an "outbox" or "messages" table. After that a separate Message Relay process publishes the events inserted into database to a message broker. You can use tools like [Debezium](https://debezium.io/) that publish changes from your "outbox" table to kafka, or create a custom process that checks database every X seconds and sends pending events.

This way we can be sure that no message/event is lost and will be published eventually, even if a message broker is down or a network is lagging. This will guarantee consistency of your data across multiple services and reliability of your application overall.

**Note**: when using [Event Sourcing](#event-sourcing) you may not need an Outbox Pattern since all the changes are stored as events anyway.

References:

- [Pattern: Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html)
- [The Outbox Pattern](http://www.kamilgrzybek.com/design/the-outbox-pattern/)

#### Retrying failed steps

Another way to prevent dual write problems is by retrying. For example, create a job that makes sure data is eventually consistent in a case of failure.

Consider this example: you need to save some data in your local database and index it in an external API

```typescript
await database.save(movie);
await externalAPI.index(movie);
```

What if your application crashes in the middle of this operation, or if external API is unavailable? Data will be saved to a local database, but not indexed in an external API.

One of the ways to solve this is to have a periodic job that queries the database every X minutes and retries indexing data that was not indexed.

You can see this implemented in code in this repo: [Fullstack application example](https://github.com/Sairyss/full-stack-application-example/tree/master/apps/api/src/app/modules) - check out a movie and indexer services. Movie is saved as `indexed: false` by default and indexer service has a `@Interval()` decorator meaning that it is executed periodically to make sure that if something failed during indexing it will be retried later (meaning it is eventually consistent).

One important detail: when retrying make sure your consumers are idempotent.

### Idempotent consumer

Messages, Commands and Events can be delivered twice (or multiple times), for example, when consuming an event from a message broker, an acknowledgement can fail for some reason or event is retried due to a timeout. It is important to process those events only **once**, otherwise your data may end up corrupted or some unwanted actions can occur in the system (for example you could change your user's credit card twice, or send the same email multiple times etc). We need some mechanism for event de-duplication.

To prevent processing events twice you should make your event consumers [idempotent](https://en.wikipedia.org/wiki/Idempotence).

Some actions are by definition idempotent. For example, updating a username twice with the same name will end up in the same result. But some actions are not idempotent, like charging money from a wallet.

One of the most used ways to make consumers idempotent is saving some kind of idempotency token (uuid generated by a user, or an ID of the event) in the database, in the same atomic transaction used to save changes caused by this event and set a unique constraint on that id. This way, even if an event/message is received multiple times, it will be processed only once.

You can go even further and prevent idempotency and duplication issues from end to end, starting from your application's clients. You can generate an idempotency token (like UUID/GUID) on a client side (lets say on a web page), send it together with a request and propagate it through your entire system. This way even if a client sends a request twice by accident (for example by refreshing a web page with filled-in form) the operation will be executed only once (if you have an Idempotent consumer).

References:

- [Pattern: Idempotent Consumer](https://microservices.io/patterns/communication-style/idempotent-consumer.html)
- [Idempotent receiver](https://martinfowler.com/articles/patterns-of-distributed-systems/idempotent-receiver.html)
- [Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/)

### Change Data Capture

[Change data capture (CDC)](https://en.wikipedia.org/wiki/Change_data_capture) is a pattern that tracks changes to data in a database in real-time allowing to process it continuously as new database events occur.

CDC is one of the techniques that can help with data consistency when you deal with multiple storage systems.

In a typical database, when you save something, data is first appended to a commit log (also called change log or [transaction log](https://docs.microsoft.com/en-us/sql/relational-databases/logs/the-transaction-log-sql-server?view=sql-server-ver15)) based on a [log data structure](https://scaling.dev/storage/log). This commit log is a source of truth, and tables are merely a views of the commit log.

We can utilize that log to duplicate changes from a main database to other databases consistently. Tools like [debezium](https://debezium.io/) allow publishing database changes to your streams ([Kafka](https://kafka.apache.org/)).

Why don't we simply save data to multiple databases at once? Because this is not safe and leads to [dual-write problems](#dual-writes-problem).

When data crosses the boundary between different technologies, asynchronous event log with [idempotent consumers](#idempotent-consumer) is reliable and robust solution.

References:

- [Change Data Capture (CDC): What it is and How it Works](https://www.striim.com/change-data-capture-cdc-what-it-is-and-how-it-works/)
- [Domain Events versus Change Data Capture](https://kislayverma.com/software-architecture/domain-events-versus-change-data-capture/)

### Time inconsistencies

If you need to determine an order of some events, the most intuitive way is using timestamps: event that has a lower timestamp should be first, right? Wrong. Clocks are unreliable, especially in distributed systems. Hardware can drift, threads, OS and runtimes can sleep, system can synchronize time using [NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol) causing time to jump forwards or backwards, current time can be resynchronized between different systems, etc.

Below we will discuss few patterns to deal with that.

#### Lamport timestamps

[Lamport timestamp](https://en.wikipedia.org/wiki/Lamport_timestamp) algorithm is a simple logical clock algorithm used to determine the order of events in a distributed computer system.

References:

- [Lamport Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/lamport-clock.html)
- [The trouble with timestamps](https://aphyr.com/posts/299-the-trouble-with-timestamps)

#### Vector clocks

[Vector clock](https://en.wikipedia.org/wiki/Vector_clock) is a data structure used for determining the partial ordering of events in a distributed system and detecting causality violations.

References:

- [Vector Clocks in Distributed Systems](https://www.geeksforgeeks.org/vector-clocks-in-distributed-systems/)
- [Vector Clocks](https://sookocheff.com/post/time/vector-clocks/)

### Conflict-free replicated data type (CRDT)

[Conflict-free replicated data type (CRDT)](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) is a data structure which can be replicated across multiple computers in a network, where the replicas can be updated independently and concurrently without coordination between the replicas, and where it is always mathematically possible to resolve inconsistencies that might come up.

References:

- [crdt.tech](https://crdt.tech/)
- [Automerge](https://github.com/automerge/automerge) - JavaScript library of data structures based on CRDTs for building collaborative applications

### Consensus algorithms

#### Byzantine generals problem

[Byzantine generals problem](https://en.wikipedia.org/wiki/Byzantine_fault)

TODO

#### Raft

[Raft](<https://en.wikipedia.org/wiki/Raft_(algorithm)>) is a consensus algorithm used in distributed computing (for example, in distributed databases).
Raft servers are either _leaders_ or _followers_. If a _leader_ becomes unavailable, consensus is achieved via _leader_ election by the _followers_.

References:

- [The Raft Consensus Algorithm](https://raft.github.io/)
- [Raft - Understandable Distributed Consensus](https://thesecretlivesofdata.com/raft/) - visualization of a Raft algorithm

#### Paxos

[Paxos](<https://en.wikipedia.org/wiki/Paxos_(computer_science)>)

TODO

---

## Other topics/patterns

### Event Sourcing

When using Event sourcing your data is saved as a sequence of immutable events.
This is valuable when auditing is needed, for example when dealing with money transactions.

TODO

References:

- [Event sourcing pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
- [Event immutability and dealing with change](https://www.eventstore.com/blog/event-immutability-and-dealing-with-change)

#### CQRS

CQRS stands for Command and Query Responsibility Segregation.

When using CQRS we write data to a database that provides fast writing speeds, and create a copy of that data in a database that is optimized for reads. Commands are forwarded to a write database, queries are forwarded to a read database. This can improve read and write performance of your application.

Your application can be separated into commands and queries using CQS pattern, you can read about it and see code examples here: [Commands and Queries](https://github.com/Sairyss/domain-driven-hexagon#commands-and-queries)

CQRS is a great fit when used together with Event Sourcing, but can also be used without it.

References:

- [CQRS pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)

#### Projections

Projections are usually used in Event Sourced systems to create a "view" from a stream of events by reducing those events to a single value (imagine JS [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) function `[event1, event2, event3].reduce(...)`). This is essentially a [left-fold](<https://en.wikipedia.org/wiki/Fold_(higher-order_function)>) operation in the functional world.

With projections you can create as many views on your data as you want. Projection can be a simple SQL table, a Document in a NoSQL database, a node in a Graph Database, etc. This can satisfy query needs for everyone, since you can't really query much from a stream of events.

Usually you would subscribe to a stream of events and change projection state when any event happens. A naive approach would be to change projection state in the database directly by executing a query for each event, but this will be very slow during replays. To make projections performant during replays consider loading a batch of events and changing state in-memory first using reducer/left-fold operation combined with [pure functions](https://en.wikipedia.org/wiki/Pure_function) and some patterns like [identity map](https://www.sourcecodeexamples.net/2018/04/identity-map-pattern.html), then execute all queries in batches instead of doing it one by one.

References:

- [Projections 1: Theory](https://www.eventstore.com/blog/projections-1-theory)

### Actor Model

[The actor model](https://en.wikipedia.org/wiki/Actor_model) is a computer science concept of concurrent computation that uses "actors" as the fundamental agents of computing (similarly to objects in [OOP](https://en.wikipedia.org/wiki/Object-oriented_programming) world). Unlike typical OOP objects, Actors only communicate by messages, so sending input and output is done only through message passing, not method calling. Actors can also create other actors, thus making an actor tree.

Actor model can be a great addition for distributed, concurrent and/or real time systems (like chats).

The most popular implementations of actor model include [Erlang](https://www.erlang.org/) programming language and [Akka](https://akka.io/) toolkit for [Java](https://www.java.com/en/) and [Scala](https://www.scala-lang.org/)

References:

- [How the Actor Model Meets the Needs of Modern, Distributed Systems](https://doc.akka.io/docs/akka/current/typed/guide/actors-intro.html)
- [The actor model in 10 minutes](https://www.brianstorti.com/the-actor-model/)

### API Federation

The goal of API federation is to expose resources from multiple parts of the system by providing a unified API with common and consistent schema, abstracting and hiding complexities of an underlying system that consist of many parts, services, databases, etc. (similarly to [database federation](#database-federation)).

API Federation describes set of practices and tools to strategically think about how to deal with complex APIs in organizations that are creating distributed systems (usually microservices).

For example, when we have multiple services that expose some related data:

- "Customer" service exposes data related to your customer
- "Orders" service exposes customers orders
- "Shipping" service exposes shipping status of an order

Customer may want to query his orders together with a shipping status and some other details. Instead of querying all three of those services separately, in federated approach this could be only one request that gets unified internally into one response, making it easier for clients to use your APIs.

Federated APIs can be built using unified HTTP [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete#:~:text=In%20computer%20programming%2C%20create%2C%20read,computer%2Dbased%20forms%20and%20reports.) interfaces, or using [GraphQL](https://graphql.org/) graphs.

References:

- [API Federation: growing scalable API landscapes](https://engineering.salesforce.com/api-federation-growing-scalable-api-landscapes-a0f1f0dad506?gi=c5efe3b03777)
- [How Netflix Scales its API with GraphQL Federation](https://netflixtechblog.com/how-netflix-scales-its-api-with-graphql-federation-part-1-ae3557c187e2)
- [Introduction to Apollo Federation](https://www.apollographql.com/docs/federation/)

### Chaos Engineering

[Chaos Engineering](https://en.wikipedia.org/wiki/Chaos_engineering) is a practice of experimenting on distributed system (usually in production environment) to test its resilience by introducing some chaos: disabling parts of the systems like services and servers, simulating hard drive failures, adding artificial latency, simulating high traffic, etc. to test how the system will behave in these conditions.

Distributed systems are inherently chaotic. Parts of such systems communicate through unreliable networks, which means those interactions can cause unpredictable outcomes. Chaos Engineering helps uncover system's weak spots and find unreliable parts to make them more resilient. The harder it is to disrupt the steady state, the more confidence we have in the behavior of the system.

Here are some tools for Chaos Engineering:

- [Chaos Monkey](https://netflix.github.io/chaosmonkey/)
- [Gremlin](https://www.gremlin.com/)
- [Chaos Mesh](https://chaos-mesh.org/)

When practicing Chaos Engineering it is important to use [monitoring](https://github.com/Sairyss/backend-best-practices#monitoring), [logging](https://github.com/Sairyss/backend-best-practices#logging) and tracing tools to have proper observability of the system.

Read more:

- [[YouTube] What Is Chaos Engineering?](https://www.youtube.com/watch?v=r4M0RM3QGxk)
- [Chaos engineering](https://www.techtarget.com/searchitoperations/definition/chaos-engineering)
- [5 Best Chaos Engineering Tools](https://harness.io/blog/devops/chaos-engineering-tools/)
- [Chaos Engineering and Observability with Visual Metaphors](https://www.infoq.com/articles/chaos-engineering-observability-visual-metaphors/)

### Distributed tracing

Distributed tracing is a method of observing requests and transactions as they propagate through distributed systems (like microservices). Tracing Tools help pinpoint where failures occur, what causes poor performance, and lets you trace cross-process transactions.

Distributed tracing is a must-have component for organizations that have complex distributed systems and workflows.

- [OpenTelemetry](https://opentelemetry.io/) - a collection of tools, APIs, and SDKs to instrument, generate, collect, and export telemetry data (metrics, logs, and traces).

References

- [Learning Distributed Tracing 101](https://tracing.cloudnative101.dev/docs/index.html)
- [Evolving Distributed Tracing at Uber Engineering](https://eng.uber.com/distributed-tracing/)
- [[Book] Mastering Distributed Tracing](https://www.shkuro.com/books/2019-mastering-distributed-tracing/)

### Bulkhead pattern

The Bulkhead pattern is a type of application design that is tolerant of failure. In a bulkhead architecture, elements of an application are isolated into pools so that if one fails, the others will continue to function. It’s named after the sectioned partitions (bulkheads) of a ship’s hull. If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.

References:

- [Bulkhead pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/bulkhead)

### Local-first software

TODO

References:

- [Local-first software](https://www.inkandswitch.com/local-first/)

### Consistent Hashing

[Consistent hashing](https://en.wikipedia.org/wiki/Consistent_hashing)

TODO

References:

- [Consistent Hashing](https://medium.com/system-design-blog/consistent-hashing-b9134c8a9062)

#### Hash ring

TODO

References:

- [Consistent Hash Rings Explained Simply](https://akshatm.svbtle.com/consistent-hash-rings-theory-and-implementation)

### Peer-to-peer

In [peer-to-peer (P2P)](https://en.wikipedia.org/wiki/Peer-to-peer), individual components are known as peers. Peers may function both as a client, requesting services from other peers, and as a server, providing services to other peers. A peer may act as a client or as a server or as both, and it can change its role dynamically with time.

Peer-to-peer is used in file-sharing networks (torrents), cryptocurrency-based products like (blockchain, bitcoin), etc.

#### Gossip protocol

[Gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol) is a peer-to-peer communication mechanism that allows designing highly efficient distributed communication systems (P2P).

References:

- [[YouTube] Parallel & Distributed Computing - Gossip Protocol](https://www.youtube.com/watch?v=qJpPjzg44R8)

#### Distributed Hash Table (DHT)

[Distributed Hash Table](https://en.wikipedia.org/wiki/Distributed_hash_table) is a distributed system / decentralized data store that provides a lookup service based on key-value pairs similar to [hash tables](https://en.wikipedia.org/wiki/Hash_table). DHT provides an easy way to find information in a large collection of data because all keys are in a consistent format, and the entire set of keys can be partitioned in a way that allows fast identification on where the key/value pair resides.

References:

- [Distributed Hash Tables: Architecture and Implementation](https://www.usenix.org/legacy/publications/library/proceedings/osdi2000/full_papers/gribble/gribble_html/node4.html)

##### Kademlia

[Kademlia](https://en.wikipedia.org/wiki/Kademlia) is a distributed hash table for decentralized peer-to-peer computer networks.

Kademlia provides a way for large amount of computers to self-organize into a network, communicate with other computers on the network, and share resources (like files and blobs).

Kademlia is used by projects like [BitTorrent](https://en.wikipedia.org/wiki/BitTorrent), [IPFS](https://en.wikipedia.org/wiki/InterPlanetary_File_System), [Ethereum](https://en.wikipedia.org/wiki/Ethereum), [I2P](https://en.wikipedia.org/wiki/I2P), and others.

References:

- [Distributed Hash Tables with Kademlia](https://codethechange.stanford.edu/guides/guide_kademlia.html)

##### Chord

[Chord](<https://en.wikipedia.org/wiki/Chord_(peer-to-peer)>) is a protocol and algorithm for a peer-to-peer distributed hash table.

### OSI Model

OSI stands for [Open Systems Interconnection](https://en.wikipedia.org/wiki/OSI_model). The OSI Model is a conceptual model that defines a common basis for network communication standards for the purpose of systems interconnection. OSI Model defines a logical network and describes computer packet transfer by using various layers of protocols.

You need to know basics of OSI model in order to have a better understanding of some problems in distributed computing (for example Layer 4 and 7 load balancing).

It is a 7 layer architecture with each layer having specific functionality to perform:

1. Physical layer - responsible for the physical connection between the devices
2. Data-link - responsible for the node-to-node delivery of the message
3. Network - works for the transmission of data from one host to the other located in different networks
4. Transport - provides services to the application layer and takes services from the network layer
5. Session - responsible for the establishment of connection, maintenance of sessions, authentication, ensures security
6. Presentation - data from the application layer is extracted here and manipulated as per the required format to transmit over the network
7. Application - this layer is implemented by the network applications

---