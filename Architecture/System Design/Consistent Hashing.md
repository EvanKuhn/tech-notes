# Consistent Hashing

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
