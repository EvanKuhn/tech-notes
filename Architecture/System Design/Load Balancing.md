# Load Balancing

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
