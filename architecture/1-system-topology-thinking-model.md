# System Topology: A Thinking Model, Not a Stack List

## The Confusion Nobody Talks About

Ask ten developers to draw how their system works and most of them will draw the same thing: a box labeled "backend" connected to a box labeled "database," with an arrow from a browser on the left. That is it. That is the whole picture.

Now ask those same developers what a CDN is, or a load balancer, or an API Gateway. Most of them know the words. Some of them have configured one before. But very few can tell you exactly where each one sits, why it exists, and what breaks if it is missing — because nobody ever showed them the complete picture in order.

The software industry teaches tools in isolation. You learn what Redis is. You learn what Nginx is. You learn what Kubernetes is. But nobody hands you a fixed map that shows where every piece lives and how they connect end to end — from the moment a user taps a button to the moment data comes back.

The result? Developers encounter terms like "edge cache," "SSL terminator," or "dead letter queue" and they either nod along without really knowing where it sits in the system, or they go look it up and get a definition with no picture. The definition is accurate. The picture is missing. And without the picture, the definition does not stick.

The shift starts when you stop thinking about these components as individual technologies and start seeing them as **stops in a fixed journey** — the journey every request makes from a client to data and back. That journey is the same for every system. What changes is which stops have been built, and which ones are still missing.

This document gives you the complete map.

---

## How to Use This Document

Before reading further, internalize this principle:

> **Every time you encounter a component name, ask two questions first: which layer is this? and what stop in the journey does it represent?**

The layers below are ordered intentionally — from the outermost (closest to the user) to the innermost (closest to the data). A good architect reads this top-down when designing a new system. A good engineer reads it bottom-up when tracing a bug or a slow request. Both directions are valid.

Each layer includes a **FoodNow scenario** — a consistent food delivery platform used throughout the entire document. The same system is referenced in every layer so that components can be compared against a familiar baseline rather than starting from scratch each time.

---

## About This Thinking Model

**Type:** Infrastructure Assessment Model

This is a **structural awareness model** — a mental framework that helps you see the full infrastructure picture of any system before you assess, build, or improve it.

Most engineers are trained to think inside the application — the code, the logic, the features. This model trains you to think outside the application — the roads, the layers, and the infrastructure that carries every request from a user's device all the way to the database and back.

It sits at the highest level of architectural thinking, one level above how the application itself is structured internally.

**How it helps:**

- **When taking over a codebase** — instantly know which infrastructure layers exist, which are missing, and which gaps are risks versus intentional decisions
- **When designing a new system** — start from the full picture and consciously decide which layers to build now and which to defer
- **When diagnosing a problem** — trace the request journey layer by layer to find exactly where performance, reliability, or security breaks down
- **When talking to stakeholders** — use the universal picture as a shared language that anyone from a junior engineer to a CTO can understand
- **When scaling a system** — know which layer to add next based on where the system is in its maturity progression

**Where it sits in the Architecture Thinking Model Series:**

```
Level 1 — System Topology (this model)
          How does the outside world reach the system?
          The universal infrastructure picture every system lives inside.

Level 2 — Layer Components
          What component groups live inside each layer?
          Caching Group, Security Group, Access Group, Distribution Group...

Level 3 — 6-Pattern Architecture Model
          How is the Application Server itself structured internally?
          Deployment, Communication, Code Structure,
          Data Management, Failure Handling, Code Design.

Level 4 — C4 Model
          How do you visualize and communicate the system?
          System → Container → Component → Code.
```

Think of Level 1 as the city infrastructure, Level 2 as what runs inside each road and intersection, Level 3 as how the buildings inside the city are constructed, and Level 4 as the blueprint you use to explain it all to others.

---

## Table of Contents

0. [The Confusion Nobody Talks About](#the-confusion-nobody-talks-about)
1. [What Is System Topology](#1-what-is-system-topology)
2. [Why This Model Exists](#2-why-this-model-exists)
3. [The Universal Picture](#3-the-universal-picture)
4. [The Layers Explained](#4-the-layers-explained)
   - 4.1 [Client](#41-client)
   - 4.2 [CDN / Edge Network](#42-cdn--edge-network)
   - 4.3 [Load Balancer](#43-load-balancer)
   - 4.4 [API Gateway](#44-api-gateway)
   - 4.5 [Application Server](#45-application-server)
   - 4.6 [Cache](#46-cache)
   - 4.7 [Message Queue + Worker](#47-message-queue--worker)
   - 4.8 [Object Storage](#48-object-storage)
   - 4.9 [Search Engine](#49-search-engine)
   - 4.10 [Database — Primary + Read Replica](#410-database--primary--read-replica)
   - 4.11 [External Services](#411-external-services)
   - 4.12 [Observability](#412-observability)
5. [Maturity Progression](#5-maturity-progression)
6. [How To Use This Model](#6-how-to-use-this-model)
7. [Concrete Examples — Two Real Systems](#7-concrete-examples--two-real-systems)
   - 7.1 [FoodNow — Food Delivery Platform](#71-foodnow--food-delivery-platform)
   - 7.2 [Analytics Migrator — Ideal Future Architecture](#72-analytics-migrator--ideal-future-architecture)
   - 7.3 [Side-by-Side Comparison](#73-side-by-side-comparison)

---

## 1. What Is System Topology

System Topology is the universal picture of how a client and a server are connected to each other — and every layer that sits in between them.

Every system that has a client and a server follows this same picture. The layers do not change from project to project. What changes is **which layers a project has built so far** based on its scale, maturity, and needs.

Think of it like a city's road infrastructure. Every city has the same types of roads — highways, intersections, traffic lights, bridges. A small town only needs a few of them. A megacity needs all of them. The roads are always the same roads. The city just grows into them over time.

---

## 2. Why This Model Exists

Most engineers see a system as just a backend and a database. Architects see the full picture — every stop a request makes from the moment a user clicks something to the moment data comes back.

This model exists so that when you walk into any project — whether it is a startup or an enterprise — you have a fixed mental checklist to ask:

- Which layers does this system have?
- Which are missing?
- Which are missing and dangerous?
- Which will be needed as it grows?

That is a complete architectural assessment using one universal picture.

---

## 3. The Universal Picture

```
CLIENT
  │
  ▼
CDN / Edge Network          ← static assets, geographic proximity
  │
  ▼
Load Balancer               ← distributes traffic across servers
  │
  ▼
API Gateway                 ← auth, rate limiting, routing
  │
  ▼
Application Server          ← your business logic lives here
  │
  ├──▶ Cache                ← fast reads, avoid hitting the database
  ├──▶ Message Queue        ← async tasks, background processing
  │         │
  │         ▼
  │       Worker            ← consumes and executes queued tasks
  ├──▶ Object Storage       ← files, images, documents, blobs
  └──▶ Search Engine        ← full-text search across large datasets
  │
  ▼
Database
  ├── Primary               ← handles all write operations
  └── Read Replica          ← handles read queries, offloads primary
  │
  ▼
External Services           ← payments, email, SMS, third-party APIs
  │
  ▼
Observability               ← logging, metrics, tracing
```

---

## 4. The Layers Explained

Each layer below follows the same structure:

- **What it is** — plain explanation of what this component is
- **What it does** — the specific job it performs in the system
- **Why it matters** — its role in the bigger picture
- **What breaks without it** — the real consequence of not having it
- **FoodNow scenario** — how this layer shows up in a real food delivery platform

---

### 4.1 Client

**What it is**
The client is whatever the user is holding or looking at. A browser, a mobile app, a desktop app, or even another system making requests programmatically.

**What it does**
It sends a request — "give me this page", "save this data", "process this order" — and waits for a response. Everything below this layer exists to handle that request correctly, quickly, and safely.

**Why it matters**
The entire system exists to serve the client. Every architectural decision you make should trace back to improving the client's experience. If you ever lose sight of that, you end up building infrastructure for its own sake.

**What breaks without it**
Without a client there is no system. This layer is always present by definition.

**FoodNow scenario**

> FoodNow has three clients simultaneously. The mobile app — used by customers to browse restaurants and place orders. The restaurant portal — a web browser interface where restaurant staff manage their menu and incoming orders. The rider app — a mobile application where delivery riders receive pickups and update their location. All three are clients. All three send requests. All three wait for responses. The entire infrastructure below exists to serve all three of them reliably at the same time.

---

### 4.2 CDN / Edge Network

**What it is**
CDN stands for Content Delivery Network. It is a network of servers spread across the world — in different cities and countries — that store copies of your static content close to your users.

**What it does**
Instead of a user in Manila waiting for a response from a server in the United States, the CDN serves the content from a nearby server — maybe in Singapore. Images, CSS files, JavaScript bundles, videos — anything that does not change per user gets served from the edge, not from your origin server.

**Why it matters**
Speed is directly tied to revenue, user retention, and perception of quality. A one-second delay in page load can cost significant conversion. The CDN solves the physics problem of distance. You cannot make the internet faster, but you can move your content closer to the people who need it.

**What breaks without it**
Every user hits your origin server regardless of where they are in the world. A user in Europe and a user next door to your datacenter get the same slow experience. Your origin server handles every request including static files it should never have to touch. Under high traffic it gets overwhelmed serving content that a CDN would have handled automatically.

**FoodNow scenario**

> FoodNow's app needs to load restaurant photos, food images, and the JavaScript bundle every time a customer opens it. Without a CDN, every customer in Manila, Cebu, Davao, and Singapore is hitting the same origin server in Manila — loading the same restaurant cover photo, the same menu item images, the same app bundle on every request. With CDN, those assets are cached at edge servers in each region. A customer in Singapore loads the FoodNow app in milliseconds because the images are served from a nearby CDN node — not from Manila. The origin server never sees those requests.
>
> ```
> Without CDN:
> Customer in Singapore → request → Manila server → Manila server returns image
>                                    (high latency, origin overloaded)
>
> With CDN:
> Customer in Singapore → request → Singapore CDN edge → returns cached image
>                                    (low latency, origin untouched)
> ```

---

### 4.3 Load Balancer

**What it is**
A load balancer is a traffic director that sits in front of your application servers. It receives all incoming requests and decides which server should handle each one.

**What it does**
It distributes traffic across multiple instances of your application server so no single server gets overwhelmed. If one server goes down, it stops sending traffic to it and routes to the healthy ones. It keeps the system alive even when individual parts fail.

**Why it matters**
One server has a ceiling. At some point it cannot handle more traffic no matter how powerful it is. The load balancer is what allows you to scale horizontally — add more servers instead of buying a bigger one. It is also what gives you zero-downtime deployments, because you can take servers in and out of rotation without users noticing.

**What breaks without it**
You have one server handling everything. When traffic spikes, it slows down or crashes and everyone feels it. When you deploy a new version, the site goes down. There is no redundancy — one hardware failure brings down the entire system.

**FoodNow scenario**

> FoodNow gets hammered at lunch and dinner. From 12:00 to 13:00 and 18:00 to 20:00, order volume spikes by 5x. Without a load balancer, one Order Service instance handles all of it — slows down, crashes, customers get errors during peak hours. With a load balancer, FoodNow runs five Order Service instances during peak hours. The load balancer distributes incoming orders across all five. If one instance gets an out-of-memory error and crashes, the load balancer stops sending requests to it and redistributes the load across the remaining four — customers never notice.
>
> ```
> Peak hours (12:00-13:00):
>
> Without load balancer:
>     1000 orders/min → 1 server → server CPU at 100% → timeout errors
>
> With load balancer:
>     1000 orders/min → load balancer → 5 servers (200 orders/min each)
>     Server 3 crashes → load balancer detects → redistributes to 4 servers
>     Customers see no errors
> ```

---

### 4.4 API Gateway

**What it is**
The API Gateway is the single entry point for all API requests into your system. Every request from the client passes through it before reaching your application.

**What it does**
It handles responsibilities that every API request needs regardless of what it is asking for — authentication, rate limiting, request routing, and logging. It is the bouncer, the traffic cop, and the translator all in one.

**Why it matters**
Without a gateway, every single service in your system has to implement its own authentication and rate limiting. That is duplicated work across every service and inconsistent enforcement. The gateway centralizes all of that in one place so your application servers only focus on business logic.

**What breaks without it**
Every service reinvents the same wheel. Authentication is implemented ten different ways across ten services. One service forgets rate limiting and gets hammered by a bot. You have no single place to see all incoming traffic. Changing an authentication policy means updating every service individually.

**FoodNow scenario**

> FoodNow's mobile app talks to three backend services: Order Service, Restaurant Service, and Delivery Service. Without an API Gateway, each service needs to validate the JWT token on every request, check if the user's account is active, enforce per-user rate limits, and log all incoming calls. That is the same code written and maintained three times. With an API Gateway, the customer's request hits the gateway first. The gateway validates the token, checks rate limits, and routes to the correct service. The services never see unauthenticated requests. When FoodNow needs to tighten rate limits during a DDoS attack, they change it in one place — not in three services.
>
> ```
> Without API Gateway:
>     Mobile app → Order Service   (validates token, checks rate limit, logs)
>     Mobile app → Restaurant Service (validates token, checks rate limit, logs)
>     Mobile app → Delivery Service   (validates token, checks rate limit, logs)
>     Same logic, three places. One service forgets. Security gap.
>
> With API Gateway:
>     Mobile app → API Gateway (validates token, checks rate limit, logs)
>         → routes to Order Service     (just handles orders)
>         → routes to Restaurant Service (just handles menus)
>         → routes to Delivery Service   (just handles tracking)
>     One place. Consistent. Services stay focused.
> ```

---

### 4.5 Application Server

**What it is**
This is your actual backend — the code your team wrote. It contains the business logic, the rules, and the processes that make your product what it is.

**What it does**
It receives the request that has passed through all the layers above, executes the business logic, talks to the database and other services, and returns a response. This is where your features live.

**Why it matters**
This is the reason all the other layers exist. The CDN, load balancer, and API gateway are all infrastructure protecting and enabling this layer. The database, cache, and queue are all resources this layer uses. Everything orbits around the application server.

**What breaks without it**
There is no product. This layer is always present by definition.

**FoodNow scenario**

> The FoodNow Order Service is the application server for order processing. When a customer places an order it receives the request from the API Gateway and runs all the business logic: check the restaurant is still open, verify the customer's account is active, validate the promo code, calculate the delivery fee, confirm items are still available, reserve a restaurant slot, and initiate payment. None of that logic lives in the gateway, the load balancer, or the database. It lives here. Every other layer in the system exists to get requests to this layer efficiently and to give this layer access to the resources it needs.

---

### 4.6 Cache

**What it is**
A cache is a fast, temporary storage layer — typically Redis or Memcached — that sits between your application server and your database.

**What it does**
It stores the results of expensive operations — database queries, API calls, computed values — so the next time someone asks for the same thing, you return it from memory in milliseconds instead of hitting the database again.

**Why it matters**
Databases are fast but they have limits. When thousands of users are all asking for the same data simultaneously — a product page, a leaderboard, a user profile — hitting the database for every request creates a bottleneck. The cache absorbs that load. It is the difference between a system that handles 10,000 users and one that handles 10 million.

**What breaks without it**
The database receives every read request no matter how repetitive. Under load it slows down, then fails. Response times grow. The entire system degrades because the database — the most expensive and least replaceable layer — is doing work that memory could have handled at a fraction of the cost.

**FoodNow scenario**

> FoodNow's Restaurant Service gets hit constantly as customers browse. Every customer that opens the app sees a list of restaurants. Every time they tap a restaurant, they see that restaurant's menu. The restaurant list and menus change rarely — maybe once a day when hours change or new items are added. Without a cache, every customer browsing generates a database query. During lunch rush — 10,000 concurrent users browsing restaurants — that is 10,000 database queries per second for data that has not changed in hours. With Redis cache, the restaurant list is loaded from the database once and stored in memory. All 10,000 users read from Redis — the database sees near-zero read traffic for browse operations.
>
> ```
> Without cache (lunch rush):
>     10,000 users browsing → 10,000 DB queries/sec → DB overwhelmed → timeouts
>
> With cache:
>     DB query runs once → result stored in Redis (TTL: 5 minutes)
>     10,000 users browsing → 10,000 Redis reads → DB sees ~0 browse queries
>     Restaurant updates menu → cache invalidated → next request refreshes from DB
> ```

---

### 4.7 Message Queue + Worker

**What it is**
A message queue is a buffer that holds tasks waiting to be processed. A worker is a background process that picks tasks off the queue and executes them.

**What it does**
It decouples work that does not need to happen immediately from the request cycle. When a user signs up, you need to send a welcome email, create their profile, notify analytics, and provision their account. Instead of doing all of that before responding to the user, you put those tasks in a queue, immediately respond "welcome aboard", and let workers process the tasks in the background.

**Why it matters**
Not everything needs to happen in real time. Forcing long-running tasks into the request cycle makes the user wait and ties up your application servers. The queue allows your system to absorb traffic spikes gracefully — tasks pile up during peaks and workers drain them when capacity is available. If a worker fails, the task stays in the queue and gets retried automatically.

**What breaks without it**
Every task runs synchronously in the request cycle. Users wait while emails send, reports generate, and files process. Under load, application servers get tied up with background work and cannot handle new incoming requests. A failure mid-task loses the work entirely with no retry mechanism.

**FoodNow scenario**

> When a FoodNow customer places an order, multiple things need to happen — but the customer should not wait for all of them. The Order Service places the order in the database and immediately returns a confirmation to the customer. The rest is queued. A worker picks up the `OrderPlaced` task and sends a push notification to the customer. Another worker picks up `NotifyRestaurant` and sends the order to the kitchen screen. Another picks up `InitiatePayment` and charges the customer's GCash. Another updates the analytics dashboard.
>
> ```
> Without queue:
>     Customer places order
>         → send push notification (300ms)
>         → notify restaurant (200ms)
>         → process payment (1500ms)
>         → update analytics (100ms)
>         → customer waits 2100ms for confirmation screen
>
> With queue:
>     Customer places order
>         → order saved to DB
>         → confirmation returned immediately (50ms)
>         → tasks queued in background
>     Worker 1: send push notification
>     Worker 2: notify restaurant kitchen
>     Worker 3: process GCash payment
>     Worker 4: update analytics
>     Customer sees confirmation in 50ms. Everything else happens behind the scenes.
> ```

---

### 4.8 Object Storage

**What it is**
Object storage is a service for storing files — images, videos, documents, backups, exports. Examples are Azure Blob Storage, Amazon S3, and Google Cloud Storage.

**What it does**
It stores and serves files at massive scale without those files ever touching your application server. Users upload directly to storage, and files are served directly from storage to the client.

**Why it matters**
Files are fundamentally different from data. They are large, they do not change often, and there can be billions of them. Storing files on your application server or in your database is one of the most common and costly architectural mistakes. Object storage is purpose-built for files — infinitely scalable, cheap, durable, and fast.

**What breaks without it**
Files get stored on the application server's local disk or worse, in the database. The disk fills up. The database bloats. Deployments become dangerous because they might overwrite uploaded files. Scaling becomes impossible because files only exist on one server. A server failure means permanent data loss.

**FoodNow scenario**

> FoodNow has hundreds of partner restaurants. Each restaurant has a cover photo, dozens of food item photos, a business permit document, and a weekly promotional banner. That is potentially thousands of images and documents, growing every week. Restaurant owners upload these through the portal. Without object storage, these files land on the Restaurant Service server's local disk. When the team deploys a new version, the server is replaced — all uploaded files are gone. With Azure Blob Storage, restaurant owners upload directly to storage. The files never touch the application server. The Restaurant Service only stores the file URL in the database. The file itself lives in Azure Blob Storage permanently — and gets served via CDN at the edge.
>
> ```
> Without object storage:
>     Restaurant uploads photo → saved to /app/uploads/restaurant_7.jpg
>     New deployment → server replaced → /app/uploads wiped → photo gone
>     1000 restaurants × 20 photos = 20,000 files on an application server
>
> With Azure Blob Storage:
>     Restaurant uploads photo → goes directly to blob storage
>     DB stores: { restaurant_id: 7, photo_url: "https://cdn.foodnow.com/r7.jpg" }
>     New deployment → server replaced → photo URL still valid → photo still there
>     1,000,000 files → no problem → infinitely scalable
> ```

---

### 4.9 Search Engine

**What it is**
A dedicated search service — typically Elasticsearch or OpenSearch — optimized for full-text search, filtering, and ranking across large datasets.

**What it does**
It indexes your data in a way that makes complex search queries fast — searching across thousands of fields, ranking results by relevance, handling typos, supporting filters and facets. It offloads search work that relational databases were never designed for.

**Why it matters**
A SQL database can do basic search but it becomes painfully slow on large datasets with complex queries. Search engines are built from the ground up for exactly this workload. When your product has a search bar that needs to return results in milliseconds across millions of records, a dedicated search engine is not optional — it is necessary.

**What breaks without it**
Search runs directly against the database using slow LIKE queries. It cannot rank by relevance. It cannot handle typos. It degrades the entire database's performance for everyone. As data grows, search becomes unusable and users stop trusting it.

**FoodNow scenario**

> A FoodNow customer types "chickn burga" into the search bar — with two typos. Without a search engine, the query runs as `SELECT * FROM menu_items WHERE name LIKE '%chickn burga%'` against the database — returns nothing, slows the DB, and the customer gets a blank screen. With Elasticsearch, the search engine handles typo tolerance, returns "chicken burger," and ranks results by distance, rating, and availability. The customer finds what they want in milliseconds. The database never sees the query.
>
> ```
> Without search engine:
>     "chickn burga" → DB LIKE query → 0 results → frustrated customer
>     100 concurrent searches → 100 DB table scans → DB degraded for everyone
>
> With Elasticsearch:
>     "chickn burga" → Elasticsearch handles typo → finds "chicken burger"
>     Results ranked by: distance from customer, restaurant rating, availability
>     Response: 40ms → 12 matching restaurants → database never touched
> ```

---

### 4.10 Database — Primary + Read Replica

**What it is**
The database is the permanent, structured store of your application's data. The primary handles writes. Read replicas are synchronized copies of the primary that handle read queries.

**What it does**
The primary receives all write operations — inserts, updates, deletes. It continuously replicates its data to one or more read replicas. Read-heavy operations like reports, dashboards, and list queries are directed to the replicas, protecting the primary from read load.

**Why it matters**
Most applications read far more than they write. Without read replicas, every query — whether it is a simple user lookup or a complex analytics report — hits the same instance that is also handling writes. Separating reads from writes is one of the highest-leverage decisions for scaling a database layer.

**What breaks without it**
All reads and writes compete for the same resources. A heavy analytics query can freeze the entire application. There is no horizontal scaling path for read traffic. One database failure takes down everything — reads and writes simultaneously — with no fallback.

**FoodNow scenario**

> FoodNow's operations team runs a dashboard that shows live order volume, revenue by hour, popular restaurants, and delivery performance metrics. These are complex queries joining multiple tables across millions of order records. Without read replicas, the dashboard's aggregation queries run against the same database instance that is processing live customer orders. A heavy dashboard query locks rows — customers placing orders during the lunch rush start seeing delays. With a read replica, the dashboard queries hit the replica. The primary database is protected and dedicated to write operations — placing orders, updating statuses, recording payments. A dashboard query running for 10 seconds has zero impact on a customer placing an order.
>
> ```
> Without read replica:
>     Dashboard query (SELECT SUM...) → primary DB
>     Lunch rush order placement    → primary DB
>     Both competing → primary DB locks → orders slow → customers frustrated
>
> With read replica:
>     Dashboard query → read replica (can be slow, nobody cares)
>     Order placement → primary DB (fast, protected, no competition)
>     Primary DB only handles writes — reads go elsewhere
> ```

---

### 4.11 External Services

**What it is**
Third-party services your application depends on — payment processors, email providers, SMS gateways, mapping services, identity providers, analytics platforms.

**What it does**
Handles specialized capabilities that are not worth building in-house. Stripe handles payments. SendGrid handles email. Twilio handles SMS. Instead of building and maintaining these complex systems yourself, you integrate with services that have already solved them at scale.

**Why it matters**
Every external service represents a build-vs-buy decision made in your favor. Payment processing alone involves PCI compliance, fraud detection, dispute management, and global currency support — years of work. Using Stripe means you get all of that in an afternoon of integration. Your team stays focused on what makes your product unique.

**What breaks without it**
You build everything yourself. This sounds powerful but it means your team spends months building payment systems, email infrastructure, and SMS delivery instead of building your actual product — and you build it worse than a company whose entire existence is solving that one problem.

**FoodNow scenario**

> FoodNow depends on several external services simultaneously. GCash and PayMaya handle payment processing — FoodNow does not want to touch raw financial transactions or manage PCI compliance. Firebase handles push notifications to the mobile app — FoodNow does not operate its own push notification infrastructure. Google Maps provides distance calculation, routing, and the live map on the tracking screen. Twilio sends SMS fallback notifications when push fails. None of these capabilities were built by the FoodNow team. Each one would take months to build and years to maintain to the same quality. The FoodNow team integrated each in days and focuses entirely on the food delivery experience.
>
> ```
> What FoodNow did NOT build:
>     Payment processing    → GCash API + PayMaya API
>     Push notifications    → Firebase Cloud Messaging
>     Mapping and routing   → Google Maps Platform
>     SMS fallback          → Twilio
>     Email receipts        → SendGrid
>
> What FoodNow DID build:
>     Order orchestration, restaurant matching, rider assignment,
>     menu management, loyalty points, promo engine
>     — the things that make FoodNow specifically FoodNow
> ```

---

### 4.12 Observability

**What it is**
Observability is the combination of three things — logging (what happened), metrics (how much and how fast), and tracing (which path did a request take through the system).

**What it does**
Logging captures events and errors as they happen. Metrics track numbers over time — response times, error rates, CPU usage, request counts. Tracing follows a single request as it travels through every layer of the system so you can see exactly where it slowed down or failed.

**Why it matters**
A system without observability is a black box. When something breaks you have no idea what happened, where it happened, or why. Observability turns your system from something you hope works into something you can understand, measure, and continuously improve. It is not a nice-to-have — it is what separates a system that is operated from a system that is guessed at.

**What breaks without it**
When something breaks you are blind. You restart servers and hope for the best. You have no idea which part of the system is slow. You cannot tell whether a deployment made things better or worse. You find out about problems when users complain, not when they start. Every incident is a mystery and every fix is a guess.

**FoodNow scenario**

> On a Saturday night, FoodNow customers start complaining that order confirmations are taking too long. Without observability, the on-call engineer restarts servers and guesses. With observability, they open the dashboard in 30 seconds and see the full picture. Metrics show the Order Service response time spiked at 20:15 — correlates exactly with the complaints. Distributed traces show each order request is spending 2.3 seconds in the GCash payment step instead of the normal 0.3 seconds. A log search for that time window shows GCash returning intermittent 503 errors. Root cause found in 2 minutes: GCash is degraded. Circuit breaker gets tuned. Team notifies customers. Problem solved before it becomes a full outage.
>
> ```
> Without observability:
>     Customers complain → engineer restarts servers → problem persists
>     → tries restarting again → guesses DB → actually it was GCash
>     Time to root cause: 45 minutes of guessing
>
> With observability:
>     Alert fires at 20:15 → metrics show Order Service latency spike
>     → traces show 2.3s on payment step → logs show GCash 503 errors
>     Time to root cause: 2 minutes
>     Fix applied: circuit breaker tuned, GCash team notified
> ```

---

## 5. Maturity Progression

No project starts with all layers. Every layer is optional until the system demands it. The key is knowing which stage you are at and which layer comes next.

**The FoodNow journey:**

```
Stage 1 — FoodNow MVP (just launched)
    Client → App Server → Database
    One server, one database, five developers.
    Everything else is premature optimization.

Stage 2 — FoodNow getting real traffic
    + Load Balancer
    One server cannot handle the lunch rush anymore.
    Add two more servers. Load balancer distributes across all three.

Stage 3 — FoodNow needs security and control
    + API Gateway
    Three mobile platforms, two web apps — all calling the same backend.
    Authentication and rate limiting need to be consistent.
    One gateway enforces the rules.

Stage 4 — FoodNow experiencing performance issues
    + Cache
    Restaurant menu queries are hammering the database.
    Same 500 restaurants being read 10,000 times per minute.
    Cache the menus. Database breathes.

Stage 5 — FoodNow handling async workloads
    + Message Queue + Worker
    Order placement is timing out because it waits for
    push notifications, emails, and payment processing.
    Queue everything. Return confirmation immediately.

Stage 6 — FoodNow growing file storage needs
    + Object Storage
    Restaurant photos on the app server disk.
    A deployment wiped them last Tuesday.
    Move all files to blob storage. Never lose them again.

Stage 7 — FoodNow search is unusable
    + Search Engine
    "chicken burger" returns nothing when typed as "chickn brger."
    Database LIKE queries are degrading read performance for everyone.
    Elasticsearch handles typos, ranking, and filters. Database is freed.

Stage 8 — FoodNow analytics killing the database
    + Read Replica
    Operations dashboard queries are locking rows during lunch rush.
    Read replica absorbs all reporting queries.
    Primary database is protected for writes only.

Stage 9 — FoodNow cannot see what is happening
    + Observability
    Third incident this month where engineers found out from customer complaints.
    No metrics, no traces, no structured logs.
    Logging, metrics, and tracing added across all services.
    Next incident: root cause in 2 minutes, not 45.
```

---

## 6. How To Use This Model

When you walk into any project, run through these five questions:

```
Question 1 — Which layers does this system currently have?

Question 2 — Which layers are missing?

Question 3 — Of the missing layers, which are intentionally absent?
             (not needed yet — the system is not at that scale)

Question 4 — Of the missing layers, which are gaps?
             (needed now but not built yet)

Question 5 — Of the missing layers, which are risks?
             (missing and actively dangerous to the system right now)
```

The answers give you a complete architectural assessment of any system — regardless of its size, technology stack, or domain. One universal picture. Five questions. Full clarity.

**The common confusion — cleared:**

> "Is the API Gateway the same as FastAPI authentication?"

No. FastAPI authentication is code inside your application — a function or middleware that runs inside the App Server layer. The API Gateway is a separate infrastructure component that sits outside your application, before the request even reaches your code. The gateway says "are you who you say you are?" FastAPI says "okay, but are you allowed to do this specific thing?" Both can coexist and serve different roles.

> "Can a system skip some layers permanently?"

Yes. A simple internal tool with five users will never need a CDN, a load balancer, or a search engine. The model is a reference, not a checklist. Every layer exists for a reason. If the reason does not apply to your system at its current scale, the layer is not needed. The maturity progression shows when each layer typically becomes necessary.

> "Do these layers change based on the technology used?"

No. Whether you use Azure, AWS, or GCP — whether your backend is Python, Node, or Java — the topology is the same. The technologies change. The layers do not. An API Gateway on Azure (Azure API Management) and an API Gateway on AWS (AWS API Gateway) are different products serving the same layer in the same position in the topology.

---

## 7. Concrete Examples — Two Real Systems

This section puts the model to work. Two systems are drawn side by side — FoodNow (a consumer-scale food delivery platform) and the Analytics Migrator (an enterprise internal migration tool). The same reference topology is used for both. What makes them different is which layers are present, what technology implements each one, and what is specific to each domain.

---

### 7.1 FoodNow — Food Delivery Platform

**System context:** A consumer-facing food delivery platform serving customers, restaurant partners, and delivery riders simultaneously. Thousands of concurrent users. Real-time order tracking. Payment processing. Peak traffic at lunch and dinner.

**Architect's reasoning:** This is a consumer product with unpredictable traffic spikes, real-time requirements, globally distributed users, and strict financial compliance for payments. Every layer of the topology is justified and actively in use. The scale demands full infrastructure maturity.

```
┌─────────────────────────────────────────────────────────────────────┐
│  CLIENTS                                                            │
│  Mobile App (iOS + Android)     → customers placing orders         │
│  Restaurant Portal (Web)        → staff managing menus and orders  │
│  Rider App (iOS + Android)      → riders accepting and tracking    │
│  Admin Dashboard (Web)          → internal ops and reporting       │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  CDN / EDGE — Azure CDN                                            │
│  What it serves: restaurant photos, food images, Vue/React bundle  │
│  Cache TTL: images 24hrs, app bundle 1hr                          │
│  Why: users in PH, SG, MY — all served from nearest edge node     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LOAD BALANCER — Azure Application Gateway                         │
│  Instances: Order Service ×5, Restaurant Service ×3               │
│             Delivery Service ×4 (scales with active orders)        │
│  Health check: every 30 seconds per instance                      │
│  Auto-scale: triggers at 70% CPU — adds instances during lunch     │
│  SSL termination: HTTPS decrypted here, plain HTTP internally      │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  API GATEWAY — Azure API Management                                │
│  Auth: Azure AD B2C JWT tokens validated on every request         │
│  Rate limiting: 100 req/min per user, 1000 req/min per restaurant  │
│  Routing:                                                          │
│    /orders/*         → Order Service                               │
│    /restaurants/*    → Restaurant Service                          │
│    /delivery/*       → Delivery Service                            │
│    /users/*          → User Service                                │
│  Why: 4 backend services — one enforcement point for all rules     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  APPLICATION SERVERS — Microservices (FastAPI, Python)             │
│                                                                     │
│  Order Service      → placement, validation, promo logic           │
│  Restaurant Service → menus, hours, availability                   │
│  Delivery Service   → rider assignment, live tracking              │
│  User Service       → auth, profiles, loyalty points              │
│  Notification Fn    → Azure Functions, event-triggered, stateless  │
│  Invoice Fn         → Azure Functions, fires on order completion   │
│                                                                     │
│  Why microservices: payment failures must not crash menu browsing  │
│  Why functions: notifications are burst, event-driven, zero idle   │
└─────────────────────────────────────────────────────────────────────┘
         │            │             │             │
         ▼            ▼             ▼             ▼
┌──────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐
│    CACHE     │ │  QUEUE   │ │ STORAGE  │ │   SEARCH     │
│              │ │          │ │          │ │              │
│ Azure Cache  │ │  Azure   │ │  Azure   │ │   Azure      │
│ for Redis    │ │ Service  │ │  Blob    │ │  Cognitive   │
│              │ │   Bus    │ │ Storage  │ │   Search     │
│ Restaurant   │ │          │ │          │ │              │
│ menus: 5min  │ │ Topics:  │ │restaurant│ │ Indexes:     │
│ User session │ │ order.   │ │ -photos  │ │ restaurants  │
│ TTL: 30min   │ │ placed   │ │ receipts │ │ menu_items   │
│ Active rider │ │ payment. │ │ invoices │ │              │
│ loc: 10sec   │ │ processed│ │ promos   │ │ Typo-tolerant│
│              │ │ restaurant│ │          │ │ Geo-filtered │
│              │ │ .unavail │ │ CDN      │ │ Ranked by    │
│              │ │          │ │ origin   │ │ distance +   │
│              │ │ Workers: │ │ for all  │ │ rating       │
│              │ │ Payment  │ │ images   │ │              │
│              │ │ Notif    │ │          │ │              │
│              │ │ Invoice  │ │          │ │              │
└──────────────┘ └──────────┘ └──────────┘ └──────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  DATABASE — Azure Database for PostgreSQL                          │
│                                                                     │
│  Primary DB        → all write operations                          │
│    orders, order_items, payments, users, restaurants               │
│                                                                     │
│  Read Replica 1    → order history queries, customer-facing reads  │
│  Read Replica 2    → ops dashboard, analytics, reporting queries   │
│                                                                     │
│  Why two replicas: dashboard queries must never compete with       │
│  customer-facing reads during lunch peak                           │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  EXTERNAL SERVICES                                                  │
│                                                                     │
│  GCash API          → payment processing (PH market)               │
│  PayMaya API        → payment processing (PH market)               │
│  Google Maps        → routing, distance, geocoding, live map       │
│  Firebase FCM       → push notifications to mobile apps            │
│  Twilio             → SMS fallback when push fails                 │
│  SendGrid           → email receipts and marketing                 │
│                                                                     │
│  All wrapped in Circuit Breaker + Retry + Fallback                 │
│  GCash down → PayMaya fallback → show "pay on delivery" option     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  OBSERVABILITY                                                      │
│                                                                     │
│  Azure Application Insights → distributed tracing, APM            │
│  Azure Monitor              → metrics dashboards, custom alerts    │
│  Log Analytics Workspace    → centralized structured JSON logs     │
│  PagerDuty                  → on-call escalation                   │
│                                                                     │
│  Alert rules:                                                       │
│    Order Service p95 latency > 2s → page on-call engineer          │
│    Payment error rate > 2%        → page payments lead             │
│    GCash circuit breaker open     → Slack + PagerDuty              │
│                                                                     │
│  Correlation ID: every order traced from mobile tap                │
│  all the way through 6 services back to the confirmation           │
└─────────────────────────────────────────────────────────────────────┘
```

**Key architectural decisions for FoodNow:**

| Layer | Decision | Why |
|---|---|---|
| Deployment | Microservices | Payment failures must not crash order browsing |
| Functions | Notification + Invoice as serverless | Burst, event-triggered, zero idle cost |
| Queue | Azure Service Bus Topics | Restaurant unavailable must fan-out to 3+ services simultaneously |
| Queue | Azure Service Bus Queue | Payment processing must be exactly-once, never double-charged |
| Search | Dedicated search engine | Menu search needs typo tolerance and geo-ranking — DB LIKE cannot do this |
| DB | Two read replicas | Dashboard queries must never compete with customer reads at peak |
| External | Circuit Breaker on all | GCash outage during holiday sale must degrade gracefully, not crash |

---

### 7.2 Analytics Migrator — Ideal Future Architecture

**System context:** An enterprise internal tool used by data engineering teams to migrate stored procedures from legacy databases (SQL Server, Oracle, PostgreSQL) into Microsoft Fabric and dbt models. Used by engineers, data platform leads, and client stakeholders. Multiple concurrent client engagements requiring full isolation.

**Architect's reasoning:** This is an internal tool today but it needs to become a multi-tenant, enterprise-grade platform. It runs long orchestration pipelines (30-60 minutes), makes hundreds of LLM calls per run, deploys artifacts to Microsoft Fabric, and needs to be trusted by client stakeholders with their production data. The architecture must reflect that trust — security, observability, isolation, and reliability are non-negotiable at enterprise scale.

**Important note:** This is the ideal future state — the fully realized version after all restructuring tiers and feature phases are complete. Not the current state.

```
┌─────────────────────────────────────────────────────────────────────┐
│  CLIENTS                                                            │
│  Web Browser (Vue.js SPA)  → engineers, data platform leads        │
│  CLI (Python binary)       → CI/CD pipelines, scheduled jobs       │
│                               headless execution, scripted runs    │
│                                                                     │
│  Why CLI matters: enterprise workflows need headless execution     │
│  CI/CD cannot click buttons — it needs a programmatic entry point  │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  CDN / EDGE — Azure Static Web Apps (built-in CDN)                 │
│  What it serves: Vue.js app bundle, CSS, static assets             │
│  Why: internal tool — no global edge needed                        │
│  Azure Static Web Apps gives CDN for free with built-in hosting    │
│  Not a priority — a simple Nginx would also work at this scale     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LOAD BALANCER — Azure Application Gateway                         │
│  Routes to multiple API instances                                  │
│  SSL termination                                                   │
│  WAF (Web Application Firewall) enabled                            │
│  Why: multi-tenant with client data — WAF is non-negotiable        │
│  Auto-scale: when multiple teams run pipelines concurrently        │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  API GATEWAY — Azure API Management                                │
│                                                                     │
│  Auth:   Azure AD / Entra ID SSO — JWT tokens                     │
│          API Key auth for CLI and CI/CD programmatic access        │
│  RBAC:   Admin (full), Engineer (run pipelines), Viewer (read)     │
│  Scoping: every API call scoped to requesting user's projects      │
│  Rate limiting: prevent runaway automation scripts                 │
│                                                                     │
│  Why: current Basic Auth is a shared credential — zero identity    │
│  Every action must be traceable to a named user for audit trail    │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  APPLICATION SERVERS                                                │
│                                                                     │
│  FastAPI Backend (stateless, ×2 instances)                         │
│    → Project management, validation, scoring, report endpoints     │
│    → Publishes migration jobs to Service Bus queue                 │
│    → Never runs pipelines directly — delegates to Worker           │
│                                                                     │
│  Migration Worker (dedicated process, ×N instances)                │
│    → Was: background daemon thread (Tier 1 restructuring)          │
│    → Now: dedicated worker consuming from Service Bus              │
│    → Runs: fix loops, improve loops, wave runner, dbt execution    │
│    → Scales independently from API — add workers during big runs   │
│                                                                     │
│  Why separate: a 60-minute pipeline cannot share a process         │
│  with the API server — they compete for CPU and memory             │
│  Worker crash must not take down the UI                            │
└─────────────────────────────────────────────────────────────────────┘
         │            │             │
         ▼            ▼             ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────────────────────┐
│    CACHE     │ │    QUEUE     │ │         STORAGE              │
│              │ │              │ │                              │
│ Azure Cache  │ │ Azure        │ │  Azure Blob Storage          │
│ for Redis    │ │ Service Bus  │ │                              │
│              │ │              │ │  migration-artifacts/        │
│ SSE event    │ │ Queue:       │ │   → generated dbt models     │
│ subscriptions│ │ migration-   │ │   → dbt project zips         │
│ (replaces    │ │ jobs         │ │                              │
│ in-memory    │ │              │ │  validation-reports/         │
│ EventBus)    │ │ Worker picks │ │   → HTML/PDF exports         │
│              │ │ up job →     │ │   → stakeholder deliverables │
│ Runtime      │ │ runs pipeline│ │                              │
│ config       │ │              │ │  project-archives/           │
│ overrides    │ │ DLQ:         │ │   → import/export packages   │
│ (replaces    │ │ failed jobs  │ │                              │
│ in-memory    │ │ captured,    │ │  llm-call-logs/              │
│ dict)        │ │ not silently │ │   → full prompt + response   │
│              │ │ dropped      │ │   → cost tracking per run    │
│ LLM activity │ │              │ │                              │
│ ring buffer  │ │ Why: pipeline│ │  Why blob: generated dbt     │
│ TTL-based    │ │ must survive │ │  files must survive deploys  │
│              │ │ API restarts │ │  and be shareable across     │
│              │ │              │ │  team members                │
└──────────────┘ └──────────────┘ └──────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  DATABASE — Azure Database for PostgreSQL                          │
│  (replaces SQLite — SQLite cannot support multi-user or HA)        │
│                                                                     │
│  Primary DB        → project writes, task writes, event writes     │
│    Tables: projects, tasks, agent_events, llm_calls,               │
│            source_objects, target_models, validation_runs          │
│                                                                     │
│  Read Replica      → validation score queries, dashboard reads     │
│                      reporting endpoints, export generation        │
│                                                                     │
│  Why PostgreSQL: multi-user requires proper connection pooling,    │
│  concurrent writes, and row-level locking — SQLite is single-writer│
│                                                                     │
│  Per-client isolation: each client project scoped by project_id   │
│  Row-level security enforced — Client A cannot see Client B data   │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  EXTERNAL SERVICES                                                  │
│                                                                     │
│  Azure OpenAI           → LLM calls (GPT-4o, GPT-4o-mini)         │
│    Circuit Breaker:     3 failures → 60s cooldown → probe → resume │
│    Multi-model routing: simple tasks → GPT-4o-mini (cheap+fast)   │
│                         complex SQL  → GPT-4o (accurate+thorough) │
│    Cost cap: $20 per pipeline run, tracked in real time            │
│                                                                     │
│  Microsoft Fabric       → target deployment for migrated models    │
│    dbt-fabric adapter wired into dbt runner                        │
│    Workspace management, incremental deploy, rollback on failure   │
│                                                                     │
│  Source Databases       → client legacy systems                    │
│    PostgreSQL (current), SQL Server (Phase 3), Oracle (Phase 3)   │
│    Connection retry: 3 attempts with exponential backoff           │
│    Credentials: pulled from Azure Key Vault, never in code         │
│                                                                     │
│  dbt Core               → SQL transformation execution engine      │
│    Version pinned: dbt-core==1.9.1 (never floating >=1.7)         │
│    Runs inside Worker process, not API server                      │
│                                                                     │
│  Azure AD / Entra ID    → SSO identity provider                   │
│    Engineers log in with their org credentials                     │
│    No separate account management required                         │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  OBSERVABILITY                                                      │
│                                                                     │
│  Azure Application Insights → distributed tracing                 │
│    Correlation ID: every pipeline run traced from API trigger      │
│    through Worker → LLM calls → dbt execution → validation         │
│    One run ID connects everything end to end                       │
│                                                                     │
│  Azure Monitor + Log Analytics → structured JSON logs             │
│    Fields: project_id, phase, agent, model_name, iteration        │
│    Query: "show all logs for project X from last run"             │
│    Currently: plain text logs — zero queryability in production    │
│                                                                     │
│  Prometheus + Grafana → custom metrics                            │
│    LLM calls per minute, success rate, cost per run               │
│    Validation score trends per project                             │
│    dbt run duration by model complexity                            │
│    Circuit breaker state (open/closed/half-open)                  │
│                                                                     │
│  Alerts:                                                           │
│    LLM error rate > 5%            → Slack + email to engineer     │
│    Pipeline run cost > $18        → warning before $20 cap        │
│    Circuit breaker open           → immediate Slack notification  │
│    Validation score drop > 20%    → flag for human review         │
└─────────────────────────────────────────────────────────────────────┘
```

**Key architectural decisions for the Analytics Migrator ideal state:**

| Layer | Decision | Why |
|---|---|---|
| Client | CLI as first-class client | Enterprise CI/CD cannot click buttons — programmatic entry point is required |
| API Gateway | Azure AD SSO + API Keys | Every action must trace to a named user — shared Basic Auth has no identity |
| App Server | Worker as separate process | 60-minute pipelines cannot share a process with the API server |
| Cache | Redis replaces in-memory dicts | In-memory state dies on restart and cannot be shared across multiple instances |
| Queue | Azure Service Bus + DLQ | Pipeline jobs must survive API restarts and failed jobs must never silently disappear |
| Storage | Azure Blob for all artifacts | Generated dbt files on the app server disk get wiped on every deployment |
| Database | PostgreSQL replaces SQLite | SQLite is single-writer — cannot support concurrent users or horizontal scaling |
| External | Circuit Breaker on Azure OpenAI | LLM outage during a 50-model pipeline must fail fast, not hang for 100 minutes |
| External | Multi-model routing | Paying GPT-4o rates for simple column renames wastes 30-50% of LLM budget |
| External | Key Vault for all credentials | Client database credentials in environment variables is the highest-risk gap |
| Observability | Correlation IDs across all layers | Without tracing a request across API → Worker → LLM → dbt, debugging is impossible |

---

### 7.3 Side-by-Side Comparison

The same reference topology. Two completely different architectures. Both justified by different context, scale, and domain requirements.

```
LAYER              FOODNOW                        ANALYTICS MIGRATOR
────────────────────────────────────────────────────────────────────────
Client             Mobile apps + Web + Rider app  Web SPA + CLI
CDN                Azure CDN (restaurant photos)  Azure Static Web Apps
Load Balancer      App Gateway + auto-scale ×5   App Gateway + WAF
API Gateway        Azure APIM + B2C JWT          Azure APIM + Entra SSO + API Keys
App Server         Microservices per domain       FastAPI + dedicated Worker process
Cache              Redis (menus, sessions, GPS)   Redis (SSE, config, activity buffer)
Queue              Service Bus (payment + fan-out) Service Bus (migration jobs + DLQ)
Storage            Blob (photos, receipts)        Blob (dbt artifacts, reports, archives)
Search Engine      Azure Cognitive Search         Not needed at this scale
Database           PostgreSQL + 2 read replicas   PostgreSQL + 1 read replica
External Services  GCash, Maps, Firebase, Twilio  Azure OpenAI, Fabric, dbt, source DBs
Observability      App Insights + PagerDuty       App Insights + Prometheus + Grafana
────────────────────────────────────────────────────────────────────────
Missing layers     None — full maturity           No search engine (not needed)
Scale              Consumer, thousands concurrent Internal, tens concurrent
Biggest risk       GCash outage during peak       LLM outage mid-pipeline
Biggest gap        None — fully built             SQLite in prod, no identity, no tracing
```

The topology did not change. The questions asked at each layer did not change. The answers — the technology, the scale, the domain specifics, the justifications — are entirely different because the two systems serve entirely different purposes.

That is exactly how the model works.

---

*Adrian Arante — Universal System Topology Reference*
*Architecture Thinking Model Series*
