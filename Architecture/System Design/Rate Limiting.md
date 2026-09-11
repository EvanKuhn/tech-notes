# Rate Limiting

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
