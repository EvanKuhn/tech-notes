# Databases - SQL vs NoSQL

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
