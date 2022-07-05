---
title: Welcome to my blog
date: "2022-04-24T16:38:03.284Z"
description: "Hello World"
---

![Hello World Image](./hello_world.jpg)

## Hi!

The primary job of any database management system is reliably storing data and making it available to users.

Databases are modular systems and consist of multiple parts: 
    
a transport layer accepting requests, 
a query processor determining the most efficient way to run queries, 
an execution engine carrying out the operations, 
and a storage engine

@picture

The storage engine (or database engine) is a software component of a database management system responsible for storing, retrieving, and managing data in memory and on disk, designed to capture a persistent, long-term memory of each node.

While databases can respond to complex queries, storage engines look at the data more granularly and offer a simple data manipulation API, allowing users to create, update, delete, and retrieve records. One way to look at this is that database management systems are applications built on top of storage engines, offering a schema, a query language, indexing, transactions, and many other useful features.

For flexibility, both keys and values can be arbitrary sequences of bytes with no prescribed form. Their sorting and representation semantics are defined in higher-level subsystems. For example, you can use int32 (32-bit integer) as a key in one of the tables, and ascii (ASCII string) in the other; from the storage engine perspective both keys are just serialized entries.

Every database system has strengths and weaknesses.

Database choice is always a combination of the factors like 
(

how the database performs, 
but also helps you learn how to operate, debug, 
and find out how friendly and helpful its community is
)

, and performance often turns out not to be the most important aspect: it’s usually much better to use a database that slowly saves the data than one that quickly loses it.


To compare databases, it’s helpful to understand the use case in great detail and define the current and anticipated variables, such as:

    - Schema and record sizes
    
    - Number of clients
    
    - Types of queries and access patterns
    
    - Rates of the read and write queries
    
    - Expected changes in any of these variables

Knowing these variables can help to answer the following questions:

    * Does the database support the required queries?
    
    * Is this database able to handle the amount of data we’re planning to store?
    
    * How many read and write operations can a single node handle?
    
    * How many nodes should the system have?
    
    * How do we expand the cluster given the expected growth rate?
    
    * What is the maintenance process?


Database management systems can serve different purposes: some are used primarily for temporary hot data, some serve as a long-lived cold 
storage, some allow complex analytical queries, some only allow accessing values by the key, some are optimized to store time-series data, and some store large blobs efficiently

# Next blog ?
DBMS Architecture

Every database is built slightly differently, and component boundaries are somewhat hard to see and define. 

The architecture presented in Figure 1-1 demonstrates some of the common themes in these representations.

Database management systems use a client/server model, where database system instances (nodes) take the role of servers, and application instances take the role of clients.

Client requests arrive through the transport subsystem. Requests come in the form of queries, most often expressed in some query language. The transport subsystem is also responsible for communication with other nodes in the database cluster.

Upon receipt, the transport subsystem hands the query over to a query processor, which parses, interprets, and validates it. Later, access control checks are performed, as they can be done fully only after the query is interpreted.

The parsed query is passed to the query optimizer, which first eliminates impossible and redundant parts of the query, and then attempts to find the most efficient way to execute it based on internal statistics (index cardinality, approximate intersection size, etc.) and data placement (which nodes in the cluster hold the data and the costs associated with its transfer). The optimizer handles both relational operations required for query resolution, usually presented as a dependency tree, and optimizations, such as index ordering, cardinality estimation, and choosing access methods.

The query is usually presented in the form of an execution plan (or query plan): a sequence of operations that have to be carried out for its results to be considered complete. Since the same query can be satisfied using different execution plans that can vary in efficiency, the optimizer picks the best available plan.

The execution plan is carried out by the execution engine, which aggregates the results of local and remote operations. Remote execution can involve writing and reading data to and from other nodes in the cluster, and replication.

Local queries (coming directly from clients or from other nodes) are executed by the storage engine. The storage engine has several components with dedicated responsibilities:

Transaction manager
This manager schedules transactions and ensures they cannot leave the database in a logically inconsistent state.

Lock manager
This manager locks on the database objects for the running transactions, ensuring that concurrent operations do not violate physical data integrity.

Access methods (storage structures)
These manage access and organizing data on disk. Access methods include heap files and storage structures such as B-Trees (see “Ubiquitous B-Trees”) or LSM Trees (see “LSM Trees”).

Buffer manager
This manager caches data pages in memory (see “Buffer Management”).

Recovery manager
This manager maintains the operation log and restoring the system state in case of a failure (see “Recovery”).

Together, transaction and lock managers are responsible for concurrency control (see “Concurrency Control”): they guarantee logical and physical data integrity while ensuring that concurrent operations are executed as efficiently as possible.

TODO - Check how it works on PostgreSQL



# Next blog In Memory versus Disk Based DBMS ?

Database systems store data in memory and on disk. In-memory database management systems (sometimes called main memory DBMS) store data primarily in memory and use the disk for recovery and logging. Disk-based DBMS hold most of the data on disk and use memory for caching disk contents or as a temporary storage. Both types of systems use the disk to a certain extent, but main memory databases store their contents almost exclusively in RAM.

Accessing memory has been and remains several orders of magnitude faster than accessing disk,1 so it is compelling to use memory as the primary storage, and it becomes more economically feasible to do so as memory prices go down. However, RAM prices still remain high compared to persistent storage devices such as SSDs and HDDs.

Main memory database systems are different from their disk-based counterparts not only in terms of a primary storage medium, but also in which data structures, organization, and optimization techniques they use.

Databases using memory as a primary data store do this mainly because of performance, comparatively low access costs, and access granularity. Programming for main memory is also significantly simpler than doing so for the disk. Operating systems abstract memory management and allow us to think in terms of allocating and freeing arbitrarily sized memory chunks. On disk, we have to manage data references, serialization formats, freed memory, and fragmentation manually.

The main limiting factors on the growth of in-memory databases are RAM volatility (in other words, lack of durability) and costs. Since RAM contents are not persistent, software errors, crashes, hardware failures, and power outages can result in data loss. There are ways to ensure durability, such as uninterrupted power supplies and battery-backed RAM, but they require additional hardware resources and operational expertise. In practice, it all comes down to the fact that disks are easier to maintain and have significantly lower prices.

The situation is likely to change as the availability and popularity of Non-Volatile Memory (NVM) [ARULRAJ17] technologies grow. NVM storage reduces or completely eliminates (depending on the exact technology) asymmetry between read and write latencies, further improves read and write performance, and allows byte-addressable access. (https://dl.acm.org/doi/10.1145/3035918.3054780)


Before the operation can be considered complete, its results have to be written to a sequential log file. We discuss write-ahead logs in more detail in “Recovery”. To avoid replaying complete log contents during startup or after a crash, in-memory stores maintain a backup copy. The backup copy is maintained as a sorted disk-based structure, and modifications to this structure are often asynchronous (decoupled from client requests) and applied in batches to reduce the number of I/O operations. During recovery, database contents can be restored from the backup and logs.

Log records are usually applied to backup in batches. After the batch of log records is processed, backup holds a database snapshot for a specific point in time, and log contents up to this point can be discarded. This process is called checkpointing. It reduces recovery times by keeping the disk-resident database most up-to-date with log entries without requiring clients to block until the backup is updated.

Disk-based databases use specialized storage structures, optimized for disk access. In memory, pointers can be followed comparatively quickly, and random memory access is significantly faster than the random disk access. Disk-based storage structures often have a form of wide and short trees (see “Trees for Disk-Based Storage”), while memory-based implementations can choose from a larger pool of data structures and perform optimizations that would otherwise be impossible or difficult to implement on disk [GARCIAMOLINA92]. Similarly, handling variable-size data on disk requires special attention, while in memory it’s often a matter of referencing the value with a pointer.

For some use cases, it is reasonable to assume that an entire dataset is going to fit in memory. Some datasets are bounded by their real-world representations, such as student records for schools, customer records for corporations, or inventory in an online store. Each record takes up not more than a few Kb, and their number is limited.




