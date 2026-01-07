# 16. Microservices. Microservice Architecture (MSA)

## Introduction

- **Microservices** = many small services that do one job each. They talk to each other via HTTP or through a Gateway.
- Each service runs separately (in its own container), can be updated and deployed independently from others.
- **API Gateway** = single entry point that sends requests to the right service.
- Microservice architecture is a distributed system.
- **Pros:**
  - **Scalability** - each one scales independently based on traffic and load
  - **Flexibility** - they can use different languages and DBs
  - **Resilience** - AKA fault tolerance. Failure in one does not crash the entire system.
  - **Speed of development** - Each team can focus on it's own microservice.
- **Cons:**
  - **Complexity** - Distributed system requires handling service discovery, communication, and orchestration
  - **Data consistency** - more challenging to maintain transactional integrity across services.
  - **Deployment overhead** - Needs advanced CI/CD, logging, monitoring setup
  - **Latency** - Inter-service communication can be slower than in monolith.

## Load balancing

Used in backend applications to distribute incoming traffic evenly across multiple servers.

- **Main algorithms:**
  - **Round Robin:** Even rotation: → Server A → B → C → then back to A...  
    Great for evenly spread traffic if all servers are equal.
  - **Least Connections:** Look at which server is least busy → send request there.
  - **IP Hash:** aka Sticky Session. Use client's hash function to always route them to the same server.
    Useful when session data is stored in the server’s memory and sticky sessions are needed.
  - **Health Checks:** Load balancer pings servers. If one is down → no traffic goes to it until it recovers.

## Concurrency

- **amqplib** - Node.js lib used to talk to RabbitMQ
- **cluster** - Node.js built-in module that lets use spawn multiple NodeJS processes.
- **amqplib + cluster** = parallel processing. Load balancing is handled automatically by NodeJS.
- **Nginx** - used for load balancing. Supports all 3: Round Robin, Least Connections, IP Hash.
- **Async Saga** - Asynchronous Saga pattern means compensating actions that happen asynchronously in a distributed system to undo some actions.
  Example:  
  Place order → charge → reserve product → send notification.  
  If reserving the product fails, the saga rolls back:  
  ✖ cancel notification, ✖ refund, ✖ cancel order.
- **Idempotency of microservices** - Re-executing a transaction doesn't alter the system beyond the first execution.  
   Even if the same message is processed twice, the system state remains the same.
- **Limit concurrency** - It's necessary to limit the number of messages being processed simultaneously.
  - Set up acknowledgements in RabbitMQ (only mark messages as "done" when acknowledgement is received from consumer).
  - Prefetch Size. Limit a number of messages consumer can process so that RabbitMQ does not flood app with more messages
    than it can handle
- **Dead letter exchange (DLX)** - If a message fails (rejected or expired), RabbitMQ can move it to a DLX.  
  Then it can retry later (with a delay), or log it for debugging.

## gRPC

**RPC** - Remote Procedure Call.  
A fast way for one service to call a function in another service, as if it was local — but over the network.  
“Hey BookingService, run calculatePrice() and send me the result.”

**Advantages:**

- **HTTP/2** for speed (better than HTTP/1.1 used by REST)
  - Simple and fast bi-directional data streaming.
- **Protobuf** instead of JSON (smaller, faster, strongly typed)
  - Binary format instead of text. Transmits and compresses faster.
- **Supports many Programming Languages** In a proto file, we specify what's on input, output, and get protoc code generation for any language.
- **Deadlines/Timeouts:** gRPC allows setting timeouts for each call.
- **Request Cancellation:** gRPC supports client-side request cancellation.

**Consul** - Service discovery and configuration tool. Consul is a binary standalone app that runs on its own machine.  
Services register themselves in Consul to discover who is running and where
It allows for:

- Registering services.
- Discovering services.
- Health checking.
- Optionally: key-value store, DNS or HTTP APIs

**Protobuf**
A binary format, easy to compress efficiently, strict typing. (ALL OF THIS AS OPPOSED to JSON, where you can pass different data types in one field in different cases).  
In case of Protobuf the server must first serialize the data into Protobuf, and the client must deserialize it.

Protobuf in practice:

- Define a service in .proto file (API contract)
- Generate code (client and server stubs)
- Implement service in code
- Implement client in code

**Important note on .proto**

- `.proto` file defines a strict schema
- Must live in a shared repo or central registry
- Client code should be auto-generated, not handwritten

**.proto file**:

```proto
syntax = "proto3";

package flights;

message FlightSearchRequest {
  string fromLocation = 1;
  string toLocation = 2;
}

message FlightSearchResponse {
  repeated string flightNumbers = 1;
}

service FlightService {
  rpc SearchFlights(FlightSearchRequest) returns (FlightSearchResponse);
}
```

**Real-life implementations**

- **Search Service + Schedule Service**

  - Real-time search with fresh schedule data
  - Fast responses
  - Could use server-streaming to get a stream of results.

- **Booking Service + Pricing Service** .
  - Up-to date price is crucial when booking.
  - No need for RabbitMQ because we need instant price update

```
Client (frontend)
  ↓
API Gateway / BFF
  ↓
gRPC call to Booking Service
  ↓
gRPC call to Pricing Service
  ↓
Booking response sent back
```

### 🛡️ Why gRPC is internal-only

- Uses _binary format_ (hard for browsers to consume)
- No _CORS_, _caching_, _auth_ or _versioning_ built-in
- Technically possible to expose externally (via gRPC-Web, Envoy), but **bad practice**

## Interaction between microservices

1. Synchronous call via _HTTP_.
   (_Simple_, but results in _tight compling_.)

2. _Message queue_ (e.g., `RabbitMQ`, `Kafka`).  
   (Almost always preferred nowdays. _Loose coupling_, but _eventual consistency_)

3. _gRPC_
   (_Fastest_, uses `HTTP2`, but requieres setup and strict `.proto` schema)

## Security in MSA

- Using authentication and authorization at the API Gateway level.
- Network policies for isolating services from each other.  
   Meaning explicitly saying which servers are allowed to talk.  
   Usually set-up at the infrastructure level (K8s: NetworkPolicy YAML files, Custom Docker networks + firewall rules)
- Encrypting traffic between services (TLS).  
   Meaning enforcing encrypted traffic (HTTPS) or mTLS (mutual TLS) where services identify each other's certificates

## Transactionality in MSA

**Transactionality** means doing a group of operations where either all succeed or none do.

In monolith it is easy - do a transaction / rollback a transaction.  
in MSA more complicated because it is a distributed system.

- Two-Phase Commit (2PC) - old fashioned way.
- Saga Pattern - break down transaction into steps. When one fails, undo them one by one.

## Microservice versions

Semantic versioning - system of rules that defines how microservice versions are numbered.
1.2.3 - major version, minor version, patch

- Major version - introduces breaking changes
- Minor version - new functionality
- Patch - bug fixes

## Service dictionary

Service Discovery - helps services find each other in a dynamic environment where hosts and ports can frequently change.  
Tools like **Consul** or **Kubernetes DNS** can be used for this purpose.

## Reverse Proxy

**Reverse proxy:** (Like Nginx or HAProxy) usually sits right in front of everything. It's the first thing the client hits.
**TLS termination** - the place where encrypted HTTPS traffic is decrypted.

```
Client
↓
Reverse Proxy ← (TLS termination happens here)
↓
API Gateway.
1. BFF (For single client. For instance BFF for web)
2. Apollo as BFF (When using multiple clients. For instance Web, Mobile etc.)
↓
BFF or Microservices
```

## Microservice Design

- **Microservice:** Independent module responsible for a specific functionality. Operates independently, has it's own DB.
- **API Contract:** Clear contracts for communication between microservices should be defined. REST, GraphQL, or gRPC are popular approaches for creating API.
- **Separation of responsibilities:** Each microservice should have a clearly defined responsibility and should not handle functionalities of other services.
- **Scalability and fault tolerance**
  - Microservices allow for horizontal scaling, making them suitable for large systems.  
    Horizontal - means more spawning more instances.
  - Clustering - (In NodeJS) run multiple processes of the same service on different CPU cores.
  - Load Balancing - Using a reverse proxy (Nginx or HAProxy). Distribute requests among different instances of the microservice.  
    Reverse proxy intercepts requests and redirects them to available microservice entities thus ensuring load balancing.
- **Security and Authorization**
  - **Transport Encryption (TLS/SSL):** Using HTTPS to ensure data encryption during transmission between microservices.  
    Applying certificates and mutual authentication to verify the authenticity of microservices.
  - **Authentication:** (Who you are) Identifying microservices to each other through secret keys or certificates.  
    Using a centralized Identity Provider (IdP), such as OAuth2 or OpenID Connect, for microservices authentication.
  - **Authorization:** (What you can access) - Defining and controlling access rights of microservices to resources and other microservices.  
    Utilizing Role-Based Access Control (RBAC) or other models for managing permissions.
  - **API Gateway:** Using an API Gateway as a central entry point, providing access management and security at the API level.  
    The API Gateway can perform authentication, authorization, rate limiting, and other security tasks.
  - **Network Policies:** Applying network policies and firewall to control and limit interactions between microservices.  
    In Kubernetes, this can be done using Network Policies YAML.
- **Logging and Monitoring:** The system should include centralized logging and monitoring of all microservices. ELK stack
- **Orchestration and Containerization:** Using Docker, Kubernetes, and other tools for managing and deploying microservices.
- **Testing:** Integration and unit testing are crucial for ensuring the quality of microservices.

> 💡 **OAuth2** - protocol for allowing services to act on behalf of a user. Used for Authorization. Example: Log in with Google.

> 💡 **OpenID Connect (OIDC)** - built on top of OAuth2. Used for Authentication. Returns user credentials like name, email.

> 💡 **Role-Base Access Control (RBAC)** - restrict access based on user roles. Example: 'admin' - full access, 'user' - read only.

> 💡 **Network Policies** - K8S YAML file that defines "Who can talk to whom over the network".

## CAP Theorem

One of the most fundamental concepts in distributed systems (like MSA).  
Distributed system can only **guarantee two of three** things at the same time:

| Term                   | What it means (simple)                                               |
| ---------------------- | -------------------------------------------------------------------- |
| ✅ Consistency         | All nodes **see the same data** at the same time.                    |
| ✅ Availability        | The system **always responds**, even if it’s not the freshest data.  |
| ✅ Partition Tolerance | The system **still works** even if one server can’t talk to another. |

You need **Partition tolerance** in MSA anyway, so you choose between **C**, **A**.

> 💡 Network partition in this case is **flaky connection** type of situation. We are not talking _no internet_ at all or _airplane mode_

**Examples**

- **PostgreSQL** - 100% **CP** because when network partition happens, we prioritise data consistency. Better fail than risk inconsistent results (multiple users book same room, triple payment withdrawal)
- **Group chat** - **CA** because it is better to get messages out of order / duplicates than just freeze UI
- **Redis** - CP
- **Cassandra** - AP (always available)

## Distributed Transactions and Saga

A _Saga_ breaks a big operation (that touches multiple services) into smaller local transactions, one per service.

- If everything goes well → all local changes are committed.
- If something fails → previous services run compensating transactions to undo their part.

**Choreography vs Orchestration**

- **Choreography**: (Kafka, RabbitMQ) Each service just reacts to events and sends out new events — like dancers following the rhythm without a central controller.
- **Orchestration**: (Temporal, Zeebe, AWS Step Functions) A central "orchestrator" tells services when to do what — like a conductor in a music orchestra.

> 💡 Choreography (event‑driven, decentralized coordination) is the more modern approach

**Technologies used**
In today’s microservices world, event‑driven choreography (Kafka, RabbitMQ) are default for simple sagas.

Choreography example:
Scenario: Order & Product Services

1. Client places an order.
2. `OrdersService` writes the order to DB (status = in process) and sends a message to RabbitMQ.
3. `ProductsService` listens for that message, checks stock, and updates DB (or not).
4. Sends back a success/failure message.
5. `OrdersService` updates the order accordingly or compensates (e.g. cancels the order).

**ProductsService**

```ts
// RabbitMQ client
const amqp = require("amqplib/callback_api");
// PostgreSQL client
const { Client } = require("pg");

// Connect to RabbitMQ
amqp.connect("amqp://localhost", (error0, connection) => {
  if (error0) throw error0;

  // open a virtual connection via RabbitMQ
  connection.createChannel(async (error1, channel) => {
    if (error1) throw error1;

    // declare a queue for listening to product availability
    channel.assertQueue("check-product-availability", { durable: false });

    // consume messages
    channel.consume(
      "check-product-availability",
      async (msg) => {
        // get orderId, productId
        const { orderId, productId } = JSON.parse(msg.content.toString());
        const client = new Client();
        await client.connect();

        // check products in stock
        const product = await client.query(
          "SELECT quantity FROM products WHERE id = $1",
          [productId]
        );

        // begin transaction
        await client.query("BEGIN");

        // if product available
        if (product.rows[0].quantity > 0) {
          //reduce the stock
          await client.query(
            "UPDATE products SET quantity = quantity - 1 WHERE id = $1",
            [productId]
          );

          try {
            // try send a message to a queue
            channel.sendToQueue(
              "product-availability",
              Buffer.from(
                JSON.stringify({ orderId, productId, available: true })
              )
            );

            // transaction commit
            await client.query("COMMIT");
          } catch (error) {
            // rollback a transaction in case of error
            await client.query("ROLLBACK");

            console.error("Error sending message, rollback", error);
          }

          // if product not available
        } else {
          // send back "sorry, not available"
          channel.sendToQueue(
            "product-availability",
            Buffer.from(
              JSON.stringify({ orderId, productId, available: false })
            )
          );
        }

        await client.end();
      },
      { noAck: true }
    );
  });
});
```

## Transactional Outbox

**Core problem:** How do I **reliably publish events** when my system changes state, **without losing or duplicating** events?

**Example:**
`FlightService` -- displays flight info to the frontend.  
`ScheduleService` -- keeps the flight schedule up to date.

`ScheduleService` --(RabbitMQ)--> `FlightService`.  
If `ScheduleService` updates local DB but the message is not sent, `FlightService` displays stale data.

**Transactional outbox** - Instead of trying to send the event right away, we save the event to the **outbox table** first, in the same transaction as the main data change.

- Later background worker picks up saved event
- Sends it to the message broker
- Guarantees delivery

Outbox step by step:

```
Step 1: UPDATE something in your DB
Step 2: INSERT a message about it into a table called "outbox"
(Both in the SAME DB transaction — atomic!)

Later:
Step 3: A background worker reads "outbox", sends the message
Step 4: Marks it as sent
```

> 💡 Background worker might be a simple NodeJS loop that checks outbox table once a few ms.

> 💡 Atomicity between A and B means that either both succeed or neither do.

**Example 1 ✈️**

1. A new price is set in the **Pricing Service**.
2. The **Pricing Service**:

- Updates the price in its local database.
- Writes a `PriceChanged` event to its **Outbox table** (in the same transaction).

3. A background process sends the event to the **message broker**.
4. **Booking Service** subscribes to these events.
5. On receiving the message, **Booking Service updates its local pricing info**.

**Example 2 💰**

1. An operator updates a flight time in the **Schedule Service**.
2. The **Schedule Service**:

- Updates its local database.
- **Adds an event to its Outbox table** inside the same DB transaction (e.g., `FlightScheduleChanged`).

3. A background worker reads unsent events from the Outbox and sends them to a **message broker** (e.g., RabbitMQ or Kafka).
4. **Search Service** is subscribed to this event topic/queue.
5. On receiving the message, **Search Service updates its cache** to reflect the new flight time.

### There are alternatives to it

Those solve the **same core problem**

### CDC (Change Data Capture)

**CDC** Idea - You **already write** to the database, why not **listen to DB changes** directly instead of writing an outbox table?

> 💡 WAL (Write-ahead-log) is an **append-only log** where a database records every change **before** it is applied to actual data, so changes can be recovered or replicated.

You need separate software for this to work, like `Debezium` for postgres

```
App → Postgres (write)
Postgres → WAL
Debezium → reads WAL
Debezium → publishes event
```

### Event sourcing

**Event Sourcing** is not just an alternative implementation, it is an **alternative architecture**

- Instead of saving the current **state**(e.g., "Balance = $500"),
- You save **every event** that led to that state (e.g., "Deposit $200", "Withdraw $50", etc).

Very hard to do, maintain. New event versions must support old event versions.

## Listen to yourself

This is basically no outbox strategy at all.  
Write DB, then send event. **Event might be lost !** if system crashes midway

## Transactional inbox

The goal of it is don't run the **business logic twice** on the same message

When consuming a message:

- Start DB transaction
- Check if `message_id` already exists in `inbox` table
- If yes → skip processing
- If no →
  - save `message_id` to `inbox_table`
  - execute business logic
- Commit DB transaction
- Then:
  - Kafka: commit offset
  - RabbitMQ - `ack`

Why this is useful for RabbitMQ as well

Scenario without inbox (problem):

- RabbitMQ delivers message `payment_id=123` to consumer
- Consumer runs DB update: `payments.status = "paid"` ✅
- Consumer calls external payment API and charges customer ✅
- Consumer **crashes before sending ack** ❌
- RabbitMQ thinks: “not acknowledged” → re-delivers the same message
- Consumer receives it again and charges customer again ❌❌

RabbitMQ did its job: it prevented message loss.
But you still got **duplicate business execution**

## CQRS

**CQRS** stands for Command Query Responsibility Segregation.

Reading data - **queries**
Writing / modifying data - **commands**

**✅ Pros:**

- **Performance:** Reads and writes often have different scaling and performance needs.
- **Complex business logic:** Write operations are complex, while reads are simple — mixing them complicates code.
- **Different storage needs:** You might want different databases for reads (fast) and writes (transactional and consistent).
- **Clearer code separation:** Easier to maintain and evolve each part independently.

**⚠️ Cons:**

- **More complexity:** (more moving parts)
- **Eventual consistency** (especially with asynchronous updates).
- Often paired with Event Sourcing, but doesn’t require it.

> 💡 **Eventual consistency** - data updates don't happen instantly everywhere, but all nodes will become consistent over time - eventually.

**Example:**
Let’s say we’re building a system for an airline company.  
We need to perform a flight / hotel booking.

**➕ Commands (Write actions)**

- Book a flight
- Cancel a flight
- Update seat availability
- Change flight time or price

Handled by Command services: `BookingService`, `ScheduleService`.

**🔍 Queries (Read actions)**

- Search flights
- View available seats

Handled by Query services: `SearchService`, `CustomerPortalService`.

**💽 Databases:**

- **Commands** use PostgreSQL or another strong RDBMS (ensures transactions).
- **Queries** use ElasticSearch or a read-optimized DB (faster for full-text search).

**🌊 Workflow:**

```plaintext
User makes change (Command)
        |
        v
[Command Service]
  - Writes to DB
  - Inserts event into Outbox
        |
        v
[Outbox Worker]
  - Reads unsent events from DB
  - Sends events (via gRPC or message broker)
        |
        v
[Query Service]
  - Receives event (via gRPC or broker)
  - Updates its query DB
```

## DDD (Domain-Driven Design)

**DDD** is about organizing your code to match the business logic, not just technical layers.

Instead of thinking “controllers → services → DB".  
**DDD** says:  
“Let’s model the real-world business domain clearly in code.

### DDD building blocks:

1. **Entity**

- Something **with id** that changes over time
- Examples: User, Flight, Booking

```ts
class User {
  constructor(id, name) {
    this.id = id;
    this.name = name;
  }
}
```

2. **Value Object**

- Defined only by their value
- No unique ID
- **Immutable**
- Examples: Money, Coordinates, DateRange

```ts
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }
}
```

3. **Aggregates**

```ts
class Order {
  constructor(id, items = []) {
    this.id = id;
    this.items = items;
  }

  addItem(item) {
    this.items.push(item);
  }
}
```

- Has **It's own id**
- A cluster of entities/value objects that are treated as a unit.
- Has a **single root** (aggregate root) that controls access.  
  `const order = new Order();`
- All changes must go through the root to maintain **data integrity**.  
  `order.addItem(item);` - correct  
  `order.items.push(item)` - incorrect. Outside the aggregate

4. **Repositories**

- **Only** works with Aggregates. Not with entities, not with value objects.
- **Hide DB logic** — provide access to aggregates/entities
- Think: "Give me Order 123", or "Save this User"

```ts
// suppose we have a User aggregate
export class User {
  constructor(public readonly id: string, public email: string) {}

  changeEmail(newEmail: string) {
    this.email = newEmail;
  }
}

// Repository interface in domain layer
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
}
```

5. **Domain Services**

- If logic **naturally belongs to aggregate** --> leave it there
- If logic **spans multiple aggregates** --> domain service

```ts
// User aggregate
export class User {
  constructor(public readonly id: string, public email: string) {}

  // belongs to User aggregate
  changeEmail(newEmail: string) {
    this.email = newEmail;
  }
}
```

```ts
export interface UserRepository {
  findByEmail(email: string): Promise<User | null>;
}

export interface PasswordHasher {
  verify(password: string, hash: string): Promise<boolean>;
}

export interface TokenService {
  issueToken(payload: { userId: string }): string;
}

// Authentication domain service
export class AuthenticationService {
  constructor(
    private readonly users: UserRepository,
    private readonly hasher: PasswordHasher,
    private readonly tokens: TokenService
  ) {}

  // Does a lot of things: user, hash, tokens, etc. Does not fit inth User aggregate
  async verifyUser(credentials: { email: string; password: string }) {
    const user = await this.users.findByEmail(credentials.email);

    const ok = await this.hasher.verify(credentials.password, user);

    const token = this.tokens.issueToken({ userId: user.id });

    return token;
  }
}
```

6. **Domain Events**

- Announce that something **just happened** in the domain.
- Allow **other parts of the system to react** (like microservices).

```ts
class UserRegisteredEvent {
  constructor(userId, timestamp) {
    this.userId = userId;
    this.timestamp = timestamp;
  }
}
```

7. **Bounded Contexts**

- A **boundary around a part of the system** where terms, logic, and models have a specific meaning
- Booking context, Pricing context etc.
- Microserivces **often map 1:1 to bounded context** but DDD does not enforce microservices.

> 🚧 Models/terms inside one context don’t leak into another.

### DDD Travel-tech service example

| **DDD Concept**     | **Airline Example**                        |
| ------------------- | ------------------------------------------ |
| **Entity**          | `User`, `Booking`, `Flight`                |
| **Value Object**    | `Money`, `FlightDate`, `AirportCode`       |
| **Aggregate**       | `Booking` with nested `FlightSegments`     |
| **Repository**      | `BookingRepository` to save/load bookings  |
| **Domain Service**  | `PaymentService`, `FareCalculationService` |
| **Domain Event**    | `FlightBookedEvent`, `PaymentFailedEvent`  |
| **Bounded Context** | `Booking`, `Search`, `Scheduling` services |

## Event-Driven Design (EDD)

**What it is (beginner view)**

- 📣 Services communicate by emitting events: _"Something happened"_ → others react.
- Instead of direct calls like "do X now", a service says _what happened_ and moves on.
- Example: `OrderCreated` event that **Payment** and _Notification_ services pick up.
- Ways to make EDD : _WebHooks_, _Queues_

**Core pieces (🧩)**

```bash
Product--(Event)--[Broker]--(Event)-->Consumer
```

### Pub / Sub (Publish / Subscribe)

_One_ event → delivered to _all interested subscribers_.

```css
[Order Service]  ──▶  [Broker: "order.created" topic]
                             ├──▶ [Email Service]
                             ├──▶ [Analytics Service]
                             └──▶ [Inventory Service]
```

### Work Queue (Competing Consumers)

_One_ task message → handled by _exactly one worker_.

```css
[Order Processor] ─▶ [Broker: "order.tasks" queue]
                          ├──▶ [Worker #1]
                          ├──▶ [Worker #2]
                          └──▶ [Worker #3]
```

### Pros and Cons

- ✅ Decouple teams/services; reduce tight, synchronous dependencies.
- ✅ Fan‑out work (notify many services)
- ✅ Smooth spikes (buffer in broker).
- ✅ Async workflows and eventual consistency are acceptable.
- ⚠️ Avoid for strict request/response or hard real‑time needs.

## Fallback (MSA)

Fallback in the context of MSA is a procedure applied when one of the microservices becomes unavailable, to ensure controlled system behavior and prevent complete stoppage.
A fallback can include:

- Returning predefined data (caching)
- Triggering an alternative microservice
- Returning an error with an informative message.

## Clean / N-Tier Architecture

**Both promote:**

- Separation of Concerns
- Keeping business logic isolated from infrastructure code
- Making code testable, scalable, and easy to change

**🧱 N-Tier Architecture (Traditional Layered)**

**Layers (Top-down)**

- **Presentation** UI / API (e.g., Express Controller, GraphQL Resolver)
- **Business Logic** Services, business rules
- **Data Access** DB calls, ORM (e.g., Sequelize, Prisma)

Data flow
`plaintext Client → Controller → Service → Repository → DB `

**Presentation** depends on **Business**,  
**Business depends** on **Data Access**.

🔁 Often ends up in tight coupling, and it’s not always easy to switch technologies.

**🧱 Clean Architecture (Onion/Circle)**

The outer layers depend on the inner layers — not the other way around.

**🧅 Layers (Inside-out)**

| **Layer**                | **What it contains**                           | **Depends on** |
| ------------------------ | ---------------------------------------------- | -------------- |
| **Entities**             | Core business models & rules (e.g., `Booking`) | None           |
| **Use Cases**            | Application logic / business workflows         | Entities       |
| **Interface Adapters**   | Controllers, Presenters, Gateways              | Use Cases      |
| **Frameworks & Drivers** | Express, DB, HTTP, external APIs               | Adapters       |

🔁 You can swap out Express for Fastify, or PostgreSQL for MongoDB, without touching core logic.

## Scalability and Availability (Fault TOLERANCE)

| **Term**            | **Meaning**                                                       |
| ------------------- | ----------------------------------------------------------------- |
| **Scalability**     | Can the system handle more load (traffic, data, users)?           |
| **Availability**    | Can the system stay online and responsive even if things fail?    |
| **Fault Tolerance** | Can parts of the system fail without taking the whole thing down? |

1. **Horizontal scaling**

- Add more machines / instances to share load.
- Use NGINX or HAProxy to balance load between them.

```plaintext
    Request
        |
        v
    Load Balancer
        |
        v
    App Instance 1
    App Instance 2
    App Instance 3
```

2. **Vertical Scaling**

- Upgrade one machine’s power (more RAM, CPU).
- **Example:** Upgrade an AWS EC2 from `t3.medium` to `m5.2xlarge`.

3. **Microservice Architecture**

- Break a monolith into focused services.
- **Example:** AuthService, OrderService, NotificationService

4. **Caching**

- Store frequently requested data in memory.
- **Example (Redis):** Slow DB query results, Session data, Product data

5. **Database Scaling**

- **Sharding:** Split data across DBs (e.g., by region)
- **Replication:** Read-only DB copies for scaling reads
- **Query optimization:** Indexes, denormalization, fewer joins

**Example:**

- MongoDB replica set = 1 primary + multiple read replicas

6. **Asynchronous Processing**

- Use background workers for long tasks. Faster response, smooth UI.
- Example: use RabbitMQ for email sending tasks.

7. **CDN (Content Delivery Network)**

- Distribute static content near the user.
- **Example:** serve js, css, fonts, images, vids
- Tools: **CloudFront**, **Akamai**

8. **Stateless Architecture**

- Each request is independent of others.
- **Example:** No session on server. Each request includes auth + payload

9. **Auto-Scaling**

- System adds/removes servers automatically.
- Efficient resource usage, handles spikes.
- **Example (AWS Auto Scaling):** add / remove more EC2 instances depending on load.

10. **Deployment & Rollbacks**

- Ensure minimal downtime and the ability to quickly roll back changes.
- **Example:** Using Docker and Kubernetes for deployment and scaling management.

### Circuit Breaker

A **Circuit Breaker** is a software mechanism that detects failures, counts them and stops calling a failing service until it starts working again.

`Service --> (Circuit Breaker) --> Resource`

- **Closed State**: No problem
- **Open State**: Problem. Number of errors exceeds a certain threshold within a set time period
- **Half-Open State**: After the "cooldown" time checks a resource health on a limited amount of requests then switches to either close or open again.

## Types of consistency

**Consistency models** - what a client can expect to see after reads and writes in a distributed system.

- **Strong consistency** - After a write completes, all reads see the latest value.  
  Slower, correctness prioritized.  
  Example: Banking apps, hotel booking

- **Eventual consistency** - After a write, reads may see old data, but eventually all nodes converge.  
  Faster responses, temporary inconsistency allowed.
  Example: social media likes

- **Weak consistency** - No guarantees about **when / if** your read will reflect the latest write
  Example: logs, metrics, logs

- **Read-your-own-writes** - Client always sees it's own writes while others not yet.
  Example: user updates facebook profile info and sees it, his friends currently not.

- **Monotonic reads** - Once a client has seen a value, it will never see an older value.
  Example: you've seen 10 messages in a chat, next read must be >=10 messages, not 5.

- **Casual consistency** - Cause and effect relationships are preserved
  Example: your friend commented on your facebook post, you replied. Noone should see a reply without your his message first

## Rollout strategies

This is about how you introduce changes into a **already running system**.  
Git workflow is the one we see on a daily basis

- **All-at-once** - huyak huyak i v prod. Srazu

- **Rolling update** - Say you have 5 servers, update them gradually 1 by 1.  
  No downtime + can stop rollout at any time

- **Blue / Green** - Blue - old version, Green - new version. Both run at the same time, you switch the traffic instantly back and forth on a load
  balancer level.

- **Canary** release - 1% of traffic, 5% of traffic, and then 100% eventually. Roll back if errors on some percentage.

- **Feature flags** - Hide new features with conditions in the code. If error, change condition to false, on the next request this logic is hidden. No redeploys.

- **Shadow traffic** - Production traffic duplicated from old `/v1` into `/v2` for our observation only. User does not see responses from `/v2`

- **A/B testing** - Split users by experiment. Say we have old and new search algorighm. Give 50% users the old one, 50% users the new one and observe metrics.

## Rate limiting

**Rate limiting** controls how much load a system accepts so it doesn’t collapse

Limits two things:

- How many requests per time (rate)
- How many requests at the same time

- **Fixed window** - Allow X requests **per minute** max. Counter changes every minute.  
  Unfair, have bursts at window edges.

- **Sliding window** - Allow X requests to happen in the **last 60 sec** at any moment.
  Fairer distribution, no bursts

- **Token bucket** (Super popular) - We have a bucket with tokens, those tokens refils at a constant rate, each request consumes 1 token.  
  Lets say we refil **10 tokens per second**, bucket **capacity is 50 tokens**

... dont understand how it differs from leaky, dont understand how it supports bursts, is there a separate rule for bursts ?

- **Leaky bucket** - we have a queue of requests. Those requests "leak-out" at a constant time.  
  Leak rate: **10 RPS**, requests come in, we fill the queue. If the queue is full, then drop requests.

- **Semaphore** (Concurrency limit, NOT exactly a rate limiting) - limits not per second, not per minute, but **at the same time**.  
  Max concurrent DB queries = 10, so if a new 11th arrives it must wait or fail.

## Protecting from third party errors

1. **Timeouts** (Always) - no request without timeout.
2. **Fail test** - if service is down, stop retying immediately
3. **Retries** - (Service is mostly healthy) so makes sense to retry
4. **Circuit breaker** - (Service is most certanly unhealthy) - Stop to retry, give the system time to recover.  
   Protects system from repeated failures and retry-storms.
5. **Fallbacks** - cached data, try later etc.
