# API Design

* From [API Design for System Design Interviews](https://www.hellointerview.com/learn/system-design/core-concepts/api-design)
* **Common API Types:**
    * **REST** \- The default. Uses HTTP. Simple CRUD operations. Models *resources.*
    * **GraphQL** \- Invented by Facebook, lets client request exactly what it needs. More efficient.
    * **RPC** \- Efficient binary protocol used for internal inter-service comms. Models *procedures.*
* **Tips**:
    * Use REST 90% of the time
    * If you need real-time updates, you'll want something like **websockets**
    * Only spend \~5 minutes on API design in an interview
* **REST:**
    * **Resources** are the core entities. REST APIs operate on resources. These are **nouns.**
    * HTTP operators define the **verbs**
    * Path names should use the **plural** resource name.
    * Examples:
        * `GET /events`                    - Get all events
        * `GET /events/{id}`               - Get a specific event
        * `GET /venues/{id}`               - Get a specific venue
        * `GET /events/{id}/tickets`       - Get available tickets for an event
        * `POST /events/{id}/bookings`     - Create a new booking for an event
        * `GET /bookings/{id}`             - Get a specific booking
    * **Required data goes in the URL**
        * For example, the venue id is required to lookup the venue
    * **Optional data goes in the query params**
        * For example, optional filters when listing the set of venues, such as location.
    * For **relationships**, you have two options:
        * Nesting, eg: /events/{id}/tickets
        * Flat hierarchy, eg: /tickets?event\_id=123
    * **HTTP Methods:**
        * GET		Read resources
        * POST		Create a new resource. *Not idempotent\!*
        * PUT		Create or update a resource. Idempotent.
        * PATCH		Update existing resource. Idempotency depends on impl.
        * DELETE	Delete a resource
    * **Passing data:**
        * **Path params** to identify the particular resource(s)
        * **Query params** to affect filters, sorting, or other endpoint behavior
        * **Request body** contains data used to create or update a resource
    * **Returning data**
        * Done via **status code** and **response body**
        * Status codes:
            * 2xx \- Success
            * 3xx \- Extra action needed, like a redirect
            * 4xx \- Client error
            * 5xx \- Server error
        * Some common codes:
            * 400 Bad Request
            * 401 Unauthorized
            * 403 Forbidden
            * 404 Not Found
            * 405 Method Not Allowed
            * 409 Conflict
            * 429 Too Many Requests
            * 500 Internal Server Error
            * 504 Gateway Timeout
    * **Security:**
        * Done by securing each endpoint
        * Eg: Shopping app has access to `/products` but not `/employees`
* **GraphQL**
    * Created by FB in 2012 to solve inefficiencies with REST APIs
    * REST APIs often hit **N+1 queries:**
        * 1 query to list all resources
        * Plus N queries, one per resource
    * N+1 queries are very inefficient. GraphQL lets the client request *just the data they want*, resulting in fewer API calls and more a more responsive user experience.
    * Uses **a single endpoint per resource** and lets the user specify what data they want
    * Example schema and query:
        *   ```
            # SCHEMA

            type Event {
                id: ID!
                name: String!
                date: DateTime!
                venue: Venue!
                tickets: [Ticket!]!
            }

            type Venue {
                id: ID!
                name: String!
                address: String!
            }

            type Query {
                event(id: ID!): Event
                events(limit: Int, after: String): [Event!]!
            }


            # QUERY

            query {
                event(id: "123") {
                    name
                    date
                    venue {
                        name
                        address
                    }
                    tickets {
                        section
                        price
                        available
                    }
                }
            }
            ```
        * Notice how the relationships are defined in the schema
        * With a REST API, you'd need to make multiple requests for all of this data\!
    * **When to use GraphQL**
        * When your clients have diverse needs (eg: Web client needs different data than the mobile client)
        * GraphQL also allows for faster iteration; REST changes tend to rely on backend schema changes.
        * But normally, just use REST
    * **Security:**
        * Done at the **field-level** (different from REST)
* **RPC**
    * Remote Procedure Call
    * Allows one service to run a procedure on another service
    * Fast and efficient for internal inter-service communication
        * Uses binary encodings; no parsing of HTTP or JSON
        * Less data going across the network
    * **Not good for external-facing APIs**: too complex and requires coordinating software versions, etc
        * HTTP is better for public APIs, as everything knows HTTP
    * Operates on *procedures* instead of resources
        * Much more flexible. A procedure could do anything.
    * Popular RPC tech:
        * **gPRC,** which uses **protocol buffers**
        * Apache **Thrift**
    * **.proto files** are used by protocol buffers and extended by gRPC to include service definitions.
    * .proto file example:
        *   ```
            service TicketService {
                rpc GetEvent(GetEventRequest) returns (Event);
                rpc CreateBooking(CreateBookingRequest) returns (Booking);
                rpc GetAvailableTickets(GetTicketsRequest) returns (TicketList);
            }

            message GetEventRequest {
                string event_id = 1;
            }

            message Event {
                string id = 1;
                string name = 2;
                int64 date = 3;
                Venue venue = 4;
            }
            ```
        * Notice the `message` definitions define the objects
        * And the `service` contains a number of `rpc` procedures
        * .proto files are used by client and server to agree on the API, and thus the binary encoding
        * Type safety / type checking is also built in\!
    * **When to use RPC:**
        * For **internal** services
        * When **performance** is critical
        * For strong **type checking**
        * If you need **streaming** (supported by gRPC)
* **(REST) API Design Principles**
    * **Design around resources, not actions**
        * Nouns, not verbs\!
    * **Be consistent everywhere**
        * Naming conventions, parameter names, response shapes, etc
        * Consistency means users can easily predict how to use your API
    * **Follow the principle of least surprise**
        * Do what everyone else does. Eg: GET never writes data.
    * **Keep the API stateless**
        * The request should contain all necessary data
        * This allows for easy scaling behind the LB: just add more servers
    * **Make retries safe**
        * Failures can happen; want to allow clients to retry
        * Queries should be idempotent wherever possible
        * If non-idempotent methods (POST), use **idempotency keys**
    * **Paginate anything that can grow**
        * Assume any collection that *can* grow *will* grow, to beyond a page in length
        * Support pagination for all resource collections
        * Also set a cap on max page length to protect your service / app
    * **Secure every endpoint by default**
        * Make auth *opt-out* rather than *opt-in*
        * Every endpoint must require a security token by default
        * Only when explicitly marked public should an endpoint forego authentication
    * **Evolve without breaking clients**
        * When making changes, ensure backwards compatibility
        * Eg: Adding a new field is safe. Removing, renaming, or changing the meaning of an existing on is not
        * This is why API paths are often prefixed with a version, eg: `/v1/products/`, so we can create a new version
        * Example [versioning methods](https://redocly.com/blog/api-versioning-best-practices):
            * Path-based: `/api/v1/users` vs `/api/v2/users`
            * Param-based: `/api/users?version=2`
            * Header-based: `X-API-Version: 2.0`
    * **Make errors actionable**
        * On error, make sure the client (computer or human) can tell what went wrong, and what to do about it
        * Return 3 things:
            * The proper HTTP status code
            * Machine-readable error code, in body
            * Human-readable error message, in body
* **Common API Patterns**
    * **Pagination**
        * When returning a large set of results, we need to break up the set into pages
        * Done in two ways: offset-based and cursor-based. In either case we tell the server:
            * how many results we want per page
            * which page/offset to return
        * **Offset-based**
            * Simplest approach, most common
            * Tell server (a) how many results, and (b) at what offset (result number) to start
                * Eg: `GET /products?offset=20&limit=10`
            * Has **problems with large data sets**:
                * If a result was inserted between requests, it could be skipped, and/or cause duplicate results
                * This is because the **skip count** (number of initial records to skip) is recalculated with each request
        * **Cursor-based**
            * A **cursor** is a token returned to the client that references the next resource to start with
            * The client must pass that cursor on subsequent requests
            * Internally the cursor might reference a row / id in a database table somewhere
            * **Prevents missing or duplicate results** from being returned
                * This is because the API doesn't recalculate the resources to skip with each request.
                * Rather, it just picks back up from where it left off, via the cursor
            * Security tip: it is highly encouraged to **obfuscate** any internal implementation details so that an attacker cannot use the cursor for an attack.
                * Say for example that you want to use a database resource id as a cursor
                * You might encrypt the id via symmetric key encryption so only your servers can read the id
                * Or generate a UUID, cache `UUID => internal data` (short TTL), and fetch data on next request
    * **Filtering and Sorting**
        * Clients will frequently want to do this
        * **Use query parameters**
        * Eg: `/events?city=NYC&status=upcoming&sort=date`
        * Can also support sorting in reverse order via "minus" , eg: `sort=-date` (newest first)
        * Stay **consistent** across all endpoints
            * Mismatched names across endpoints is one of the most common consistency violations in real API reviews
    * **Idempotency Keys**
        * Used to support idempotency for non-idempotent request methods (eg. POST)
        * Idea:
            * Client generate idempotency key (token) and sends it with each request
            * Server stores the key along with each request so it can detect duplicate requests.
            * Stripe's API does this
            * Guessing it uses a cache in the backend to store these keys.
        * Example request:

| POST /events/123/bookings Idempotency-Key: 8e03978e-740c-4c2e-abd1-52f427d5c177 {   "tickets": \[{"section": "VIP", "quantity": 2}\] } |
| :---- |

    * **Consistent Error Responses**
        * Return 3 things: HTTP status code, error code, and error message
        * Eg:

| {   "error": {     "code": "SEAT\_UNAVAILABLE",     "message": "Section VIP has only 1 seat remaining"   } } |
| :---- |

        * The **error code** is for the application. These codes should not change
        * The **error message** is for humans and can change without breaking the app
    * **Versioning Strategies**
        * Path-based: `/api/v1/users` vs `/api/v2/users`
            * Simple, obvious to users, **preferred**
            * Allows for routing based on version
        * Param-based: `/api/users?version=2`
        * Header-based: `X-API-Version: 2.0`
* **Security Considerations**
    * Basic question: "Who is making the request? And are they allowed to do what they're asking?"
        * **Authentication**: Determines if the user is who they say they are
        * **Authorization**: Determines the permissions of the user
    * **API Keys**
        * Long, randomly-generated strings that act like passwords.
        * Must be included with each request.
        * Server generates keys, stores them in DB along with user, permissions, etc
        * **Good** for server-to-server communication, or to expose to 3rd-party devs
        * **Bad** for users: they shouldn't have to know about long tokens
        * Example request:
            	GET /events
            	Authorization: Bearer sk\_live\_abc123...
    * **JSON Web Tokens (JWT)**
        * Encode user information directly into the token itself
        * Server creates token with user id, permissions, expiration, and then cryptographically signs the token
        * Client will send the token with each request
        * Benefits:
            * No DB lookups; server can read info directly from token
            * Signed: any server with key can verify authenticity
        * Example token payload:
            *

| {   "user\_id": "123",   "email": "john@example.com",   "role": "customer",   "exp": 1640995200 } |
| :---- |

    * **Which to use?**
        * Use **API keys** for **internal service** communication and **external developer** access.
        * Use **JWT tokens** for **user sessions** in web and mobile applications.
    * **Role-Based Access Control (RBAC)**
        * System that assigns specific permissions on a user-level basis
        * On request, lookup permissions in the RBAC system before allowing action
        * Will require some sort of RBAC system, and a DB to hold the data
    * **Rate Limiting and Throttling**
        * Protects your system by preventing clients from sending too many requests too quickly
        * Common strategies:
            * **Per-user limits**: 1000 requests per hour per authenticated user
            * **Per-IP limits**: 100 requests per hour for unauthenticated requests
            * **Per-endpoint limits**: 10 booking attempts per minute to prevent ticket scalping
        * Typically done via **API gateway** or via application **middleware**
        * If rate limited, return `429 Too Many Requests`
