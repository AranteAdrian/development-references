# Architecture Patterns: A Thinking Model, Not a Menu

## The Confusion Nobody Talks About

Ask ten developers what "architecture patterns" means and you will get ten different answers. One says Microservices. Another says MVC. A third says Event-Driven. Someone else says Circuit Breaker. All of them are right — and that is exactly the problem.

The software industry has used the word "architecture" for over 20 years to describe things that answer completely different questions. Books like the Gang of Four's *Design Patterns* (1994) and Martin Fowler's *Patterns of Enterprise Application Architecture* (2002) laid incredible groundwork — but they also blurred the lines between patterns that operate at entirely different levels of a system.

The result? Developers watch tutorials, see stunning diagrams, and walk away confused — not because they are not smart enough, but because nobody told them that what they are looking at is **six different conversations happening at once**.

The shift starts when you stop treating architecture patterns as a menu where you pick one, and start treating them as a **thinking model** — a set of questions you ask at different zoom levels of a system. Each group of patterns answers a different question. None of them compete with each other. A system will always use patterns from every group simultaneously.

This document gives you that thinking model.

---

## How to Use This Document

Before reading further, internalize this principle:

> **Every time you encounter a pattern name, ask two questions first: what zoom level is this? and what decision does it answer?**

The six groups below are ordered intentionally — from the highest level of thinking (how the system exists in the world) down to the lowest level (how a single class is written). A good architect thinks top-down. A good developer reads bottom-up to understand context. Both directions are valid.

Each pattern includes a **FoodNow scenario** — a consistent food delivery platform used throughout the entire document. The same system is referenced in every group so that patterns can be compared against a familiar baseline rather than starting from scratch each time.

---

## Group 1 — Deployment: How Does the System Exist?

**The question this answers:** How do I physically structure and run my system? What are the boundaries of deployment?

This is the first decision in any system design because it shapes everything else. Before you think about how components communicate or how code is organized, you need to know what the deployable units are.

| Pattern | When to choose it | What you give up |
|---|---|---|
| **Monolith** | Small team, early stage, low complexity — deploy everything as one unit | Scales as one unit — cannot scale parts independently |
| **Modular Monolith** | Growing complexity but not ready to split — organize internally, deploy as one | Still one deployment — a bad deploy takes everything down |
| **Microservices** | Each part needs to scale, deploy, or fail independently | Operational complexity — networking, distributed tracing, service discovery |
| **Serverless** | Event-triggered, short-lived tasks — pay per execution, no idle cost | Cold start latency — first invocation after idle is slower |

**FoodNow scenario:**

> **Monolith** → FoodNow v1 at launch. Orders, payments, restaurants, delivery tracking, and user accounts all live in a single deployable Python app. One `app.py`, one database, one server. A developer can run the entire platform locally with one command. When lunch rush hits and CPU spikes — the whole application scales as one unit. Simple and fast to build.
>
> ```
> foodnow-app/
> ├── orders.py
> ├── payments.py
> ├── restaurants.py
> ├── delivery.py
> └── users.py
> → deployed as one unit → one server → one database
> ```

> **Modular Monolith** → FoodNow v2 after six months. The codebase grew messy — payment logic was mixed into order logic. The team reorganizes into modules with clear boundaries and no cross-module direct imports. Still one deployment, but now each module owns its own logic and data access.
>
> ```
> foodnow-app/
> ├── modules/
> │   ├── orders/       (owns order logic)
> │   ├── payments/     (owns payment logic)
> │   ├── restaurants/  (owns menu and scheduling)
> │   ├── delivery/     (owns tracking)
> │   └── users/        (owns auth and profiles)
> → still one deployment → one database → cleaner boundaries
> ```

> **Microservices** → FoodNow v3 when payment failures were crashing the order browsing experience. The team splits into independent services. Now a payment service outage does not affect a customer browsing the menu. Lunch rush → scale only the order service. No need to scale payments at the same rate.
>
> ```
> order-service     → deployed independently on its own container
> payment-service   → deployed independently, isolated failures
> restaurant-service→ deployed independently
> delivery-service  → deployed independently, scales with active orders
> user-service      → deployed independently
> → each service owns its own database → scales independently
> ```

> **Serverless** → FoodNow notifications. Every time an order status changes — placed, confirmed, picked up, delivered — a push notification must be sent. These are short, bursty, event-triggered tasks with no need for an always-on server. Each notification fires a function, runs for 200ms, and terminates. Zero cost between events.
>
> ```
> sendOrderConfirmationFn()  → triggers on OrderPlaced event
> sendPickupAlertFn()        → triggers on OrderPickedUp event
> sendDeliveryConfirmFn()    → triggers on OrderDelivered event
> generateInvoiceFn()        → triggers on OrderCompleted event
> → no idle cost → scales to zero between events
> ```

**Common confusion:** People compare Microservices to Event-Driven as if they are alternatives. They are not. Microservices is a deployment decision. Event-Driven is a communication decision (Group 2). You can have a Monolith that is Event-Driven. You can have Microservices that use REST. They live in different groups and answer different questions.

---

## Group 2 — Communication: How Do Components Talk?

**The question this answers:** When component A needs something from component B — does A wait for an answer, or does it fire and move on?

Once you know your deployment structure, the next question is how the pieces communicate. This decision has a massive impact on scalability, resilience, and user experience.

---

### Understanding the Hierarchy First

Before looking at the patterns, this group has a structure that trips people up:

```
Event-Driven is NOT a pattern at the same level as Message Queue and Pub/Sub.
Event-Driven is the UMBRELLA CONCEPT.
Message Queue and Pub/Sub are the specific HOW underneath it.
```

Think of it this way:

```
SYNCHRONOUS (caller waits)
└── REST

ASYNCHRONOUS / EVENT-DRIVEN (caller does not wait)
    Something happens → other things react → nobody is waiting for the result
    └── Message Queue   → one event, one receiver
    └── Pub/Sub         → one event, many receivers

PERSISTENT CONNECTION (live channel stays open)
└── WebSocket / SSE
```

**Event-Driven** is the situation: *something happens and other things need to react, but nobody is waiting for the result.* Message Queue and Pub/Sub are two different ways to implement that situation depending on how many receivers need to respond.

This is why you were right to sense a connection between them — they are connected. Event-Driven is the why. Message Queue and Pub/Sub are the how.

---

### The Patterns

| Pattern | Level | Core behaviour | When to choose it | What you give up |
|---|---|---|---|---|
| **REST** | Synchronous | Caller waits for response | User needs an immediate answer | Tight coupling — caller and receiver must both be available |
| **Event-Driven** | Async umbrella | Something happens, others react, nobody waits | Any time the caller does not need to wait for the result | Harder to debug — no direct call chain to trace |
| **↳ Message Queue** | Event-Driven impl. | One event, one receiver, guaranteed delivery | Task processing, job handoff — exactly-once processing required | Added infrastructure — queue must be managed and monitored |
| **↳ Pub/Sub** | Event-Driven impl. | One event, many receivers simultaneously | Fan-out — multiple systems need to react to the same event | Eventual consistency — subscribers may lag behind publisher |
| **WebSocket / SSE** | Persistent connection | Live channel stays open between two parties | Real-time updates, streaming responses, live tracking | Long-lived connections — resource cost at scale |

---

### The Decision Tree

```
Does the caller need to wait for the result right now?
│
├── YES → REST (synchronous)
│
└── NO → Event-Driven (asynchronous)
         │
         Does only ONE receiver need to handle it?
         │
         ├── YES → Message Queue
         │         (exactly-once, task processing)
         │
         └── NO  → Pub/Sub
                   (all subscribers get a copy, fan-out)

Is the connection ongoing and real-time?
└── WebSocket / SSE (persistent live channel)
```

---

### The Distinction Between Message Queue and Pub/Sub

This is the most commonly confused pair in Group 2 because both sit under the Event-Driven umbrella.

```
Message Queue                    Pub/Sub
─────────────────────────────    ─────────────────────────────
One sender → ONE receiver        One sender → ALL subscribers
Message consumed and deleted     Message copied to each subscriber
Worker competes for the message  Every subscriber gets their own copy
Used for: task distribution      Used for: event broadcasting
Guarantee: exactly-once          Guarantee: at-least-once per subscriber

Azure Service Bus Queue          Azure Event Grid / Service Bus Topic
```

The single deciding question: **should exactly one thing handle this, or should every interested party know about it?**

---

**FoodNow scenario:**

> **REST** → Customer taps "Place Order." The app sends `POST /orders` to the Order Service and waits. The response comes back: `200 OK { order_id: 1234, status: "confirmed" }`. The customer sees the confirmation screen. Both the app and the Order Service had to be available at the same time. The customer waited for the answer before anything else happened.

> **Event-Driven (the umbrella situation)** → The moment the order is confirmed, the system needs to react — but nobody is waiting. The Order Service does not call the Kitchen, Payment, or Rider services directly. It fires an event and moves on. Everything that needs to happen next happens asynchronously. This is the Event-Driven situation. Message Queue and Pub/Sub below are the specific tools used to implement it.

> **↳ Message Queue (Event-Driven implementation — one receiver)** → Payment processing. When a customer pays, a `ProcessPayment` job is placed into a queue. One payment worker picks it up, charges GCash, marks the job complete, and removes it from the queue. A second worker cannot pick up the same job. The payment is processed exactly once — no double charging.
>
> ```
> PaymentQueue → [ProcessPayment { order: 1234, amount: 350 }]
>     Worker A picks it up → charges GCash → removes from queue
>     Worker B sees nothing → moves to next job
> (exactly-once delivery guaranteed)
> ```

> **↳ Pub/Sub (Event-Driven implementation — many receivers)** → A restaurant marks itself as "temporarily closed." The Restaurant Service publishes a `RestaurantUnavailable` event to a topic. Three separate subscribers react independently: the Search Service hides the restaurant from results, the Order Service rejects new orders, and the Notification Service alerts affected customers. The Restaurant Service does not know any of them exist. All three receive their own copy of the same event simultaneously.
>
> ```
> RestaurantService publishes → RestaurantUnavailable { id: 7 }
>     SearchService subscribes    → hides from browse results
>     OrderService subscribes     → rejects new orders for id: 7
>     NotificationService subscribes → alerts pending customers
> (all three receive a copy — publisher knows none of them)
> ```

> **WebSocket / SSE** → The customer opens the "Track My Order" screen. A WebSocket connection opens between the app and the Delivery Service. Every 5 seconds, the rider's GPS coordinates are pushed to the customer's screen in real time. The rider moves on the map. The connection stays open until the order is marked delivered. No polling. No repeated requests. Just a live channel.

---

**The common confusion — cleared:**

> "Isn't Event-Driven just another name for Message Queue or Pub/Sub?"

No. Event-Driven is the situation. Message Queue and Pub/Sub are the tools. You can say a system is Event-Driven without specifying which tool it uses. Once you choose the tool, you choose Message Queue (one receiver) or Pub/Sub (many receivers) based on how many things need to react.

> "Can a system be Event-Driven and also use REST?"

Yes — absolutely. Cricket AI uses REST for the Flask frontend talking to the FastAPI backend (synchronous, user is waiting), and Event-Driven via Azure Event Grid and Service Bus for the blob-triggered pipeline (nobody waiting). Both communication patterns coexist in the same system serving different moments.



---

## Group 3 — Code Structure: How Is the Inside Organized?

**The question this answers:** Within a single service or application, how do I organize the code so it is maintainable, testable, and clear?

This group only applies inside a service boundary. It has nothing to do with how services talk to each other. A common mistake is mixing this with Group 1 — they live at completely different zoom levels.

| Pattern | Core idea | When to choose it | What you give up |
|---|---|---|---|
| **Layered** | Controller → Service → Repository — strict one-way dependency | Standard CRUD apps, simple internal logic | Rigid layers — a change in one layer ripples through all layers below |
| **MVC** | Model, View, Controller separation | UI-driven applications | View and controller still tightly coupled to each other |
| **Clean Architecture** | Business logic at the center, everything else depends on it | Complex domain, long-lived systems | More files, more abstractions — overhead for simple problems |
| **Hexagonal (Ports & Adapters)** | Core logic surrounded by swappable adapters | When you need to swap providers — databases, APIs, LLMs | More interfaces and indirection — harder to read for newcomers |
| **Domain-Driven Design (DDD)** | Organize code around business domains and bounded contexts | Large teams, complex business rules, microservices boundaries | Steep learning curve — requires deep domain understanding upfront |

**FoodNow scenario:**

> **Layered** → The FoodNow Notification Service. Simple job: receive an event, find the right template, send the message. Three clear layers and nothing more.
>
> ```
> NotificationController  → receives the event payload
>         ↓
> NotificationService     → selects template, formats message
>         ↓
> NotificationRepository  → reads template from database
>         ↓
> sends via SMS or push
>
> The controller never touches the database.
> The repository never formats messages.
> Each layer has exactly one job.
> ```

> **MVC** → The FoodNow Restaurant Portal. Restaurant owners log in to manage their menu, hours, and incoming orders through a web interface.
>
> ```
> Model      → MenuItem, Order, Restaurant (SQLAlchemy classes)
>              represents data and database relationships
>
> View       → menu_edit.html, order_list.html (Jinja2 templates)
>              renders the HTML the restaurant owner sees
>
> Controller → RestaurantBlueprint routes
>              handles HTTP requests, calls model, passes to view
>
> Owner adds a new menu item:
> Controller receives POST /menu/add
>     → creates MenuItem model object
>     → saves to database
>     → redirects to updated menu view
> ```

> **Clean Architecture** → The FoodNow Order Service. Complex rules: discount logic, fraud checks, restaurant availability, minimum order amounts, promo code validation. Business logic cannot be contaminated by FastAPI or database details.
>
> ```
> Core (inner ring — no external imports):
>     Order entity, OrderItem entity
>     PlaceOrderUseCase — all business rules live here
>     Discount rules, fraud checks, availability checks
>
> Adapters (outer ring — depends on core):
>     FastAPIOrderController → calls PlaceOrderUseCase
>     PostgresOrderRepository → implements OrderRepository port
>     GCashPaymentAdapter → implements PaymentPort
>
> PlaceOrderUseCase never imports FastAPI.
> PlaceOrderUseCase never imports SQLAlchemy.
> Business rules are testable in pure Python with no database.
> ```

> **Hexagonal (Ports and Adapters)** → The FoodNow Payment Service. Must support GCash, PayMaya, credit card, and COD — and new providers will be added. The core payment logic stays the same. Only the adapter changes per provider.
>
> ```
> PaymentPort (interface):
>     def charge(amount, customer, details) → PaymentResult
>     def refund(transaction_id, amount) → RefundResult
>
> Adapters (swappable):
>     GCashAdapter(PaymentPort)    → calls GCash API
>     PayMayaAdapter(PaymentPort)  → calls PayMaya API
>     CreditCardAdapter(PaymentPort)→ calls Stripe API
>     CashOnDeliveryAdapter(PaymentPort) → no external call
>
> Add a new payment provider → write one new adapter.
> Core payment logic never changes.
> ```

> **DDD** → FoodNow at scale. Three teams own three bounded contexts. Each has its own models, language, and database. They communicate through events, not shared tables.
>
> ```
> Ordering Context:
>     Entities: Order, OrderItem, Cart
>     Language: "place order", "cancel order", "order total"
>     Database: orders_db
>
> Restaurant Context:
>     Entities: MenuItem, Schedule, KitchenCapacity
>     Language: "activate item", "set hours", "mark unavailable"
>     Database: restaurant_db
>
> Delivery Context:
>     Entities: Rider, Route, TrackingEvent
>     Language: "assign rider", "pick up", "deliver"
>     Database: delivery_db
>
> Ordering Context never directly queries restaurant_db.
> It publishes events. Restaurant Context listens and reacts.
> ```

**The decision rule:** How complex is the internal logic, and how often do external dependencies change? Simple logic with stable dependencies → Layered. Complex logic with swappable externals → Hexagonal or Clean.

---

## Group 4 — Data: How Is State Managed?

**The question this answers:** How does the system read and write data? Are reads and writes the same operation, or do they need to be separated?

Data patterns are often the most overlooked group in system design conversations, yet they have the biggest impact on performance at scale.

| Pattern | Core idea | When to choose it | What you give up |
|---|---|---|---|
| **CRUD** | Same model for reading and writing | Simple apps where read and write needs are balanced | No separation — read performance degrades as write complexity grows |
| **Repository Pattern** | Abstract data access behind an interface | When you want to swap databases without changing business logic | Extra abstraction layer — more code for simple data access |
| **CQRS** | Separate models for reads and writes | Read traffic vastly exceeds write traffic; complex query requirements | Two models to maintain — sync between them must be managed |
| **Event Sourcing** | Store events, not final state — rebuild state by replaying events | Audit trails, financial systems, undo/redo requirements | Complexity of replaying events — querying current state is non-trivial |

**FoodNow scenario:**

> **CRUD** → FoodNow delivery location tracking. Every 5 seconds the rider's app sends GPS coordinates. The system writes the latest position. The customer reads the latest position. Both the read and write use the same `delivery_locations` table with the same columns. Simple. Direct. No history needed — only the current position matters.
>
> ```
> Table: delivery_locations
> | order_id | latitude | longitude | updated_at |
>
> Write: UPDATE delivery_locations SET lat=14.5, lng=121.0 WHERE order_id=1234
> Read:  SELECT lat, lng FROM delivery_locations WHERE order_id=1234
>
> Same table. Same model. Same shape. CRUD is exactly right here.
> ```

> **Repository Pattern** → FoodNow menu access. The Order Service needs to check if a menu item is available before confirming an order. It does not care whether the menu lives in PostgreSQL, Redis cache, or a future NoSQL store. It calls `MenuRepository.get_by_id(item_id)` and gets a `MenuItem` object back.
>
> ```python
> class MenuRepository:
>     def get_by_id(self, item_id: int) -> MenuItem: ...
>     def get_by_restaurant(self, restaurant_id: int) -> List[MenuItem]: ...
>     def save(self, item: MenuItem) -> MenuItem: ...
>
> # Order Service calls:
> item = menu_repo.get_by_id(42)
>
> # When team migrates from PostgreSQL to Redis for menu caching:
> # Only MenuRepository implementation changes.
> # Order Service code does not change at all.
> ```

> **CQRS** → FoodNow order history. Customers check their order history constantly — far more reads than writes. A new order is written once. It is read many times across different screens (history list, order detail, reorder suggestion). The write model is normalized for fast inserts. The read model is a denormalized view pre-joined with restaurant names and item details — optimized purely for querying.
>
> ```
> Write Model (normalized — fast inserts):
> orders table: order_id, customer_id, restaurant_id, total, status
> order_items table: item_id, order_id, name, qty, price
>
> Read Model (denormalized — fast queries):
> order_history_view:
> | order_id | restaurant_name | items_summary | total | date | status |
> (pre-joined, pre-formatted, ready to display)
>
> Customer opens order history → queries the read model only.
> Customer places new order → writes to the write model only.
> The two models are synced asynchronously.
> ```

> **Event Sourcing** → FoodNow payment transactions. Every payment event is stored in order — never updated, never deleted. The current payment state is derived by replaying the events. If a customer disputes a charge, the full sequence is auditable.
>
> ```
> payment_events table:
> | event_id | order_id | type                | data              | timestamp |
> | 1        | 1234     | PaymentInitiated    | {amount: 350}     | 12:00:01  |
> | 2        | 1234     | GCashRequestSent    | {ref: "abc123"}   | 12:00:02  |
> | 3        | 1234     | PaymentConfirmed    | {txn: "xyz789"}   | 12:00:04  |
> | 4        | 1234     | RefundRequested     | {reason: "cancel"}| 12:05:00  |
> | 5        | 1234     | RefundProcessed     | {amount: 350}     | 12:05:03  |
>
> Current status = replay all events for order 1234.
> Dispute raised = show the full sequence to the customer.
> ```

**The decision rule:** Are your read and write patterns the same? If yes, CRUD. If reads heavily outnumber writes or need different shapes, consider CQRS. If you need a full history of everything that happened, Event Sourcing.

---

## Group 5 — Failure Handling: What Happens When Things Break?

**The question this answers:** When an external dependency fails, slows down, or behaves unexpectedly — what does the system do?

This group is often skipped entirely by developers who are new to system design. It is the difference between a system that degrades gracefully and one that cascades into full failure.

| Pattern | What it protects against | How it works | What you give up |
|---|---|---|---|
| **Retry** | Transient failures — brief network blips | Automatically retry the call after a short delay | Risk of hammering a struggling service — use with backoff |
| **Circuit Breaker** | Slow or failing external services | Stop calling after N failures; try again after a cooldown | Must tune thresholds — too sensitive and it trips unnecessarily |
| **Bulkhead** | One failing dependency taking down everything | Isolate resources per dependency — failure stays contained | Resource overhead — dedicated pools per dependency |
| **Saga** | Multi-service transactions that can partially fail | Coordinate compensating transactions across services | Complexity — compensating logic for every step must be written |
| **Dead Letter Queue** | Messages that cannot be processed | Capture unprocessable messages for inspection and retry | Operational burden — someone must monitor and action the DLQ |

**FoodNow scenario:**

> **Retry** → FoodNow calls the GCash API to charge the customer. GCash returns a 503 (temporarily unavailable). The system does not immediately fail the payment. It retries with exponential backoff — waits 1 second, tries again, waits 2 seconds, tries again, waits 4 seconds, final attempt. If still failing after 3 retries, it reports the failure to the customer gracefully.
>
> ```
> Attempt 1 → GCash 503 → wait 1s
> Attempt 2 → GCash 503 → wait 2s
> Attempt 3 → GCash 503 → wait 4s
> Attempt 4 → GCash 503 → FAIL → "Payment temporarily unavailable"
>
> Without retry: customer gets error on the first blip.
> With retry: customer never notices a 1-second hiccup.
> ```

> **Circuit Breaker** → GCash experiences an outage during a major holiday sale. FoodNow's Payment Service keeps calling GCash — each call times out after 10 seconds, blocking the payment thread. After 5 consecutive failures, the Circuit Breaker trips. For the next 60 seconds, all GCash payment attempts return "GCash temporarily unavailable" immediately — no timeout wait. After 60 seconds, one test request goes through. If it succeeds, the circuit closes and normal processing resumes.
>
> ```
> Calls 1–5  → GCash timeout (10s each) → Circuit counts failures
> Call 6     → Circuit OPEN → instant response: "unavailable"
>              no actual call made to GCash
> [60 second cooldown]
> Test call  → GCash responds 200 → Circuit CLOSES
> Call 7     → GCash called normally
>
> Without circuit breaker: 10s timeout per user during outage.
> With circuit breaker: instant response, GCash gets breathing room.
> ```

> **Bulkhead** → FoodNow's Delivery Tracking Service gets overwhelmed during a major weekend surge. Without bulkheads, the tracking service's slow database queries exhaust the shared connection pool — now the Payment Service cannot get a database connection either. With bulkheads, each service has its own isolated connection pool. Tracking Service slowdown stays contained. Payment Service is completely unaffected.
>
> ```
> Without Bulkhead:
> Shared pool: 50 connections
> Tracking uses all 50 (slow queries during surge)
> Payment waits... waits... timeout → payment fails
>
> With Bulkhead:
> Tracking pool: 20 connections (isolated)
> Payment pool:  10 connections (isolated)
> Tracking exhausts its 20 → only tracking degrades
> Payment still has its 10 → payments continue normally
> ```

> **Saga** → A customer places a FoodNow order. The system must: create the order, reserve the restaurant slot, charge the customer, and assign a rider. These are four separate services. If payment fails at step 3 after the restaurant slot was already reserved, the system must undo the reservation. Each step has a compensating action.
>
> ```
> Step 1: CreateOrder       → success → OrderCreated
> Step 2: ReserveSlot       → success → SlotReserved
> Step 3: ChargeCustomer    → FAIL    → PaymentFailed
>
> Compensating actions trigger in reverse:
> Compensate Step 2: ReleaseSlot    → slot freed
> Compensate Step 1: CancelOrder    → order cancelled
> Notify customer: "Payment failed, order cancelled"
>
> Without Saga: restaurant slot remains locked, order in limbo.
> With Saga: every failure has a defined recovery path.
> ```

> **Dead Letter Queue** → FoodNow sends a push notification when the rider picks up the order. The customer's phone is off. The push notification fails. Without a DLQ, the notification is lost silently. With a DLQ, the failed notification goes into the dead letter queue. A background worker retries it every 5 minutes. After 3 failed push attempts, it falls back to SMS. Nothing is lost.
>
> ```
> NotificationQueue → send push to customer → phone offline → FAIL
>     → message moves to DeadLetterQueue
>
> DLQ Worker (runs every 5 minutes):
> Attempt 1 → push retry → phone still offline → back to DLQ
> Attempt 2 → push retry → phone still offline → back to DLQ
> Attempt 3 → fallback to SMS → delivered
>
> Customer gets the notification. Nothing was lost silently.
> ```

**The decision rule:** For every external call your system makes, ask — what happens if this call fails? Slow response? Times out permanently? That question maps directly to which pattern you need.

---

## Group 6 — Design Patterns: How Is This Class Written?

**The question this answers:** At the code level — how do I write this class, object, or behaviour cleanly so it is reusable, extensible, and clear?

This group is the Gang of Four territory. These patterns live at the lowest zoom level — inside a single class or small set of collaborating classes. They are not architecture in the deployment sense. They are architecture in the code craftsmanship sense.

### Creational — How are objects created?

| Pattern | When to use it |
|---|---|
| **Factory** | You need to create objects without exposing creation logic |
| **Singleton** | Only one instance should ever exist |
| **Builder** | Object construction is complex with many optional parameters |

**FoodNow scenario:**

> **Factory** → FoodNow supports different restaurant types: cloud kitchens (no physical address, delivery only), franchise branches (standardized menu, fixed pricing), and independent restaurants (custom menu, variable pricing). Each type behaves differently. The Factory decides which class to instantiate based on the type — the calling code never handles this logic.
>
> ```python
> class RestaurantFactory:
>     @staticmethod
>     def create(restaurant_type: str, config: dict):
>         if restaurant_type == "cloud_kitchen":
>             return CloudKitchen(config)
>         elif restaurant_type == "franchise":
>             return FranchiseRestaurant(config)
>         elif restaurant_type == "independent":
>             return IndependentRestaurant(config)
>
> # Calling code — never sees the creation logic:
> restaurant = RestaurantFactory.create("cloud_kitchen", config)
> ```

> **Singleton** → FoodNow's database connection pool. Creating a new database connection for every request is expensive. The connection pool must be created once and shared across all request handlers. No matter how many requests come in simultaneously, they all use the same single pool instance.
>
> ```python
> class DatabasePool:
>     _instance = None
>
>     @classmethod
>     def get_instance(cls):
>         if cls._instance is None:
>             cls._instance = create_pool(max_connections=20)
>         return cls._instance
>
> # Every service call uses the same pool:
> pool = DatabasePool.get_instance()  # always returns the same object
> ```

> **Builder** → FoodNow orders are complex. A customer can add items, apply a promo code, set a delivery address, choose a payment method, add special instructions, and request utensils. All optional. All combinable. The Builder assembles the final Order object step by step, validates at `.build()`, and rejects invalid combinations before the order is submitted.
>
> ```python
> order = (
>     OrderBuilder()
>     .customer(customer_id=42)
>     .restaurant(restaurant_id=7)
>     .add_item(item_id=101, quantity=2)
>     .add_item(item_id=205, quantity=1, note="no onions")
>     .promo("SAVE50")
>     .delivery_address("123 Rizal St, Cavite")
>     .payment("gcash")
>     .build()  # validates everything, raises error if invalid
> )
> ```

### Structural — How do objects fit together?

| Pattern | When to use it |
|---|---|
| **Adapter** | You need to make an incompatible interface work with your code |
| **Decorator** | You want to add behaviour to an object without changing its class |
| **Facade** | You want to simplify a complex subsystem behind a clean interface |

**FoodNow scenario:**

> **Adapter** → FoodNow needs to support GCash, PayMaya, and Stripe. Each has a completely different API — different endpoints, different request formats, different response shapes. The Adapter wraps each one behind a common `PaymentPort` interface. The Order Service calls the same interface regardless of provider. The adapter translates.
>
> ```python
> class PaymentPort:  # the common interface
>     def charge(self, amount, customer_id, details) -> PaymentResult: ...
>     def refund(self, transaction_id, amount) -> RefundResult: ...
>
> class GCashAdapter(PaymentPort):
>     def charge(self, amount, customer_id, details):
>         # translates to GCash's specific API format
>         return gcash_api.create_payment(
>             msisdn=details["phone"], amount=amount
>         )
>
> class StripeAdapter(PaymentPort):
>     def charge(self, amount, customer_id, details):
>         # translates to Stripe's completely different API
>         return stripe.PaymentIntent.create(
>             amount=amount * 100, currency="php"
>         )
>
> # Order Service never knows which adapter is active:
> payment_service.charge(350, customer_id, details)
> ```

> **Decorator** → FoodNow's order placement endpoint needs: authentication check, restaurant open-hours validation, minimum order amount check, and fraud scoring — before any order logic runs. Each check is a decorator stacked on the handler function. Add or remove a check without touching the core handler.
>
> ```python
> @require_authenticated_customer
> @validate_restaurant_open
> @enforce_minimum_order_amount
> @check_fraud_score
> def place_order(request: OrderRequest) -> OrderResponse:
>     # core order logic — clean, focused, no validation noise
>     return order_service.create(request)
>
> # Each decorator adds one responsibility.
> # Remove @check_fraud_score → fraud check gone, nothing else changes.
> # Add @validate_promo_code → just stack another decorator.
> ```

> **Facade** → The FoodNow `OrderFacade` is a single entry point for placing an order. Behind it, the facade coordinates five separate services — validation, inventory, payment, notification, and analytics. The mobile app calls one method. The complexity of coordination is completely hidden.
>
> ```python
> class OrderFacade:
>     def place_order(self, request: OrderRequest) -> OrderConfirmation:
>         # coordinates everything internally:
>         validated = self.validator.check(request)
>         slot = self.restaurant_service.reserve_slot(request.restaurant_id)
>         payment = self.payment_service.charge(request.payment)
>         order = self.order_service.create(validated, slot, payment)
>         self.notification_service.send_confirmation(order)
>         self.analytics_service.track(order)
>         return OrderConfirmation(order.id, order.status)
>
> # Mobile app calls:
> confirmation = order_facade.place_order(request)
> # That is all it sees. One method. Clean.
> ```

### Behavioral — How do objects communicate and behave?

| Pattern | When to use it |
|---|---|
| **Observer** | When one change needs to notify many dependents automatically |
| **Strategy** | When you want to swap algorithms or behaviours at runtime |
| **Command** | When you want to encapsulate a request as an object |

**FoodNow scenario:**

> **Observer** → FoodNow order status changes. When an order moves to "Delivered," four separate systems need to react: the customer gets a notification, the restaurant dashboard updates, the loyalty points service adds points, and the analytics tracker records the completion. Each is an observer. Adding a fifth observer — say a review prompt service — requires no change to the Order entity.
>
> ```python
> class Order:
>     def __init__(self):
>         self._observers = []
>
>     def add_observer(self, observer):
>         self._observers.append(observer)
>
>     def update_status(self, new_status):
>         self.status = new_status
>         for observer in self._observers:
>             observer.on_status_changed(self)
>
> # Observers registered at startup:
> order.add_observer(CustomerNotificationObserver())
> order.add_observer(RestaurantDashboardObserver())
> order.add_observer(LoyaltyPointsObserver())
> order.add_observer(AnalyticsObserver())
>
> # When status changes → all four react automatically:
> order.update_status("DELIVERED")
> ```

> **Strategy** → FoodNow rider assignment. The platform needs to assign a rider to every order. Different strategies apply in different contexts: nearest rider (fastest pickup), highest-rated rider (premium orders), load-balanced rider (fair distribution), or surge-area rider (during peak). Same interface. Swap the strategy at runtime without changing the assignment logic.
>
> ```python
> class RiderAssignmentService:
>     def __init__(self, strategy: RiderStrategy):
>         self.strategy = strategy
>
>     def assign(self, order) -> Rider:
>         return self.strategy.select_rider(order)
>
> # Swap strategy based on context:
> if order.is_premium:
>     service = RiderAssignmentService(HighestRatedRiderStrategy())
> elif is_peak_hour:
>     service = RiderAssignmentService(LoadBalancedStrategy())
> else:
>     service = RiderAssignmentService(NearestRiderStrategy())
>
> rider = service.assign(order)
> ```

> **Command** → FoodNow order placement encapsulated as a command object. The command holds all data needed to place the order. It can be logged before execution (for audit), queued during high load, cancelled if the customer changes their mind, or replayed if something goes wrong during processing.
>
> ```python
> @dataclass
> class PlaceOrderCommand:
>     customer_id: int
>     restaurant_id: int
>     items: List[OrderItem]
>     payment_method: str
>     promo_code: Optional[str]
>     delivery_address: str
>     timestamp: datetime
>
> class OrderCommandHandler:
>     def handle(self, command: PlaceOrderCommand) -> OrderResult:
>         # log the command first — full audit trail
>         self.audit_log.record(command)
>         # execute the command
>         return self.order_service.create_from_command(command)
>
> # Caller creates command → handler executes:
> cmd = PlaceOrderCommand(customer_id=42, restaurant_id=7, ...)
> result = handler.handle(cmd)
>
> # Same command can be replayed if the first attempt failed.
> # Same command structure used in the audit log.
> ```

---

## The Complete Thinking Model — All Six Questions

When approaching any system design problem, work through these questions in order:

```
1. DEPLOYMENT     → What are my deployable units?
                    Monolith / Modular / Microservices / Serverless

2. COMMUNICATION  → How do components talk to each other?
                    REST / Event-Driven / Queue / Pub-Sub / Streaming

3. CODE STRUCTURE → How is the inside of each service organized?
                    Layered / Hexagonal / Clean / DDD / MVC

4. DATA           → How does the system read and write state?
                    CRUD / Repository / CQRS / Event Sourcing

5. FAILURE        → What happens when dependencies break?
                    Retry / Circuit Breaker / Bulkhead / Saga / DLQ

6. CODE DESIGN    → How are individual classes and objects written?
                    Factory / Observer / Strategy / Adapter / etc.
```

These six questions are not alternatives. They are layers. Every production system answers all six simultaneously — the difference between a junior developer and a senior architect is knowing which question each pattern belongs to, and being able to articulate why a specific answer was chosen at each layer.

---

## Where Patterns Live in Architecture Documentation

One of the most practical questions when producing architecture documentation is: which patterns belong in which document? The C4 Model and the HLD/LLD split give you the answer.

### Mapping to C4 Model Levels

The C4 Model has four levels of zoom. Each level makes different patterns visible.

| C4 Level | Audience | Patterns visible at this level |
|---|---|---|
| **L1 — System Context** | Stakeholders, non-technical | No patterns yet — just the system and its external actors |
| **L2 — Container** | Engineering team | Group 1 (Deployment) + Group 2 (Communication) |
| **L3 — Component** | Developers | Group 3 (Code Structure) + Group 4 (Data) |
| **Sequence Diagram** | Developers | Group 5 (Failure Handling) — failure branches visible here |
| **Class Diagram** | Individual developers | Group 6 (Code Design) — class-level patterns visible here |

The key insight: do not mix patterns from different C4 levels in the same diagram. A L2 Container diagram that shows class-level Strategy patterns is mixing Group 1 with Group 6 — two different zoom levels on the same canvas. This is one of the most common reasons architecture diagrams confuse readers.

### Mapping to HLD and LLD

| Document type | Groups covered | Owner |
|---|---|---|
| **HLD — High Level Design** | Group 1 (Deployment) + Group 2 (Communication) + architecture decisions | Architect / Engineering Manager |
| **LLD — Low Level Design** | Group 3 (Code Structure) + Group 4 (Data) + Group 5 (Failure) + Group 6 (Code Design) + API specs + ERD | Senior Engineer / Tech Lead |

HLD answers the what and why at system level. LLD answers the how at implementation level. They are different audiences making different decisions.

---

## Pattern Combinations That Commonly Work Together

Patterns do not exist in isolation. Certain combinations appear repeatedly in production systems because they solve complementary problems. Understanding these combinations helps you design coherently rather than pattern by pattern in isolation.

### The Standard Enterprise Starting Point

```
Monolith (Group 1)
+ REST (Group 2)
+ Layered Architecture (Group 3)
+ CRUD + Repository (Group 4)
+ Retry (Group 5)
+ Factory + Strategy (Group 6)
```

Most enterprise applications start here. It is the combination that is easiest to build, easiest to hire for, and easiest to maintain at early scale. Start here unless you have a specific reason not to.

### The Scalable Event-Driven System

```
Microservices (Group 1)
+ Event-Driven + Pub/Sub + REST (Group 2)
+ Hexagonal per service (Group 3)
+ CQRS + Event Sourcing (Group 4)
+ Circuit Breaker + DLQ + Saga (Group 5)
+ Observer + Strategy (Group 6)
```

Netflix, Uber, and most large-scale platforms converge on this combination. High complexity to build and operate — only justified when independent scaling and deployment are genuine requirements.

### The AI / Agentic System (Modern Pattern)

```
Hybrid Serverless + Containerized (Group 1)
+ Event-Driven + REST + SSE Streaming (Group 2)
+ Pipeline + Hexagonal (Group 3)
+ Repository + CQRS lite (Group 4)
+ Circuit Breaker + Retry + DLQ (Group 5)
+ Strategy per agent + Factory (Group 6)
```

This is the combination that appears in multi-agent AI pipelines — including Cricket AI. Event-driven for pipeline triggers, streaming for real-time LLM output, hexagonal for swappable LLM providers, circuit breaker for unstable LLM endpoints.

### The Modular Monolith (The Underrated Choice)

```
Modular Monolith (Group 1)
+ REST + Message Queue internally (Group 2)
+ DDD with bounded contexts (Group 3)
+ Repository per module (Group 4)
+ Retry + Circuit Breaker (Group 5)
+ Factory + Facade (Group 6)
```

The modular monolith is frequently the correct choice for teams that want the organizational benefits of Microservices without the operational overhead. Each module is internally isolated and could be extracted into a separate service later — but does not have to be.

---

## How Systems Evolve — The Pattern Journey

Most systems do not start at their final architecture. They evolve through stages as complexity grows. Understanding this journey helps you make appropriate decisions at each stage rather than over-engineering upfront.

```
Stage 1 — MVP / Proof of Concept
Monolith + REST + Layered + CRUD
→ Build fast. Validate the idea. Do not over-engineer.
→ Technical debt is acceptable here.

Stage 2 — Growing Product
Modular Monolith + REST + Layered + Repository
→ Organize the monolith internally before it becomes a mess.
→ Add module boundaries now — extract services later if needed.

Stage 3 — Scaling Pressure
Hybrid + Event-Driven + Hexagonal + CQRS
→ Specific parts are under pressure — extract only those parts.
→ Not everything needs to be a microservice.

Stage 4 — Platform Scale
Microservices + Event-Driven + Pub/Sub + DDD + CQRS + Saga
→ Full decomposition justified by team size and traffic.
→ Investment in observability, service mesh, distributed tracing essential.
```

The most common mistake in system design is starting at Stage 4 when Stage 1 or 2 is appropriate. Premature architectural complexity is a form of technical debt — it costs more to maintain than the simplicity it was meant to replace.

---

## AI and LLM Specific Patterns

As AI systems become a core part of software architecture, a new set of patterns has emerged that do not fit cleanly into the six traditional groups. These patterns are specific to systems that use large language models, retrieval systems, and autonomous agents.

### RAG — Retrieval-Augmented Generation

**The question it answers:** How does an LLM answer questions about data it was not trained on?

```
User query
    → Embed query into vector
        → Search vector database for relevant chunks
            → Inject chunks into LLM prompt as context
                → LLM generates grounded answer
```

This is a pipeline pattern (Group 3) combined with a specific data access pattern (Group 4). The retrieval step is the key — instead of relying on the LLM's training data, you retrieve relevant context at query time and inject it. Accuracy improves because the LLM is answering based on your data, not its general knowledge.

**When to use:** Any system that needs an LLM to answer questions about proprietary, recent, or domain-specific data that is not in the model's training set.

**What you give up:** Retrieval quality determines answer quality — garbage in, garbage out. Chunking strategy, embedding model, and similarity search configuration all affect results significantly.

### Agent Pattern

**The question it answers:** How does an LLM make decisions and take actions autonomously?

```
Task given to agent
    → LLM reasons about what to do
        → LLM selects and calls a tool
            → Tool returns result
                → LLM reasons again with new context
                    → Repeat until task complete
```

An agent is an LLM with access to tools — functions it can call to interact with the world. The LLM decides which tool to call, calls it, gets the result, and continues reasoning. This loop repeats until the task is complete or a stopping condition is met.

**When to use:** Tasks that require multiple steps, decisions, or interactions with external systems that cannot be determined upfront.

**What you give up:** Non-determinism — agents can take different paths to the same answer. Harder to test and debug than traditional code.

### Multi-Agent Pattern

**The question it answers:** How do you break a complex task across multiple specialized LLMs?

```
Orchestrator Agent
    → delegates to Research Agent
    → delegates to Analysis Agent
    → delegates to Summary Agent
    → collects and synthesizes all outputs
```

Rather than one agent trying to do everything, multiple specialized agents handle different aspects of a task. An orchestrator coordinates them. This maps to the Pipeline pattern (Group 3) at the agent level — each agent enriches the context and passes it forward.

**When to use:** Tasks too complex for a single agent — long context, multiple specializations needed, or parallel processing of sub-tasks.

**What you give up:** Coordination complexity — inter-agent communication, context passing, and failure handling across agents must be explicitly managed.

### Tool Calling / MCP Pattern

**The question it answers:** How does an LLM interact with external systems and APIs?

```
LLM receives task
    → LLM outputs structured tool call: { tool: "search", query: "..." }
        → Host system executes the tool
            → Result injected back into LLM context
                → LLM continues with new information
```

Tool calling standardizes how LLMs invoke external capabilities. MCP (Model Context Protocol) extends this to a standard protocol — tools are defined once and can be used by any MCP-compatible model. This maps to the Adapter pattern (Group 6) at the AI layer.

**When to use:** Any agent system that needs to interact with real-world data, APIs, databases, or services.

**What you give up:** Security surface area — the LLM is deciding what to call. Input validation and authorization on tool calls is critical.

### Guardrail Pattern

**The question it answers:** How do you prevent an LLM from producing harmful, incorrect, or off-topic output?

```
User input
    → Input guardrail (check for harmful content, PII, off-topic)
        → LLM generates response
            → Output guardrail (check for hallucination, toxicity, format)
                → Safe response delivered to user
```

Guardrails are validation layers around LLM calls. Input guardrails check what goes in. Output guardrails check what comes out. This maps to the Decorator pattern (Group 6) — the LLM call is decorated with pre and post processing without changing the core logic.

**When to use:** Any production AI system. Non-negotiable for customer-facing applications.

**What you give up:** Latency — each guardrail check adds processing time. Over-aggressive guardrails create false positives and frustrate legitimate users.

### Prompt Management Pattern

**The question it answers:** How do you version, organize, and reuse prompts across a system?

```
prompts/
├── rag/system.j2          → Jinja2 template with variables
├── agents/research.j2     → Agent-specific system prompt
└── guardrails/safety.j2   → Safety check prompt

At runtime:
template = load("prompts/rag/system.j2")
rendered = template.render(context=chunks, user=query)
```

Treating prompts as first-class artifacts — stored in files, versioned in git, rendered with templates — rather than hardcoding them in application code. This enables prompt iteration without code deployments and clear ownership of what the LLM is being asked to do.

**When to use:** Any system with more than one or two prompts. Especially important for multi-agent systems where each agent has its own system prompt.

**What you give up:** Another layer to manage — prompts are now separate artifacts that must be tested, versioned, and reviewed alongside code.

---

## Anti-Patterns — What to Avoid

Knowing what not to do is as important as knowing what to do. These are the most common architecture mistakes and the pattern group they violate.

| Anti-pattern | What it looks like | Why it hurts |
|---|---|---|
| **Distributed Monolith** | Microservices that must all deploy together and share a database | Microservices complexity with none of the benefits |
| **God Service** | One service that does everything | Defeats the purpose of any decomposition — the new monolith |
| **Chatty Microservices** | Services making dozens of synchronous calls to each other per request | Latency compounds — one slow service slows everything |
| **Shared Database** | Multiple services reading and writing the same database tables | Creates hidden coupling — one schema change breaks multiple services |
| **Event Soup** | Events everywhere with no clear ownership or schema | Impossible to trace what caused what — debugging nightmare |
| **Premature CQRS** | CQRS on a simple app with balanced read/write patterns | Double the code, no performance benefit |
| **Singleton Abuse** | Global state everywhere via Singleton pattern | Hidden dependencies, untestable code |
| **Anemic Domain Model** | Domain objects with only getters/setters, all logic in services | Defeats the purpose of DDD — just a fancy CRUD app |
| **Missing Failure Handling** | External calls with no Retry, Circuit Breaker, or timeout | One slow dependency takes down the entire system |
| **Hardcoded Prompts** | LLM prompts embedded directly in application code | Cannot iterate prompts without a code deployment |

---

## Real Project Scenario — All Six Groups Applied

Reading patterns in isolation is useful. Seeing them applied together to a real system is where understanding locks in.

The scenario below walks through a **Food Delivery Platform** — think GrabFood or FoodPanda. It is a system most people understand as users. The goal is to show how all six groups are answered simultaneously for the same product, and why each decision was made the way it was.

---

### The System: FoodNow — Online Food Delivery Platform

**What it does:** Customers browse restaurants, place orders, pay online, and track their delivery in real time. Restaurants receive and manage orders. Riders pick up and deliver. Admins monitor the platform.

**Scale:** Thousands of concurrent users, hundreds of partner restaurants, real-time delivery tracking, payment processing.

---

### Group 1 — Deployment Decision

**Question asked:** What are the deployable units? What needs to scale independently?

```
Analysis:
├── Order processing spikes at lunch and dinner
├── Payment must be isolated — a bug there cannot
│   take down the rest of the system
├── Notifications are burst events — fire and forget
├── Admin dashboard has minimal traffic
└── Each part has a different scaling profile
```

**Decision:**

```
Microservices for core domains:
├── Order Service          → scales at peak meal times
├── Restaurant Service     → scales when restaurants go online
├── Payment Service        → isolated — failure stays contained
├── Delivery Tracking      → scales with active deliveries
└── User / Auth Service    → scales with login peaks

Serverless for event-triggered work:
├── Notification Service   → fires per event, scales to zero
└── Invoice Generator      → triggers on order completion

Monolith for internal tools:
└── Admin Dashboard        → low traffic, not customer-facing
```

**Why not one Monolith?** Payment failures must not crash order browsing. Each domain has independent scaling needs. The team is large enough to own separate services.

**Why not pure Serverless?** Order processing has complex state. Delivery tracking requires persistent connections. Serverless is reserved for stateless burst tasks.

---

### Group 2 — Communication Decision

**Question asked:** For each interaction — does the caller need to wait, or can it fire and move on?

```
Interaction 1: Customer browses restaurants
→ Needs immediate results
→ REST — synchronous

Interaction 2: Customer places order
→ Needs immediate confirmation (order received)
→ REST for the confirmation response
→ Then: order placed event fires into the system

Interaction 3: Order event triggers downstream
→ Kitchen notified      → Event-Driven (no wait)
→ Payment initiated     → Event-Driven (no wait)
→ Rider assigned        → Event-Driven (no wait)
→ Audit log updated     → Event-Driven (no wait)

Interaction 4: Payment processing
→ One payment per order — must not be processed twice
→ Message Queue — exactly-once delivery guaranteed

Interaction 5: Restaurant availability broadcast
→ Multiple services need to know a restaurant went offline
→ Pub/Sub — Order Service, Search Service, and
             Notification Service all subscribe

Interaction 6: Delivery tracking
→ Customer watches their rider move in real time
→ WebSocket / SSE — persistent connection, live updates
```

**The result:** REST, Event-Driven, Message Queue, Pub/Sub, and WebSocket all coexist in the same system — each serving a different interaction with different requirements.

---

### Group 3 — Code Structure Decision

**Question asked:** How is the inside of each service organized?

```
Order Service → Clean Architecture
Reason: Complex business rules — promotions, discounts,
        restaurant availability checks, fraud rules.
        Business logic must be at the center,
        protected from infrastructure changes.

Payment Service → Hexagonal (Ports and Adapters)
Reason: Must support GCash, PayMaya, credit cards, COD.
        Each is a swappable adapter around the same
        core payment processing logic.
        Swap a payment provider → only the adapter changes.

Restaurant Portal (web app) → MVC
Reason: UI-driven — restaurant staff manage their menu,
        hours, and incoming orders through a web interface.
        Model-View-Controller maps cleanly to this.

Notification Service → Layered
Reason: Simple — receive event, look up template, send message.
        No complex domain logic. Layered is appropriate.

Delivery Tracking Service → Hexagonal
Reason: Must support multiple map providers (Google Maps,
        OpenStreetMap) and multiple communication protocols.
        Swappable adapters for each.
```

---

### Group 4 — Data Decision

**Question asked:** How does each service read and write data?

```
Order History → CQRS
Reason: Customers read their order history constantly.
        New orders are written far less frequently than read.
        Write model: normalized for fast inserts.
        Read model: denormalized for fast customer queries.
        Different shapes for different access patterns.

Payment Transactions → Event Sourcing
Reason: Every payment event must be auditable.
        "Was this charged? When? Was it refunded?"
        Every state change is stored as an event —
        charge initiated, charge confirmed, refund requested,
        refund processed. Current state is derived from history.
        Non-negotiable for financial compliance.

Restaurant Menu → Repository Pattern
Reason: Menu data is read frequently but rarely written.
        Abstracting behind a repository lets the team
        swap the underlying store (SQL to NoSQL)
        without changing service logic.

Delivery Location → CRUD
Reason: Simple — rider sends GPS coordinates,
        system stores latest position.
        Read and write patterns are identical.
        No separation needed.
```

---

### Group 5 — Failure Handling Decision

**Question asked:** What happens when each external dependency fails?

```
Payment Gateway calls → Circuit Breaker + Retry
Risk: Gateway goes slow under peak load.
Solution: Retry 3 times with exponential backoff.
          If still failing — Circuit Breaker trips.
          Returns "payment temporarily unavailable"
          instead of hanging the user's order indefinitely.

Multi-step Order Flow → Saga
Risk: Order confirmed, kitchen notified,
      but payment fails at step 3.
      The order is now in an inconsistent state.
Solution: Saga pattern — each step has a compensating action.
          Payment fails → cancel kitchen order →
          release restaurant slot → notify customer →
          refund if charged.
          Every step is reversible.

Failed Notifications → Dead Letter Queue
Risk: Push notification fails because customer's
      phone is offline. SMS gateway is down.
Solution: Failed notification goes to DLQ.
          Retried when the channel is available.
          Nothing silently disappears.

Service Isolation → Bulkhead
Risk: Delivery tracking service gets overloaded
      during a major event. Other services must
      not be affected.
Solution: Each service has its own thread pool and
          connection limit. Delivery tracking slowdown
          does not starve the payment service.
```

---

### Group 6 — Code Design Decision

**Question asked:** How are the classes and objects within each service written?

```
Payment Processors → Strategy Pattern
Same interface, different implementation per provider:

class PaymentProcessor:
    def charge(amount, details): ...

class GCashProcessor(PaymentProcessor): ...
class PayMayaProcessor(PaymentProcessor): ...
class CreditCardProcessor(PaymentProcessor): ...

At runtime, swap the strategy based on customer choice.
The Order Service does not change. The processor is plugged in.

─────────────────────────────────────────────

Order Status Changes → Observer Pattern
When an order status changes, multiple systems
need to know automatically:

order.status_changed("PICKED_UP")
    → CustomerNotificationObserver.notify()
    → RestaurantDashboardObserver.notify()
    → AnalyticsObserver.notify()
    → AuditLogObserver.notify()

Each observer registers interest once.
The Order object fires — observers handle their own logic.

─────────────────────────────────────────────

Complex Order Construction → Builder Pattern
An order has many optional components:

OrderBuilder()
  .set_customer(customer_id)
  .set_restaurant(restaurant_id)
  .add_item(item_id, quantity=2)
  .add_item(item_id, quantity=1, note="no onions")
  .apply_promo("SAVE50")
  .set_delivery_address(address)
  .set_payment_method("gcash")
  .build()

Each step is optional and chainable.
The builder handles validation before the order is created.

─────────────────────────────────────────────

Restaurant Types → Factory Pattern
Different restaurant types have different rules —
cloud kitchens, physical restaurants, franchise branches.

RestaurantFactory.create(type="cloud_kitchen", config)
RestaurantFactory.create(type="franchise", config)

The factory decides which class to instantiate.
The calling code never sees the instantiation logic.
```

---

### Complete Pattern Map for FoodNow

```
FOODNOW FOOD DELIVERY PLATFORM

┌────────────────────────────────────────────────────────┐
│  GROUP 1 — DEPLOYMENT                                  │
│  Microservices (Order, Payment, Restaurant, Delivery)  │
│  Serverless (Notifications, Invoice Generator)         │
│  Monolith (Admin Dashboard)                            │
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│  GROUP 2 — COMMUNICATION                               │
│  REST         → Browse, place order, get status        │
│  Event-Driven → Order placed triggers downstream       │
│  Message Queue→ Payment processing (exactly once)      │
│  Pub/Sub      → Restaurant availability broadcast      │
│  WebSocket    → Live delivery tracking                 │
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│  GROUP 3 — CODE STRUCTURE                              │
│  Clean Architecture → Order Service (complex rules)    │
│  Hexagonal          → Payment + Delivery (swappable)   │
│  MVC                → Restaurant Portal (UI-driven)    │
│  Layered            → Notification Service (simple)    │
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│  GROUP 4 — DATA                                        │
│  CQRS           → Order history (reads >> writes)      │
│  Event Sourcing → Payment transactions (audit trail)   │
│  Repository     → Restaurant menu (swappable store)    │
│  CRUD           → Delivery location (simple read/write)│
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│  GROUP 5 — FAILURE HANDLING                            │
│  Circuit Breaker → Payment gateway calls               │
│  Saga            → Multi-step order transaction        │
│  Dead Letter Queue → Failed notifications              │
│  Bulkhead        → Service isolation under load        │
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│  GROUP 6 — CODE DESIGN                                 │
│  Strategy  → Swappable payment processors              │
│  Observer  → Order status change notifications         │
│  Builder   → Complex order construction                │
│  Factory   → Restaurant type instantiation             │
└────────────────────────────────────────────────────────┘
```

---

### Key Takeaway from This Scenario

Every pattern in every group was chosen by answering one specific question at one specific zoom level. No pattern was chosen because it sounded impressive. Each one was chosen because it was the right answer to a real problem.

Notice also what was **not** used where it was not needed:

```
Admin Dashboard      → Monolith, not Microservices
                       Low traffic. No need for complexity.

Delivery Location    → CRUD, not Event Sourcing
                       Simple read/write. No audit needed.

Notification Service → Layered, not Clean Architecture
                       Simple logic. No complex domain.
```

Over-engineering is as dangerous as under-engineering. The thinking model tells you which question to ask. The answer to that question — given your specific context — tells you which pattern to use.

---

## Why the Industry Got Confused — A Brief History

Understanding the origin of the confusion helps it stop feeling like a personal failure.

**1994 — Gang of Four** publish *Design Patterns*. Twenty-three patterns for writing clean object-oriented code. Invaluable. But everyone starts calling them "architecture patterns" even though they operate at the class level.

**2002 — Martin Fowler** publishes *Patterns of Enterprise Application Architecture*. Mixes code structure patterns (Group 3) with data patterns (Group 4) under one roof. Still one of the best books ever written — but the blurred categorization follows the industry forward.

**2014–2018 — Microservices hype**. Netflix and Amazon make Microservices the dominant conversation. The industry starts comparing everything to Microservices — including patterns from completely different groups. "Should I use Microservices or Event-Driven?" becomes a real question people ask, even though it is like asking "Should I drive a truck or use GPS?"

**2016–present — YouTube and tutorial culture**. System design interview content explodes. Creators understandably simplify. Patterns from all six groups end up in the same diagram, the same video, the same list — with no indication that they answer different questions.

**2022–present — AI/LLM patterns**. A new category of patterns emerges that the traditional six groups do not fully capture — RAG, agents, tool calling, guardrails. These are now appearing in system design conversations without a clear framework to place them in. The confusion cycle starts again.

The confusion is not your fault. The vocabulary was never standardized. Now you have the model to cut through it.

---

## Further Reading

The following resources are worth going deeper on each group.

### Foundational Books

| Book | Author | Groups covered |
|---|---|---|
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gang of Four | Group 6 |
| *Patterns of Enterprise Application Architecture* | Martin Fowler | Groups 3 and 4 |
| *Building Microservices* | Sam Newman | Groups 1, 2, and 5 |
| *Domain-Driven Design* | Eric Evans | Group 3 (DDD) |
| *Designing Data-Intensive Applications* | Martin Kleppmann | Groups 2 and 4 |
| *Clean Architecture* | Robert C. Martin | Groups 3 and 6 |
| *Release It!* | Michael Nygard | Group 5 |

### AI and LLM Specific

| Resource | What it covers |
|---|---|
| Anthropic documentation | Prompt engineering, tool calling, agent patterns |
| LangChain / LangGraph docs | Agent orchestration patterns |
| Azure AI Foundry docs | Production LLM deployment patterns |
| *Building LLM Powered Applications* | LLM application architecture |

### Diagramming and Communication

| Resource | What it covers |
|---|---|
| c4model.com | C4 Model — how to draw architecture at the right zoom level |
| Simon Brown — *Software Architecture for Developers* | Practical architecture thinking |
| arc42.org | Architecture documentation template used in enterprise |

---

## A Practical Reminder

Architecture is communication first. A diagram is only as good as the clarity it creates for the person reading it. Before drawing anything, ask:

- Who is the audience?
- What decision do they need to make?
- What zoom level serves that decision?

The most stunning architecture diagram that nobody understands is worth less than a rough sketch that makes the right decision obvious to the right person at the right time.

The patterns in this document are tools for thinking — not trophies for displaying. Use them to answer questions. Let the answers drive the diagrams. Not the other way around.
