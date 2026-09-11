# Horizontal vs Vertical Scaling

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
