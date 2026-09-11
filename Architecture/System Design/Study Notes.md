# System Design: Study Notes

## Interview Framework ⭐⭐⭐

* **Step 1: Understand the problem (3 \- 10 mins)**
    * **Slow down** and think deeply
    * **Ask questions** about **requirements**, assumptions, etc
    * Example questions:
        * What features are we going to build?
        * How many users does the product have?
        * Traffic volume? Expected scale over 3/6/12 months?
        * Web vs mobile?
        * etc...

* **Step 2: Propose a high-level design, get buy-in (10 \- 15 mins)**
    * **Collaborate** with the interviewer. Ask for feedback.
    * **Draw diagrams**, do some back-of-envelope calculations
    * Step through a few **use cases** or flows. Consider edge cases
    * **Communicate** as you work

* **Step 3: Design deep-dive (10 \- 25 mins)**
    * At this point you should have:
        * Agreed on overall goals and scope
        * Sketched out a high-level design
        * Obtained feedback
        * Had some initial ideas to focus on for the deep dive
    * Work with interviewer to identify and prioritize components in the architecture
    * **Time management** is essential
        * Skip unnecessary details

* **Step 4: Wrap up (3 \- 5 mins)**
    * Things you may cover here:
        * Recap of the design
        * Identify bottlenecks
        * Review error cases (eg: server failure, network failure)
        * Operational: metrics, error logs, deployment
        * How to handle the next scale curve (eg: 1M \=\> 10M users)
        * Propose other refinements
    * Dos:
        * Ask for clarification
        * Understand the requirements
        * Communicate\! Speak your thoughts out loud
        * Suggest multiple approaches
        * Bounce ideas around, get buy-in
        * Don't give up\! 🙂
    * Don'ts:
        * Don't be unprepared for typical questions
        * Don't jump into a solution without clarifying requirements and assumptions
        * Don't go into too much detail on a single thing
        * Don't hesitate to ask for hints
        * Don't remain silent

**From ChatGTP:**

1. Phase 1: requirements
    1. Clarify requirements
    2. Estimate scale
2. Phase 2: initial solution
    1. Define APIs
    2. Draw the high-level architecture
    3. Explain the data model
3. Phase 3 / 4: deep dive \+ wrap up
    1. Identify bottlenecks
    2. Scale each bottleneck
    3. Discuss reliability
    4. Discuss consistency
    5. Explain tradeoffs

---

## Databases (SQL vs NoSQL)

* **Relational Databases**
    * Examples: MySQL, PostgreSQL
    * Good at:
        * storing highly structured data
        * handling relations between data
        * ensuring data integrity via schema and FK constraints
        * ACID transactions \=\> safe and reliable
        * powerful query language (joins, etc)
        * Scaling for higher *read* thoughput
    * Bad at:
        * Scaling to massive storage size
        * Horizontal scaling
        * Super low latency
        * Sparse data (empty columns)
        * Storing unstructured / semi-structured data, documents, json
        * Changing schema (complex, takes time)
    * Best for:
        * Structured data, high transaction safety and consistency requirements
* **NoSQL Databases:**
    * Examples: MongoDB, Cassandra, CouchDB, Neo4j, DynamoDB, and many others...
    * Good at:
        * Scaling out
        * Low latency
        * (Depending on DB) Storing documents, JSON, key/value, graph data
        * High write throughput
        * Changing schema (may even be schemaless)
    * Bad at:
        * Strict transaction safety / Eventual Consistency
        * Less powerful queries (against depends on DB)
    * Best for:
        * Unstructured data, graphs, documents etc
        * Low latency, high throughput, huge datasets
        * Ability to scale horizontally
* **Database Replication**
    * This is the process of replicating data to all servers within the DB cluster
    * Options:
        * Leader / follower, multi-leader, leaderless
        * Synchronous: ensures data is written to followers before user can proceed.
        * Async: doesn't wait for followers to write. Can lead to data data loss.
    * For an RDBMS like MySQL or Postgres, we typically use:
        * Binary log (**binlog**) replication
        * **Source-replica** (i.e. leader-follower)
        * Need to configure replication user, replication source, etc
        * Strictly consistent
    * For NoSQL, we typically prioritize availability over consistency. Approaches:
        * Leaderless replication
            * Write is successful when a **quorum** of nodes ack the write
            * **Last write wins** to resolve write conflicts, using timestamp or vector clock
            * **Anti-entropy:** Nodes continuously compare data via Merkle trees to fix inconsistencies
            * Any node can serve as writer
            * Eventually consistent
        * Automated master-slave
            * Used by MongoDB. One server is master, but if it goes offline then the followers will elect a new leader.
            * Like for RDBMS, only leader can write.
            * Strongly consistent
    * **Merkle Trees:**
        * These data structures allow for quickly determining if two large data sets are identical. They also allow for quick location of differences.
        * Root nodes contain data, and the hash of the data
        * Parent nodes contain hash of (child1.hash \+ child2.hash)
        * A change to a root node will change hashes in all parent nodes up to root
        * Comparison of root nodes in two trees can tell if entire datasets are equal
        * To locate differences: walk down child nodes to locate differing leaf nodes
    * **Vector Clocks**
        * A complicated "counter" used to track data versions and fix conflicts across multiple servers / nodes
        * All servers have their own counter, starting at 0
            * Eg: `{Node A: 0, Node B: 0, ... Node N: 0}`
        * When updating local data, a node increments its counter
        * All nodes send their vector clock when replicating data or sending messages
        * When receiving a vector clock from another server, the local server will update  its clock to the max of its values and the other clock's values
        * To detect conflicts:
            * If clock A has *all* values \< clock B, then A happened before B
            * If clock A has some values greater and others less than B, then there is a conflict, as multiple updates happened on different servers out of order. This can be flagged for conflict resolution.
        * **Conflict resolution:**
            * **Application code**: The DB can store both values and allow the application to resolve it (eg: in Amazon you may add an item to your cart on your laptop, and another item via your phone. The application resolves this by adding *all* items to your cart).
            * **Last Write Wins:** just take the write with the highest wall clock time. Simply but can lead to data loss.
            * **Conflict-Free Replicated Data Types (CRDTs)**: somehow allows for independent and concurrent writes without locking. (I don't understand these yet)
        * Why not just use wall time?
            * Because wall clocks can differ among servers, and network comms have latency, so this approach can easily fail.
* **ACID Transactions:**
    * A set of four properties that make DB transactions reliable:
        * **Atomicity**	The transaction happens **all or nothing**
        * **Consistency**	The database remains in a **valid state**
        * **Isolation**	Concurrent transactions don't improperly interfere
        * **Durability**	Data stays committed, even after a crash
    * Some common examples:
        * Atomicity:
            * Suppose we run this transaction to move money:
                * `Subtract $100 from account A`
                * `Add $100 to account B`
            * If the DB crashes between queries 1 and 2, then *no changes are made*
        * Consistency:
            * Suppose the DB enforces `account_balance >= 0`
            * If a query attempts to set `account_balance = -$100`, then the query fails and an error is returned
            * NB: this is not the same "consistency" as all users seeing the same data. It refers to data in the DB following all defined constraints.
        * Isolation:
            * Suppose we have two concurrent transactions:
                * `Transaction 1: Withdraw $100`
                * `Transaction 2: Withdraw $200`
            * How this is handled depends on the **consistency levels**
            * The point is to prevent reading of stale or intermediate values
        * Durability
            * Once the database reports `COMMIT successful`, then the changes are permanent. A server crash or restart won't affect the data.
            * Typically done via **Write-Ahead Log (WAL)**.
* **Isolation Levels**
    * Core question:
        * "*When multiple transactions run concurrently, how much of each other's activity are they allowed to observe?*"

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads | Concurrency |
| :---- | :---- | :---- | :---- | :---- |
| Read Uncommitted | Possible | Possible | Possible | Highest |
| Read Committed | Prevented | Possible | Possible | High |
| Repeatable Read | Prevented | Prevented | Usually Prevented \* | Medium |
| Serializable | Prevented | Prevented | Prevented | Low |

*
    * **Read Uncommitted**
        * Transactions can see changes made by other txns *before* they are committed.
        * This is a **dirty read** and is rarely acceptable
    * **Read Committed**
        * A transaction can only read *committed* data
        * However, can lead to race conditions in which txn B changes a value halfway through txn A, and then txn A uses the updated value.
        * This means txn A can read different data depending on what *other* transactions are running, or not running. This is called a **non-repeatable read** because running the txn twice could yield different results (depending on other txns running).
        * I.e. the view of the data can be *inconsistent*. Eg:
            * Transaction A:		`SELECT balance;`  \=\> $1000
            * Transaction B:		`SET balance = $500; COMMIT;`
            * Transaction A: 		`SELECT balance;`  \=\> $500 ❌
    * **Repeatable Read**
        * Now the DB gives transactions a *consistent* view of the data, regardless of what other transactions are running concurrently.
        * Eg:
            * Transaction A:		`SELECT balance;`  \=\> $1000
            * Transaction B:		`SET balance = $500; COMMIT;`
            * Transaction A: 		`SELECT balance;`  \=\> Still $1000 ✅
        * Often implemented using **MVCC — Multi-Version Concurrency Control**
            * DB maintains different versions per transaction
            * No need to lock everything that's being read \=\> higher concurrency
    * **Serializable**
        * Strongest isolation
        * Guarantees data is equivalent to running transactions *serially* (even though they can run in parallel)
        * DB can use **locks, MVCC, conflict detection, or serialization checks**.
        * Due to higher constraints, **a transaction may fail and need to be rolled back**.
    * Unsurprisingly, more protections \= higher data consistency but lower concurrency/speed
    * Problems:
        * **Dirty Reads:**
            * One transaction reads a value written by another *uncommitted* transaction.
            * Almost always unwanted
        * **Non-repeatable Read:**
            * One transaction reads a value that was updated and committed by another concurrently-running transaction.
            * The view of data is *inconsistent* when run multiple times: other transactions can affect it.
            * Has to do with existing rows, but not number of rows.
        * **Phantom Read:**
            * Happens when the **number of rows** returned by a query changes. Eg:
                * Transaction A:	`SELECT names from people;`  \=\> 3 people
                * Transaction B:	`INSERT name="Joe"; COMMIT;`
                * Transaction A:	`SELECT names from people;`  \=\> 4 people
            * Problem: **The result set (\# of rows) changed**
            * This is different from a non-repeatable read:
                * Phantom read \= change to \# of rows
                * Non-repeatable read \= change to existing row
        * **Lost Update**:
            * Two transactions both write different values to a row. The second write clobbers the first. This is a **lost update** and is caused by lack of concurrency controls (e.g. locking or MVCC)
* **Write-Ahead Log (WAL)**
    * Ensures data consistency and crash recovery
    * Main idea: before updating database on disk, write it to a durable log
    * Why?
        * DB pages can be very large; want to avoid writing them to disk constantly
        * Instead, write to the WAL (fast). Then flush to disk less frequently.
    * Log is **append-only** for speed
    * So main steps are:
        * 1\. Transaction modifies data
        * 2\. Changes exist in memory
        * 3\. Write WAL record
        * 4\. Flush required WAL to durable storage
        * 5\. COMMIT succeeds
        * 6\. Modified database pages can be written to disk later
    * **Crash recovery & replay**:
        * After a crash, DB will read the WAL, check for any uncommitted transactions, and commit them. It is then up-to-date.
        * The WAL contains a **Log Sequence Number (LSN)** that increments for each write
        * DB pages also contain the last LSN of the write that last modified it
        * On recovery, the DB can compare the LSN of each modification to the LSN of each affected page. If the page's LSN is equal or greater, we can skip that update.
    * **Buffer Pool**:
        * This is the cache of frequently-accessed pages, in memory
        * Writes will mark a page as **dirty**
        * Background process flushes dirty pages to disk
    * **Checkpoints:**
        * DB periodically creates a checkpoint in the WAL to indicate that all transactions up to that checkpoint were reflected in the DB.
        * After a recovery, the DB only needs to replay txns from the last checkpoint.
    * **Replication:**
        * The WAL is a stream of changes
        * That stream can be sent to replicas to stay up-to-date with all writes
        * **Replication Lag** \= how far behind the replica is from the leader
* **Locks & Multi-Version Concurrency Control (MVCC)**
    * **Lock**:
        * Idea: transaction prevents other txns from modifying (or maybe even reading) any items in a row
        * Types of locks:
            * **Shared/read lock**: multiple txns can read concurrently; writers are blocked.
            * **Exclusive/write lock**: used when a txn wants to modify (write to) a row
        * Locks slow things down...
    * **MVCC: Multi-Version Concurrency Control**
        * Idea: Instead of requiring locks, which slow things down, **maintain different versions of row data** per txn.
        * Now: readers don't block writers and vice versa
        * What version of data should a txn see? \=\> Depends on isolation level.
        * MVCC **does not eliminate locks**. They are still needed for things like:
            * concurrent updates to a row
            * deletes
            * schema changes
            * explicit `SELECT ... FOR UPDATE`
            * uniqueness/constraint enforcement
            * serializable transactions
    * **SELECT FOR UPDATE**
        * Sometimes you want to (a) read a value, and then (b) update it
        * Normally this won't lock the value when you read it
            * eg: `SELECT balance FROM balances WHERE id=123;`
        * This leaves us open to another txn locking the row, or another race condition in which the value is updated.
        * By appending `FOR UPDATE` to the read, we tell the DB to lock the row for future update
        * This is called **pessimistic locking** as we assume a conflict might occur during the txn.
    * **Deadlocks**
        * Two transactions attempt to acquire multiple locks, and neither can proceed. Eg:
            * Txn 1:	Acquire lock A
            * Txn 2:	Acquire lock B
            * Txn 1:	Acquire lock B ... wait
            * Txn 2:	Acquire lock A .. wait
            * ... Deadlock\!
        * DB systems can detect deadlocks and will roll back one transaction
        * Fix: Acquire locks in the same order across all txns
    * **MVCC costs: old versions**
        * Old data versions can pile up and require ever more storage
        * Also when replaying from WAL, txns will need those old versions
        * DB systems have processes to detect and clean up old versions (eg: VACUUM on MySQL)
* **Foreign keys**
    * Concept: ensure that value (eg. id) must refer to a valid row in another table
    * Example: `Order` table has foreign key `customer_id`. I.e. the order must belong to a customer.
    * DB will check/enforce FK constraints:
        * Won't allow you to add a row with invalid FK
        * When deleting data (eg: deleting customers), the DB can handle this in a few ways:
            * **RESTRICT / NO ACTION**: Prevent the delete
            * **ON DELETE CASCADE**: When deleting a user, also delete all records referring to that user via FK. DB can also cascade updates.
            * **ON DELETE SET NULL**: Likewise, when deleting a user, set all FK records to null in other tables.
    * **Join tables**
        * Many-to-many relationships are modeled via a **join table** with multiple FKs
        * Eg a university might have:
            * `Students` table \- One per student
            * `Courses` table \- One per course
            * `student_courses` table \- Models many-to-many relationship, i.e. a student can take many courses, and each course has many students.
        * The **primary key** in the join table would be `(student_id, course_id)`
        * Considerations:
            * Checking FK constraints takes time. If performance is a real concern, teams may consider removing FK constraints. But this introduces dangers\!
            * If you **shard** your DB, then FK resources may not be satisfied on a single DB node. In this case the application must check FK constraints.
* **Main points:**
    * Locks prevent conflicting operations from happening simultaneously.
    * MVCC maintains multiple versions so readers can access a consistent snapshot without unnecessarily blocking writers.
    * Isolation levels determine which versions/changes a transaction is allowed to observe.
    * MVCC doesn't eliminate locking; concurrent writes still require coordination.
    * Locks can cause contention and deadlocks, while MVCC introduces version-storage and cleanup costs.

---

## CAP Theorem

* Concept in **distributed systems** in which you can choose at most 2 out of 3 desirable traits:
    * **Consistency**		\=\> Clients see a consistent/latest view of data
    * **Availability**		\=\> Every request to a working node returns a non-error response
    * **Partition Tolerance**	\=\> System continues to operate after communications failure bn nodes
* Best way to think of it (instead of "choose 2 of 3"):
    * ***When a network partition occurs, a distributed system must choose between consistency and availability.***
    * In other words, **you have two choices: CP or AP**
* What is a network partition?
    * \=\> a failure preventing nodes from communicating
    * Assume servers A and B both have X \= 10
    * A network partition occurs, and then server A receives a request to set X \= 20
    * ... what do we do?
* **Option 1: prioritize consistency (CP)**
    * Server A sees it can't communicate with server B (or more likely, reach quorum on writes)
    * \=\> Reject the request and protect consistency
    * Tradeoff: Strong consistency, low availability
* **Option 2: prioritize availability (AP)**
    * Server A sees it can't communicate with server B (or more likely, reach quorum on writes)
    * \=\> Accept the write anyway and allow data sets to diverge between nodes
    * This requires **conflict resolution / reconciliation** to resolve data inconsistencies
    * Tradeoff: High availability, poor consistency
* **Why not prioritize CA?**
    * You can't, because **network failures will happen**
    * My take: the only way this is possible is if there are no network failures. But that's not reality...
* **What does "Consistency" mean here?**
    * Linearizability
        * So if X \= 10, and node A writes X \= 20, and node B then reads X, it should get 20 and not 10
    * ie: "*Once a write successfully completes, later operations should behave as though that write happened atomically in a single global ordering.*"
    * Another way to think of it: *The distributed replicas behave like a single consistent system.*
* **What exactly does "Availability" mean?**
    * "*Every request received by a non-failing node eventually receives a response that isn't simply an error due to the partition."*
    * I.e. the system still allows requests to be processed (perhaps in a more limited fashion)
* **Real systems aren't simply CP or AP all the time**
    * During normal operations, systems are typically both available and consistent
    * CAP tradeoffs only occur **during** a partition
    * Different systems may require different tradoffs. Your banking app will require strong consistency. But product reviews can tolerate inconsistency (eg: older reviews) and so would prioritize availability.
    * So good system design often involves asking: "*What consistency guarantees does this particular piece of data actually require?*"
* **For interviews**
    * Look out for questions around **how you handle network failures**
    * Just think about whether the system / product requires consistency or availability. Choose one or the other.

---

## Horizontal vs Vertical Scaling

* Vertical:
    * Scaling *up* the node (or whatever) you are using
    * Eg: Using an EC2 instance with larger CPU, memory, bigger hard drive, etc
    * Pros:
        * It's easy
    * Cons:
        * There's a hard limit (eg: largest instance size)
        * If using a single server, you have no redundancy or failover capability
    * This is the quick hack you use if horizontal scaling isn't currently available, or your service requirements are low/simple
* Horizontal:
    * Scaling *out*, i.e. adding more instances / hosts / nodes
    * Eg: adding more web servers behind a load balancer
    * Pros:
        * Once supported, it can actually be VERY easy
        * Ability to scale to much higher levels
        * Host size no longer the bottleneck
    * Cons:
        * More difficult to implement at first
    * Overall, horizontal is much preferred

---

## Load Balancing

* **Purpose:**
    * Evenly distribute load among multiple backend servers
    * Horizontal scaling \- Add (or remove) servers behind the LB
    * Improve availability \- if a server goes down, we can always send to another
    * Hide server failures \- if a server goes down, take it out of rotation
* **Types:**
    * **L4**:
        * Works at OSI layer 4: Transport (i.e. TCP/IP)
        * Routes based on IP address and port
        * Can be very **fast**
        * **Simple**. Can't do complicated routing.
        * Useful for non-HTTP traffic
        * Has no insight into contents of packets
        * Typically performs **TLS passthrough** (i.e. it does not terminate TLS)
    * **L7**:
        * Application layer (**HTTP** requests)
        * Will **terminate TLS**
        * Has insight into packet contents, full request url, headers, body
        * Typically routes on URL, can also route on headers
        * Can perform request rewriting, http redirect, and other more complicated actions
        * Can also do **rate limiting** and **authentication** (at least in basic ways)
* **Algorithms:**
    * **Round Robin**
        * Route to each server in turn
        * Simple
        * Best used when all servers have identical capabilities, and request durations are relatively equal
    * **Weighted RR:**
        * Useful when servers have different capabilities
        * If server A is twice the size of server B, then give it twice the requests
    * **Least Connections**
        * Route to the server with the least number of conns
        * Useful when requests can have large differences in duration
    * **Consistent Hashing**
        * Hash the source IP or session ID \=\> routes client's requests to same server
        * Useful for **sticky sessions** and also **caching** \- i.e. any time you want a client's requests to go to the same server
    * **Least response time**
        * Route to servers showing lowest response time
        * Useful for latency-sensitive applications
* **Features**
    * **Health checks**
        * A simple recurring check to ensure each server is healthy
        * Typically done by hitting a particular URL on each server, or just opening a connection
        * On HTTP servers, the URL could trigger a set of internal checks
        * If the health checks fail N times, then take the server out of rotation
        * If the health checks succeed, put the server back in rotation
        * Allows for graceful handling of server failure
    * **SSL/TLS termination**
        * Normally the LB will terminate TLS (for an L7 load balancer). This is simpler because:
            * Only the LB needs to perform encryption / decryption
            * Only the LB needs certificates
            * Servers are not burdened with performing this work
        * Also, TLS termination is *required* for L7 routing because the LB needs to look at the request contents
        * However, TLS termination on the LB means traffic is unencrypted to the servers
        * L4 load balancers can do TLS passthrough, allowing each server to decrypt/encrypt.
            * This is more load on the servers
            * ... but it allows for end-to-end encryption
        * A few things:
            * The L7 LB *can* re-encrypt when passing to the servers, for added security
            * NGINX is frequently used for TLS termination, L7 load-balancing, and reverse proxy
            * Unencrypted HTTP is used in many cases but *only in private IP spaces.* You wouldn't do this across a public/shared network.
    * **Sticky sessions**
        * Sometimes we want to have requests from a specific user land on the same server
        * Reasons: to be able to use local caching on the server, for speed
        * This is done by hashing the session id, or user id, or something else to ensure the user is routed to the same server
        * Cons:
            * A single server might become a hotspot
            * If a server goes down, all that local data is lost and must be reloaded. User may even need to reauthenticate.
    * **Reverse proxy:**
        * For **incoming** requests (the normal pattern for load balancing)
        * External request is spread across many internal servers
        * Why:
            * Protects backend servers (which can be in internal IP space)
            * Rate limiting, load balancing, caching, TLS termination
            * May even do authentication
    * Forward proxy:
        * For **outgoing** requests
        * Server makes a request *through* the proxy, to an external service
        * The proxy then forward the request, acting as a middleman
        * Why:
            * Serves as gateway from private IP space to public internet
            * Can block access to certain sites
            * Protects backend servers (by keeping them in private IP)
* **Design Principles**
    * **Stateless services:**
        * We want services to be **stateless** so that any request can be handled by any server. If the server needs data, it requests it from another service, cache, or DB.
    * **Horizontal scaling:**
        * We can easily add more servers behind the LB to increase capacity
        * Because servers are stateless, we don't need to make any changes to routing, or perform complex initialization, etc
    * **Redundancy:**
        * Servers can fail at any time, but having multiple servers means we should always have some servers to handle requests.
        * If a server fails, the LB can quickly take it out of rotation to protect uptime.
    * **Auto scaling:**
        * The LB can add (or remove) servers automatically by monitoring things like: overall request rate, CPU/mem utilization, number of conns, processing time, etc

---

## Message Queues

* **Basic idea**
    * Serving a request may require calling many downstream services. Some of these can (and should) run **asynchronously** so as not to slow down other services. This is where queuing systems come into play.
    * MQs allow for easily making certain calls asynchronous
    * Example techs: Apache Kafka, RabbitMQ, Amazon SQS, and Google Pub/Sub.
* **Why are Queues so useful?**
    * **1\. Async Processing**
        * We can easily offload async work to the queues for processing later
        * Other work, like responding to the user, doesn't need to wait
    * **2\. Decoupling services**
        * Let's say our Order Service needs to query four other services:
            * Payments, Inventory, Email, and Analytics
        * Without queues or async processing, if we need to make direct synchronous calls to these services, that means our Order service is **coupled** to those 4 other services
            * If they change (eg: API changes), our service is affected and may fail
        * Instead, we can publish to a MQ
            * This **decouples** our service from the others, leading to **better flexibility**
    * **3\. Handling traffic spikes**
        * When traffic spikes occur, our backends can become overwhelmed.
        * To prevent this, we put work in a **work queue** that servers pull from; this acts as a **buffer**
        * A server will only pull requests from the queue if it can; all other requests can wait
        * This allows for queueing of requests and protects availability
        * This is often called **load leveling** or **buffering**.
    * **4\. Scaling workers**
        * The MQ system acts as a load balancer of sorts
        * We simply add more worker nodes to pull from the message queues
* **The harder part: delivery guarantees**
    * Suppose a worker runs and charges the user's credit card, and then **crashes...**
    * We have a few delivery guarantees we could make in this situation:
        * **At most once**: process 0 or 1 times. Messages could be lost.
        * **At least once**: process 1 or more times. Messages could be processed twice. Very common.
        * **Exactly once**: Exactly once. Very difficult to implement. Best to choose narrow circumstances.
    * MQ systems can easily retry messages, which makes at-least-once a common use case
* **Idempotency:**
    * If guaranteeing at-least-once delivery, we want to make sure that processing a message multiple times won't cause problems. This is called **idempotency.**
    * Using the above example, maybe the application charges the card the first time, but then detects that the card was charged already on the second run, and doesn't charge it.
    * You want idempotency in your MQ tasks\!
* **Failed messages and dead-letter queues**
    * Simple: if a message fails to process more than N times, move it to a **Dead-Letter-Queue (DLQ)** for storage.
    * Engineers can come back later to investigate and process those messages.
* **Ordering**
    * Ideally we want all messages to be processed in order. But this is difficult.
    * As a compromise, we can ensure ordering within a single shard / partition / key
        * A partition is probably a separate process
        * This is what Kafka does, IIRC.
* **Queue vs. pub/sub vs. event stream**
    * Queue: One-to-one. One worker consumes a given messages
    * Publish / Subscribe: one-to-many. A given message is broadcast to all consumers.
    * An **event stream/log**, such as Kafka, adds ideas like durable ordered logs, partitions, offsets, consumer groups, and replay. A consumer can replay logs from a given position (eg: if it crashes).
* **Partitions, consumer groups, and horizontal scaling**
    * This is for Kafka
    * Conceptually, messages go into an **ordered log**
    * But if we need to scale out to multiple consumers, we can't guarantee ordering
    * Instead, distribute messages to **partitions**. Each partition has it's own **ordered log.**
        * Kafka only guarantees ordered processing *within each partition*, but not across partitions
    * Each consumer reads from one or more specific partitions
        * But *only one* consumer reads from a given partition
    * Kafka uses a particular message field as the **partition key** to distribute messages to partitions
        * Be careful on selection of partition key. You want even distribution across partitions. Avoid **hot partitions.**
    * **Consumer groups**
        * A group of consumers
        * Within each consumer group, each partition is assigned to *at most one* consumer
    * **Partitions limit parallelism:**
        * Only one consumer in a group can read from a partition.
        * So if you have 4 partitions, it doesn't help to have more than 4 consumers
        * **BUT:** having *too many* partitions means more overhead and can hurt performance
        * Using multiple consumer groups is helpful
            * Say a message requires multiple types of processing
            * Have separate consumer groups, one per processing type (assuming they can be done in parallel)
            * This allows for parallel execution
* **Backpressure and queue depth**
    * Queue depth is the  number of messages waiting in the queue
    * It's normal for message receive rate to be higher than processing rate for short periods.
        * Queue acts as **buffer** and shock absorber
        * Workers can get to the waiting messages in time.
    * But if queue depth becomes *too* large, we have problems:
        * New messages have to wait a long time for processing \=\> **high processing latency**
        * Also need to **measure age of oldest message** (or wait time)
    * **Backpressure**:
        * Preventing a fast producer from overwhelming a slower downstream system.
        * Mechanism to prevent upstream components from sending unlimited work downstream.
    * **Queues create backpressure** by forcing messages to wait if workers aren't ready for them
        * But they just move the problem. If the incoming message rate is too high, something needs to change.
    * **Offset**:
        * The consumer group's position in the durable message log
        * Each consumer group has its own offset
    * **Consumer failure and Rebalance**
        * Say a consumer group has four consumers: A, B, C, D
        * If consumer C dies, its partitions are assigned to other consumers
        * Those consumers can pick up at the last recorded offset for the old consumer C.
* **Ways to handle backpressure**
    * **Add more workers**
        * Simple to do, and common
        * But too many workers can cause issues elsewhere (eg: overloading the DB). The bottleneck moves.
    * **Rate limiting**
        * Tell the client it's sending too fast (eg: via HTTP `429 Too Many Requests`)
        * Client can then slow down (if programmed to do so\!)
    * **Slow the producer**
        * Depends on queueing system implementation
        * It would support a way to tell producers to slow
    * **Reject new work**
        * Return error, eg: `System busy`
        * This is basic **load shedding**
    * **Drop low-priority work**
        * Another form of **load shedding**
* **Queue Depth for Autoscaling**
    * If queue depth becomes too large, add workers
    * Could even autoscale off of incoming request rate?
* **Queue depth vs. queue latency**
    * Queue depth by itself isn't extremely helpful
    * You need to know the **latency** (time to process) as well
    * Eg: queue depth \= 100K, but processing speed is 100K msg/sec. No problem.
    * But: queue depth \= 100K, but processing speed is 100 msg/sec. Big problem\!
    * Latency tells you how long a message must wait before processing. **Latency is more important than depth.**
    * Kafka uses **consumer lag**: the number of messages a worker needs to process to catch up
* **Key Ideas**
    * Queue depth / consumer lag is a measurement of backlog
    * Backpressure is a system behavior/strategy for controlling overload
    * A queue is not an infinite solution. It converts an immediate load problem to a backlog problem. Eventually you need to deal with that backlog by increasing capacity, reducing request rate, degrading functionality or dropping work.
* **For System Design Interviews**
    * Monitor queue depth and message latency
    * Autoscale when depth/latency is high
    * Put a limit on autoscaling
    * Implement backpressure (rate limiting, load shedding) to protect system if producers are too fast

---

## Database Failover

Look for _"How does database failover happen"_ in [this ChatGPT chat](https://chatgpt.com/c/6a8c7b75-4dcc-83e8-8e03-edaddffcbcff).

---

## API Design

* From [API Design for System Design Interviews](https://www.hellointerview.com/learn/system-design/core-concepts/api-design)
* **Common API Types:**
    * **REST** \- The default. Uses HTTP. Simple CRUD operations. Models *resources.*
    * **GraphQL** \- Invented by Facebook, lets client request exactly what it needs. More efficient.
    * **RPC** \- Efficient binary protocol used for internal inter-service comms. Models *procedures.*
* **Tips**:
    * Use REST 90% of the time
    * If you need real-time updates, you'll want something like **websockets**
    * Only spend \~5 minutes on API design in an interview
* **REST:**
    * **Resources** are the core entities. REST APIs operate on resources. These are **nouns.**
    * HTTP operators define the **verbs**
    * Path names should use the **plural** resource name.
    * Examples:
        * `GET /events`                    - Get all events
        * `GET /events/{id}`               - Get a specific event
        * `GET /venues/{id}`               - Get a specific venue
        * `GET /events/{id}/tickets`       - Get available tickets for an event
        * `POST /events/{id}/bookings`     - Create a new booking for an event
        * `GET /bookings/{id}`             - Get a specific booking
    * **Required data goes in the URL**
        * For example, the venue id is required to lookup the venue
    * **Optional data goes in the query params**
        * For example, optional filters when listing the set of venues, such as location.
    * For **relationships**, you have two options:
        * Nesting, eg: /events/{id}/tickets
        * Flat hierarchy, eg: /tickets?event\_id=123
    * **HTTP Methods:**
        * GET		Read resources
        * POST		Create a new resource. *Not idempotent\!*
        * PUT		Create or update a resource. Idempotent.
        * PATCH		Update existing resource. Idempotency depends on impl.
        * DELETE	Delete a resource
    * **Passing data:**
        * **Path params** to identify the particular resource(s)
        * **Query params** to affect filters, sorting, or other endpoint behavior
        * **Request body** contains data used to create or update a resource
    * **Returning data**
        * Done via **status code** and **response body**
        * Status codes:
            * 2xx \- Success
            * 3xx \- Extra action needed, like a redirect
            * 4xx \- Client error
            * 5xx \- Server error
        * Some common codes:
            * 400 Bad Request
            * 401 Unauthorized
            * 403 Forbidden
            * 404 Not Found
            * 405 Method Not Allowed
            * 409 Conflict
            * 429 Too Many Requests
            * 500 Internal Server Error
            * 504 Gateway Timeout
    * **Security:**
        * Done by securing each endpoint
        * Eg: Shopping app has access to `/products` but not `/employees`
* **GraphQL**
    * Created by FB in 2012 to solve inefficiencies with REST APIs
    * REST APIs often hit **N+1 queries:**
        * 1 query to list all resources
        * Plus N queries, one per resource
    * N+1 queries are very inefficient. GraphQL lets the client request *just the data they want*, resulting in fewer API calls and more a more responsive user experience.
    * Uses **a single endpoint per resource** and lets the user specify what data they want
    * Example schema and query:
        *   ```
            # SCHEMA

            type Event {
                id: ID!
                name: String!
                date: DateTime!
                venue: Venue!
                tickets: [Ticket!]!
            }

            type Venue {
                id: ID!
                name: String!
                address: String!
            }

            type Query {
                event(id: ID!): Event
                events(limit: Int, after: String): [Event!]!
            }


            # QUERY

            query {
                event(id: "123") {
                    name
                    date
                    venue {
                        name
                        address
                    }
                    tickets {
                        section
                        price
                        available
                    }
                }
            }
            ```
        * Notice how the relationships are defined in the schema
        * With a REST API, you'd need to make multiple requests for all of this data\!
    * **When to use GraphQL**
        * When your clients have diverse needs (eg: Web client needs different data than the mobile client)
        * GraphQL also allows for faster iteration; REST changes tend to rely on backend schema changes.
        * But normally, just use REST
    * **Security:**
        * Done at the **field-level** (different from REST)
* **RPC**
    * Remote Procedure Call
    * Allows one service to run a procedure on another service
    * Fast and efficient for internal inter-service communication
        * Uses binary encodings; no parsing of HTTP or JSON
        * Less data going across the network
    * **Not good for external-facing APIs**: too complex and requires coordinating software versions, etc
        * HTTP is better for public APIs, as everything knows HTTP
    * Operates on *procedures* instead of resources
        * Much more flexible. A procedure could do anything.
    * Popular RPC tech:
        * **gPRC,** which uses **protocol buffers**
        * Apache **Thrift**
    * **.proto files** are used by protocol buffers and extended by gRPC to include service definitions.
    * .proto file example:
        *   ```
            service TicketService {
                rpc GetEvent(GetEventRequest) returns (Event);
                rpc CreateBooking(CreateBookingRequest) returns (Booking);
                rpc GetAvailableTickets(GetTicketsRequest) returns (TicketList);
            }

            message GetEventRequest {
                string event_id = 1;
            }

            message Event {
                string id = 1;
                string name = 2;
                int64 date = 3;
                Venue venue = 4;
            }
            ```
        * Notice the `message` definitions define the objects
        * And the `service` contains a number of `rpc` procedures
        * .proto files are used by client and server to agree on the API, and thus the binary encoding
        * Type safety / type checking is also built in\!
    * **When to use RPC:**
        * For **internal** services
        * When **performance** is critical
        * For strong **type checking**
        * If you need **streaming** (supported by gRPC)
* **(REST) API Design Principles**
    * **Design around resources, not actions**
        * Nouns, not verbs\!
    * **Be consistent everywhere**
        * Naming conventions, parameter names, response shapes, etc
        * Consistency means users can easily predict how to use your API
    * **Follow the principle of least surprise**
        * Do what everyone else does. Eg: GET never writes data.
    * **Keep the API stateless**
        * The request should contain all necessary data
        * This allows for easy scaling behind the LB: just add more servers
    * **Make retries safe**
        * Failures can happen; want to allow clients to retry
        * Queries should be idempotent wherever possible
        * If non-idempotent methods (POST), use **idempotency keys**
    * **Paginate anything that can grow**
        * Assume any collection that *can* grow *will* grow, to beyond a page in length
        * Support pagination for all resource collections
        * Also set a cap on max page length to protect your service / app
    * **Secure every endpoint by default**
        * Make auth *opt-out* rather than *opt-in*
        * Every endpoint must require a security token by default
        * Only when explicitly marked public should an endpoint forego authentication
    * **Evolve without breaking clients**
        * When making changes, ensure backwards compatibility
        * Eg: Adding a new field is safe. Removing, renaming, or changing the meaning of an existing on is not
        * This is why API paths are often prefixed with a version, eg: `/v1/products/`, so we can create a new version
        * Example [versioning methods](https://redocly.com/blog/api-versioning-best-practices):
            * Path-based: `/api/v1/users` vs `/api/v2/users`
            * Param-based: `/api/users?version=2`
            * Header-based: `X-API-Version: 2.0`
    * **Make errors actionable**
        * On error, make sure the client (computer or human) can tell what went wrong, and what to do about it
        * Return 3 things:
            * The proper HTTP status code
            * Machine-readable error code, in body
            * Human-readable error message, in body
* **Common API Patterns**
    * **Pagination**
        * When returning a large set of results, we need to break up the set into pages
        * Done in two ways: offset-based and cursor-based. In either case we tell the server:
            * how many results we want per page
            * which page/offset to return
        * **Offset-based**
            * Simplest approach, most common
            * Tell server (a) how many results, and (b) at what offset (result number) to start
                * Eg: `GET /products?offset=20&limit=10`
            * Has **problems with large data sets**:
                * If a result was inserted between requests, it could be skipped, and/or cause duplicate results
                * This is because the **skip count** (number of initial records to skip) is recalculated with each request
        * **Cursor-based**
            * A **cursor** is a token returned to the client that references the next resource to start with
            * The client must pass that cursor on subsequent requests
            * Internally the cursor might reference a row / id in a database table somewhere
            * **Prevents missing or duplicate results** from being returned
                * This is because the API doesn't recalculate the resources to skip with each request.
                * Rather, it just picks back up from where it left off, via the cursor
            * Security tip: it is highly encouraged to **obfuscate** any internal implementation details so that an attacker cannot use the cursor for an attack.
                * Say for example that you want to use a database resource id as a cursor
                * You might encrypt the id via symmetric key encryption so only your servers can read the id
                * Or generate a UUID, cache `UUID => internal data` (short TTL), and fetch data on next request
    * **Filtering and Sorting**
        * Clients will frequently want to do this
        * **Use query parameters**
        * Eg: `/events?city=NYC&status=upcoming&sort=date`
        * Can also support sorting in reverse order via "minus" , eg: `sort=-date` (newest first)
        * Stay **consistent** across all endpoints
            * Mismatched names across endpoints is one of the most common consistency violations in real API reviews
    * **Idempotency Keys**
        * Used to support idempotency for non-idempotent request methods (eg. POST)
        * Idea:
            * Client generate idempotency key (token) and sends it with each request
            * Server stores the key along with each request so it can detect duplicate requests.
            * Stripe's API does this
            * Guessing it uses a cache in the backend to store these keys.
        * Example request:

| POST /events/123/bookings Idempotency-Key: 8e03978e-740c-4c2e-abd1-52f427d5c177 {   "tickets": \[{"section": "VIP", "quantity": 2}\] } |
| :---- |

    * **Consistent Error Responses**
        * Return 3 things: HTTP status code, error code, and error message
        * Eg:

| {   "error": {     "code": "SEAT\_UNAVAILABLE",     "message": "Section VIP has only 1 seat remaining"   } } |
| :---- |

        * The **error code** is for the application. These codes should not change
        * The **error message** is for humans and can change without breaking the app
    * **Versioning Strategies**
        * Path-based: `/api/v1/users` vs `/api/v2/users`
            * Simple, obvious to users, **preferred**
            * Allows for routing based on version
        * Param-based: `/api/users?version=2`
        * Header-based: `X-API-Version: 2.0`
* **Security Considerations**
    * Basic question: "Who is making the request? And are they allowed to do what they're asking?"
        * **Authentication**: Determines if the user is who they say they are
        * **Authorization**: Determines the permissions of the user
    * **API Keys**
        * Long, randomly-generated strings that act like passwords.
        * Must be included with each request.
        * Server generates keys, stores them in DB along with user, permissions, etc
        * **Good** for server-to-server communication, or to expose to 3rd-party devs
        * **Bad** for users: they shouldn't have to know about long tokens
        * Example request:
            	GET /events
            	Authorization: Bearer sk\_live\_abc123...
    * **JSON Web Tokens (JWT)**
        * Encode user information directly into the token itself
        * Server creates token with user id, permissions, expiration, and then cryptographically signs the token
        * Client will send the token with each request
        * Benefits:
            * No DB lookups; server can read info directly from token
            * Signed: any server with key can verify authenticity
        * Example token payload:
            *

| {   "user\_id": "123",   "email": "john@example.com",   "role": "customer",   "exp": 1640995200 } |
| :---- |

    * **Which to use?**
        * Use **API keys** for **internal service** communication and **external developer** access.
        * Use **JWT tokens** for **user sessions** in web and mobile applications.
    * **Role-Based Access Control (RBAC)**
        * System that assigns specific permissions on a user-level basis
        * On request, lookup permissions in the RBAC system before allowing action
        * Will require some sort of RBAC system, and a DB to hold the data
    * **Rate Limiting and Throttling**
        * Protects your system by preventing clients from sending too many requests too quickly
        * Common strategies:
            * **Per-user limits**: 1000 requests per hour per authenticated user
            * **Per-IP limits**: 100 requests per hour for unauthenticated requests
            * **Per-endpoint limits**: 10 booking attempts per minute to prevent ticket scalping
        * Typically done via **API gateway** or via application **middleware**
        * If rate limited, return `429 Too Many Requests`

## Distributed Consistency

* Main idea: **When different copies of your data disagree, what guarantees do you provide to users?**
*   | Layer | What it means | Examples |
    | :---- | :---- | :---- |
    | **Database consistency (ACID)** | Transactions preserve database correctness | Constraints, foreign keys, invariants |
    | **Replica consistency** | Different copies of the data agree | Strong consistency, eventual consistency, quorums |
    | **Client-observed consistency** | What guarantees an individual client experiences | Read-after-write, monotonic reads, causal consistency |

* **Database consistency (ACID)**
    * Addresses *single-server* DB consistency
    * Every transaction leaves the database in a valid state, preserving all constraints
    * Enforced through: foreign keys, constraints, triggers, transactions \+ rollback, ACID
* **Replica Consistency**
    * Addresses consistency of a *distributed database* / system
    * A constant concern for distributed systems (DB master \+ multiple replicas)
        * eg: Client 1 writes, then client 2 reads. What is returned?
        * Old value? New value? Depends on the **replica consistency** guarantees
    * **Strong Consistency**
        * Every future read returns the new value (for all users)
        * This is more desirable and easier to reason about, but also slower and more complicated
        * Used in finance, banking, inventory
    * **Eventual Consistency**
        * The old value is returned for some time, but eventually the new value is returned
        * "Eventually" is not defined
        * Simpler to implement. Often used in social media, DNS, CDNs, etc
* **Client-observed consistency**
    * Addresses what an ***individual user*** experiences over time
        * May not apply to the system as a whole
    * **Read-after-write consistency**
        * Ensures you read the thing you just wrote
        * The system might route you to updated servers intelligently
    * **Monotonic Reads**
        * Guarantees that once you read data from time T, you won't see older data
    * **Monotonic Writes**
        * Guarantees that writes are applied in the order they were made
    * **Causal consistency**
        * Users should see data from a timeline (eg: user posts ) in order, so they make sense
        * Eg: "I'm engaged" and "we got married" should appear in that order
    * **Linearizability**
        * Every operation appears to happen **atomically at a single instant in time**
        * Events also are reported in the order they were written
        * This is the **strongest** form of consistency (the same as *strong consistency* I think)
    * **Sequential consistency**
        * Everyone agrees on **one global ordering**
        * *But* that ordering doesn't have to agree on wall clock time (so **event order may not match order at write time**)
    * **Quorum Consistency**
        * Balances speed against correctness
        * You want to meet some minimum number of nodes to be written to before the write succeeds
        * Similarly, you want a minimum number of nodes to read from before the read succeeds
        * N \= \# nodes, W \= \# writers, R \= \# readers
        * You want to have `R + W > N`
* **For interviews, you can probably focus on these 3:**
    * Strong consistency			(don't show old data ever)
    * Read-after-write consistency		(don't show old data to individual user)
    * Eventual consistency			(old data is okay for a short period)

---

## Rate Limiting

* Touches so many underlying topics
    * scalability, distributed systems, consistency, caching, load balancing, user experience, security
* **What is it?**
    * **Restricts how frequently a client can perform some operation**
* Can be targeted:
    * Per user
    * Per API endpoint
    * Per IP address
* Examples:
    * 100 requests / min per user
    * 5 password attempts / min
    * 10 uploads / hour
    * 1000 API calls / day
* When limit is exceeded, return error: `HTTP 429 Too Many Requests`
    * Often with header: `Retry-After: 35`
* **Benefits:**
    * Protect servers
    * Prevent abuse \- brute force logins, scraping, DoS attaches, bots
    * Fairness \- one user can't hog all the resources
    * Cost control
* **Where should it be implemented?**
    * **Option 1: API Gateway**
        * Most common answer
        * It blocks traffic early and protects the servers
    * **Option 2: Load Balancer**
        * Similar to the API Gateway option
        * Requires some traffic inspection capabilities (eg: L7 LB)
    * **Option 3: Application**
        * Allows for custom business, user, or application-specific rules
        * But the servers must process the traffic
    * **Option 4: CDN / Edge**
        * Often done by large companies like Cloudflare
        * Traffic never reaches your infrastructure
        * Lower latency and bandwidth costs
* **How to rate limit?**
    * **Per IP**			Simple but imperfect
    * **Per User**		Better after authentication
    * **Per API Key**		Common for Developer APIs
    * **Per Organization**	For SaaS products
    * **Per Endpoint**		Difference endpoints have diff costs and risks
* **Rate Limiting Algorithms**
    * **Fixed Window Counter**
        * Maintain a counter that resets every M minutes or whatever
        * Very simple to implement
        * But how to handle this in a distributed environment?
        * Also allows for large bursts at roll-over boundaries (**boundary effect**)
        * In fact, you can get a burst of R requests right *before* the boundary, and another R requests right after, leading to 2R requests in a very short period of time.
    * **Sliding Window Log**
        * Maintain timestamps for each request, and a rolling window over the last time-limit range
        * Addresses the boundary effect from Fixed Window Counter
        * Might be implemented as a log or list of events. When a new request enters:
            * Remove any "old" requests from the log (older than the request interval)
            * Check log size. Is it larger than the max request rate? If so, throttle.
            * Else, allow request and add it to the log
        * More complex to implement, more overhead (memory) to run, but super accurate
    * **Sliding Window Counter**
        * Take the current and previous window, and weight them based on time into the current window
        * `requests_made = last_window_percent * requests_last_window + requests_this_window`
        * Smooths out boundary problem, doesn't as much memory
    * **Token Bucket**
        * Most common interview algorithm; choose this usually
        * Application contains a bucket full of tokens (eg: 100 tokens)
        * Every request uses a token
        * Tokens **refill continuously** over time, in **small increments**
            * For example, given a rate limit of 100 requests/sec, the bucket size would be 100 and the refill rate might be 1 token every 10 ms (or 10 tokens every 100 ms).
            * This smooths out the incoming traffic
        * When bucket is empty, requests are denied
        * Token bucket will allow short bursts while enforcing a long-term average rate
            * If bucket has some tokens T, it allows a burst of up to T requests before throttling
    * **Leaky Bucket**
        * Requests enter a **queue**
        * Requests **can be processed** (leave the queue) **at a predefined max rate**
        * This prioritizes a smooth rate, while token bucket allows some bursting
        * If the queue fills, additional requests are rate-limited
* Which to use?

| Algorithm | Good for |
| :---- | :---- |
| Fixed Window | Simple systems |
| Sliding Window Log | Highest accuracy |
| Sliding Window Counter | Good balance of accuracy and efficiency |
| Token Bucket | APIs, cloud services, most production systems |
| Leaky Bucket | Traffic shaping and smoothing |

* **Distributed Rate Limiting**
    * We need to coordinate rate limiting across multiple servers
    * **Centralized Store**
        * Used redis to store counters. All requests INCR the counter
        * Most common interview answer
        * Why Redis?
            * Very fast / low latency
            * atomic operations (two servers doing INCR won't clobber each other)
            * TTL support
            * Supports millions of counters efficiently
* Common interview follow-ups?
    * **What happens if Redis goes down?**
        * **Fail open**
            * Allow requests through. Prioritize availability.
            * Can choose this for "cheap" and/or important operations
        * **Fail closed**
            * Block all requests. Prioritize protecting the servers.
            * Maybe use these for specific, expensive operations
        * **Use fallback limits**
            * Use a backup method to degrade gracefully
            * Eg: use a small in-memory counter
    * **Should rate-limiting happen before authentication?**
        * Can happen before *and* after
        * For authentication requests, you can rate-limit by IP
        * Once authed, rate limit by user, API key, etc
    * **Can different users have different limits**
        * Of course\!
        * Could have higher limits for higher-paying service tiers
* **What I would prioritize for interviews**
    * For senior software engineering interviews, I'd be comfortable explaining:
        * **Why rate limiting exists** (protection, fairness, abuse prevention, cost control).
        * **Where to implement it** (edge/CDN, API gateway, load balancer, application).
        * **Token Bucket and Sliding Window Counter,** including their tradeoffs.
        * **How to build a distributed rate limiter** using a shared store like Redis.
        * **Race conditions and atomic operations** in a distributed environment.
        * **Failure scenarios** (Redis outages, fail-open vs. fail-closed behavior).
    * If you can confidently discuss those topics and justify your design choices based on the system's goals, you'll be well prepared for most rate-limiting questions in system design interviews.
    * **I can also walk you through a complete "Design a Distributed Rate Limiter" interview**, which is a classic 45-minute senior-level system design question that ties all of these concepts together.

---

## Consistent Hashing

* Key Idea:
    * **How to distribute data across many servers while minimizing disruption when servers are added or removed**
* The problem:
    * You have 4 servers and hash the user ID to choose a server:  `server_id = hash(user_id) % 4`
    * But this changes when you add a server\!
    * ... Resulting in data having to be shuffled around to new servers\! 😱
    * If you're using this for caching, then you'll get lots of cache misses; the cache becomes (almost) entirely cold\!
* The goal:
    * When I add a new server, only a **small fraction** of data needs to move
* The **hash ring**:
    * Imagine a ring, with servers as points on the ring
    * **When choosing a server**: hash your input key, get a point on the ring, and move clockwise until you reach one.
    * **When adding a server**: add the server as a new point on the ring.
        * All points between it and the previous server are the ones that need to move
        * No other points need to move\! They continue to "belong" to their existing servers.
    * **Removing a server**: remove the point on the ring. Keys just map to the next server clockwise.
        * Only the removed server's keys need to move
    * Why is this so **powerful**?
        * Because it **minimizes** the number of keys that need to move\!
* **Virtual Nodes**
    * If you randomly assign points on the ring, then keys may be imbalanced between servers
        * eg: Server A gets 60%, B gets 30%, C gets 10%
    * Instead, **each server appears multiple times**
        * This allows us to **more evenly distribute the key domain** between servers
* **Replication**
    * Can use the hash ring \+ replication factor to determine servers that must be updated
    * Eg: you are server B, with replication factor 3 \=\> update servers B, C, D (next 3 servers clockwise)
    * If one node dies, 2 others have the data
* **Hot Keys**
    * If a single key is really popular, consistent hashing by itself can't help.
    * Mitigations:
        * Replication with load-balanced reads
        * Caching (at multiple layers)
        * Split hot data into smaller pieces
* **How much data moves?**
    * Typically 1 / N, where N \= number of servers
    * Eg: Have 100 servers, add 1 \=\> need to move 1% of data
* **Examples**:
    * **Cassandra**: uses consistent hashing, virtual nodes, replication
    * **DynamoDB:** uses consistent hashing concepts for partitioning and scaling data across storage nodes
    * **Redis Cluster**: hashes keys to 16K slots, assigns slots to servers. It's *similar* to a hash ring.
    * **Memcached**: client libraries use consistent hashing
* **Problems with "basic" consistent hashing**
    * **1: Partitions are not of equal size:** servers don't hash onto the ring to form equal-sized partitions
    * **2: Keys aren't evenly distributed**: they likely will not be balanced among partitions (similar to \#1)
    * **3: Cascading failure:** if a server fails, all of its keys will be sent to a single other server, rather than multiple.
* **Improvements to "basic" consistent hashing**
    * We add many **virtual nodes** (hundreds or thousands)
        * Multiple virtual nodes will map to a single physical server
        * By doing this, we even out the partition size and key distribution
        * If a server fails, it's keys will be spread evenly among the remaining servers
    * How is this implemented:
        * We can simply append a virtual node suffix the server's name
        * Eg: `server1` \==\> `server1_v1, server1_v2, ... , sever1_v100`
        * Then we hash the key \=\> get virtual node \=\> determine physical node
* **Common Interview Q's:**
    * Why not just use modulo?
        * Because too many keys will need to move when adding/removing a server
    * Why virtual nodes
        * To ensure a more even distribution of keys among servers
        * Can also assign more virtual nodes to larger servers
    * What happens when a node dies?
        * Only its keys need to move. Others do not.
    * Does consistent hashing eliminate rebalancing?
        * No, it just minimizes the amount of data movement
    * Is the ring required?
        * Not strictly. You could do something like Redis's shards. The goal is to minimize data movement on rebalancing.

---

## Service Discovery

* Core question:
    * **How does one service find another service?**
* **Core Idea:**
    * **Service Registry** holds all info about servers behind a service
    * **Registration**: When a service starts, it **registers** itself with the registry
        * ip, port, health, metadata, version (sometimes)
    * **Discovery:** When a **client** wants to use a service, it first **queries the registry** for the server list
    * The registry can auto-remove servers via health checks
* Client- vs Server-side discovery:
    * **Client-side**: The client queries the registry for servers behind a service, and chooses one.
        * Can do load-balancing.
    * **Server-side**: The client simply calls into a load-balancer, which forward the request to servers
        * The load-balancer queries the registry for server list
        * Common in API gateways and service meshes
* **Health Checks**
    * Registry periodically checks `GET /health` on servers
    * Removes any that fail, to maintain service uptime
* **Heartbeats**
    * A type of health check. Just checks if the server responds.
* **DNS-based discovery**
    * A very simply type of discovery
    * Just put endpoints in an A record. Client queries DNS for list of IP addresses
    * Con: DNS is slow to update and can be cached
* **Kubernetes**
    * Applications don't use IPs, they use service names, eg: `user-service.default.svc.cluster.local`
    * K8s tracks pods, updates endpoints, load-balances, and removes failed pods
    * TODO: learn more about k8s resources like Services, etc
* **Service Meshes:**
    * Application talks to a local service proxy (eg: Envoy sidecar)
    * Proxy handles service discovery, load-balancing, retries, TLS, observability
    * Basically, the **local proxy handles all complexity around client-service communication**
* **Popular technologies:**
    * **Kubernetes**: service-discovery, DNS-based, most common for containerized workloads
    * **Consul**: service registry, health checks, configuration, DNS interface
    * **ZooKeeper**: found in older systems
    * **etcd**: distributed key-value store. Used by k8s.
* Common interview Qs:
    * Why not hardcode IPs?
        * Because servers can fail, and scaling in/out requires code/config change
    * What if the service registry fails?
        * Want to run multiple registries / registry cluster
        * Clients / sidecars can cache results for later
    * Should clients cache results
        * Yes, for some short TTL, to protect the registry
        * You want a short TTL in case the data changes
    * What's the relationship between service discovery and DNS
        * Service discovery is about discovering the servers behind a service
        * DNS is about discovering the servers / IP addresses behind a name
        * DNS can be used to implement simple service discovery, but:
            * It can't do health checks
            * It can be slow to update, and results are often cached
            * It was not built for modern service discovery requirements

---

## Caching

* **Core question:**
    * **Can we avoid doing expensive work more than once?**
    * Trade memory for lower latency and system load
* Simple stuff:
    * Why we need caching: to reduce load, improve response time
    * Cache hit vs miss. On miss, query backend and cache result
    * TTL \= time before data item is flushed from cache, to keep it fresh
* **Cache patterns:**
    * **Cache Aside (Lazy Loading)**: query cache; on miss, query backend and store result; then return result to client. Very common.
    * **Write Through Cache:** every write (eg. to database) is also stored in the cache
    * **Write Back / Write Behind:** reads and writes always go to cache. Cache flushes to DB periodically. Dangerous if cache crashes.
    * **Write Around**: writes go directly to DB. On read, update cache. Useful when writes are frequent and reads are rare.
* **Cache Eviction Strategies**
    * **LRU (Least Recently Used)**: get rid of items accessed longest ago.
        * Very common and simple to implement
        * But a flood of requests can cause evictions of frequently-used data
    * **LFU (Least Frequently Used)**: get rid of the item infrequently accessed.
        * Avoids problems from large request spikes / scans
        * But harder to implement. Also, new items can be evicted quickly.
    * **FIFO:** remove the oldest entry. Simple but not as effective.
* **Cache invalidation**
    * **How do we keep the cache in sync with the backend/DB when data changes?**
    * Some methods:
        * Delete cache entry after every write
        * Update cache on writes
        * Use TTL
        * Version numbers
        * Event-driven invalidation
* **Cache Stampede / Thundering Heard**
    * Hot item is invalidated. Then 1000 servers all query the DB for the data.
    * Maybe a **retry-storm** is triggered
    * Mitigations:
        * **Request coalescing** (one request reloads data while the others wait)
        * **Distributed locks:** only one client / worker gets the lock and is able to update the cache
        * **Refresh popular items before expiry**
        * **Add random jitter to TTLs (and also to requests)**
        * **Retry jitter \+ exponential backoff**
* **Cache penetration / negative results**
    * An attacker could repeatedly ask for a data item that doesn't exist
    * It's not found in the cache, so the DB is queried
    * Mitigation: cache **negative results** so the cache can return "not found"
* **Cache overload**
    * Imagine if a million cache entries all expired at the same time
    * The DB could get crushed
    * Mitigation: Add random jitter to TTL of each item
* **Distributed Cache**
    * Using multiple servers in a single logical cache (eg: Redis, memcached)
    * Use **consistent hashing** to distribute keys among nodes
* **Multi-level caching**
    * Large systems often have multiple layers of caching
    * Eg: browser cache, CDN, API cache, Redis, DB
* **What should you cache?**
    * Anything that doesn't change often, and is accessed frequently or expensive to compute
    * user profiles, product catalogs, search results, configuration, feature flags, recommendation results, rendered pages, session data, expensive computation
    * NOT: rapidly changing data, one-time operations
* **Measuring effectiveness**
    * Look at the **cache hit rate**

---

## Event-Driven Systems

* **Core Idea:**
    * **Instead of one service directly telling another service what to do, a service publishes an event saying what happened.**
    * Other systems decide if they want to take action
    * Systems become **loosely coupled, more scalable, easier to extend**
* **Core problem:**
    * In standard system, one service needs to know about many downstream services
    * Adding/removing a downstream service requires changes to the first service
* **Instead:**
    * Customer interacts with first service (eg: Order service)
    * Order service published event on **event bus**
    * Event is consumed by downstream services (payment, inventory, shipping, etc)
    * Order service doesn't know about any of these\!
* **What is an event?**
    * Should be **a fact that something happened**
        * Eg: OrderPlaced, PaymentReceived
    * NOT a command:
        * Eg: ChargeCreditCard, ShipPackage (bad\!)
    * **Events should not be commands**
* **Components**
    * **Event Producer**: produces and sends the event
    * **Event Broker**: manages storing and routing of events
        * Examples: Kafka, RabbitMQ, Amazon SQS / SNS
    * **Event Consumer**: consumes events and may take action
* **Loose Coupling**
    * This is the **biggest benefit**
* **Asynchronous Communication**
    * An event can be received **in parallel** by each consumer
    * We don't need to make a series of requests
    * This **can improve latency and throughput**
    * Many consumers can act on the same event. This is called **fan out.**
* **Event ordering**
    * Different systems have different event-ordering guarantees
        * Eg: kafka guarantees order within a partition
* **Delivery guarantees**
    * **At least once**	\- Most common. Want **idempotency**.
    * **At most once**	\- Less common. Messages can be lost.
    * **Exactly once**	\- Strongest but very hard to implement
* **Other things (see Message Queues)**
    * Eventual consistency
    * Retries \- broker can redeliver non-acked messages
    * Dead Letter Queue
* **Event Schema**
    * The schema of the event.
    * Typically versioned, backwards compatible. Use schema registries.
* **Schema Registry**
    * Stores schema for use in an event-driven system
    * Producer looks up schema by ID, sends ID along with message
    * Broker forwards messages without looking at them
    * Consumer reads schema ID, gets schema from registry, and uses schema to decode rest of message
    * The Schema Registry can enforce **backward compatibility** to prevent breaking changes
* **Common interview questions**
    * Why use events instead of REST
        * REST is synchronous and services are highly coupled. It is relatively simple.
        * Events are async and loosely coupled. A bit more complicated. But very scalable and resilient
    * When should you NOT use events?
        * When you need immediate answers (not async)
        * eg: need to do authentication before any other user actions
    * Can consumers fall behind
        * Absolutely, if publish rate becomes to large and queue fills
        * This creates **consumer lag**. Mitigation: add more consumers, apply backpressure, load shedding

---

## Kubernetes Basics

* What it is:
    * Container orchestration service
    * Manages workloads, self healing, load balancing, service discovery, networking, storage, deployments
* Super high level:
    * **Pods** are the units that run your application.
    * **Deployments** maintain the desired number of pods and enable rolling updates.
    * **Services** provide stable networking and load balancing to pods.
    * **Autoscaling** adjusts capacity as demand changes.
    * **Self-healing** replaces failed pods automatically.
    * **Scheduling** spreads workloads across nodes while respecting resource constraints.
    * **Rolling deployments** reduce downtime during releases.
    * **Kubernetes orchestrates containers**—it is not your app itself, but the platform that keeps it running reliably.

---

# To Study...

List:
* Review SQL Basics
* See here for more: [Hello Interview \- System Design in a Hurry](https://www.hellointerview.com/learn/system-design/in-a-hurry/introduction)

More:
* [Hello Interview \- System Design in a Hurry](https://www.hellointerview.com/learn/system-design/in-a-hurry/introduction)
* Websockets & real-time updates
