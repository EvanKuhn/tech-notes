# Event-Driven Systems

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
