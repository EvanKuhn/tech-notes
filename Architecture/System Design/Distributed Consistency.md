# Distributed Consistency

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
