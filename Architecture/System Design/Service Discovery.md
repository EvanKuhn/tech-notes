# Service Discovery

* Core question:
    * **How does one service find another service?**
* **Core Idea:**
    * **Service Registry** holds all info about servers behind a service
    * **Registration**: When a service starts, it **registers** itself with the registry
        * ip, port, health, metadata, version (sometimes)
    * **Discovery:** When a **client** wants to use a service, it first **queries the registry** for the server list
    * The registry can auto-remove servers via health checks
* Client- vs Server-side discovery:
    * **Client-side**: The client queries the registry for servers behind a service, and chooses one.
        * Can do load-balancing.
    * **Server-side**: The client simply calls into a load-balancer, which forward the request to servers
        * The load-balancer queries the registry for server list
        * Common in API gateways and service meshes
* **Health Checks**
    * Registry periodically checks `GET /health` on servers
    * Removes any that fail, to maintain service uptime
* **Heartbeats**
    * A type of health check. Just checks if the server responds.
* **DNS-based discovery**
    * A very simply type of discovery
    * Just put endpoints in an A record. Client queries DNS for list of IP addresses
    * Con: DNS is slow to update and can be cached
* **Kubernetes**
    * Applications don't use IPs, they use service names, eg: `user-service.default.svc.cluster.local`
    * K8s tracks pods, updates endpoints, load-balances, and removes failed pods
    * TODO: learn more about k8s resources like Services, etc
* **Service Meshes:**
    * Application talks to a local service proxy (eg: Envoy sidecar)
    * Proxy handles service discovery, load-balancing, retries, TLS, observability
    * Basically, the **local proxy handles all complexity around client-service communication**
* **Popular technologies:**
    * **Kubernetes**: service-discovery, DNS-based, most common for containerized workloads
    * **Consul**: service registry, health checks, configuration, DNS interface
    * **ZooKeeper**: found in older systems
    * **etcd**: distributed key-value store. Used by k8s.
* Common interview Qs:
    * Why not hardcode IPs?
        * Because servers can fail, and scaling in/out requires code/config change
    * What if the service registry fails?
        * Want to run multiple registries / registry cluster
        * Clients / sidecars can cache results for later
    * Should clients cache results
        * Yes, for some short TTL, to protect the registry
        * You want a short TTL in case the data changes
    * What's the relationship between service discovery and DNS
        * Service discovery is about discovering the servers behind a service
        * DNS is about discovering the servers / IP addresses behind a name
        * DNS can be used to implement simple service discovery, but:
            * It can't do health checks
            * It can be slow to update, and results are often cached
            * It was not built for modern service discovery requirements
