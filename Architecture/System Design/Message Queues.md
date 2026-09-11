# Message Queues

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
