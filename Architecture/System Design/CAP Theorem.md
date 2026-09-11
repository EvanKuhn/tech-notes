# CAP Theorem

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
