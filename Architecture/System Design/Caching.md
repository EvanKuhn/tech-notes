# Caching

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
