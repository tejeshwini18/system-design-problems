# System Design – HLD & LLD Index

This folder contains **High-Level Design (HLD)** and **Low-Level Design (LLD)** documents for common system design interview topics and **company/product case studies**.

**Standard process (every design):**  
**Step 1:** Clarify Requirements (functional / non-functional, constraints).  
**Step 2:** High-Level Design (components, interfaces, data flow, flow diagram).  
**Step 3:** Detailed Design (database SQL/NoSQL, API design, full API endpoints).  
**Step 4:** Scale & Optimize (load balancing, sharding, caching).  

See [System Design Template](SYSTEM_DESIGN_TEMPLATE.md) for the document structure. Every design includes **both**:
- **High-Level Architecture** — component diagram (boxes and connections).
- **Flow Diagram** — sequence or process flow for the main use case.

Diagrams use **Mermaid** (renders in GitHub/IDE) and optionally **Eraser** (paste code into [Eraser.io](https://app.eraser.io)).

---

## Part 1: Common System Design Topics (1–17)

| # | System | HLD | LLD |
|---|--------|-----|-----|
| 1 | [Scalable Chat Application](01-scalable-chat-application/) (WhatsApp/Slack) | [HLD](01-scalable-chat-application/HLD.md) | [LLD](01-scalable-chat-application/LLD.md) |
| 2 | [URL Shortener](02-url-shortener/) (Bitly/TinyURL) | [HLD](02-url-shortener/HLD.md) | [LLD](02-url-shortener/LLD.md) |
| 3 | [API Rate Limiter](03-api-rate-limiter/) | [HLD](03-api-rate-limiter/HLD.md) | [LLD](03-api-rate-limiter/LLD.md) |
| 4 | [Distributed Cache System](04-distributed-cache/) | [HLD](04-distributed-cache/HLD.md) | [LLD](04-distributed-cache/LLD.md) |
| 5 | [Notification System](05-notification-system/) (Email/SMS/Push) | [HLD](05-notification-system/HLD.md) | [LLD](05-notification-system/LLD.md) |
| 6 | [Ride-Hailing System](06-ride-hailing/) (Uber/Ola) | [HLD](06-ride-hailing/HLD.md) | [LLD](06-ride-hailing/LLD.md) |
| 7 | [Video Streaming Platform](07-video-streaming/) (YouTube/Netflix) | [HLD](07-video-streaming/HLD.md) | [LLD](07-video-streaming/LLD.md) |
| 8 | [Payment Gateway](08-payment-gateway/) (Stripe/Razorpay) | [HLD](08-payment-gateway/HLD.md) | [LLD](08-payment-gateway/LLD.md) |
| 9 | [News Feed System](09-news-feed/) | [HLD](09-news-feed/HLD.md) | [LLD](09-news-feed/LLD.md) |
| 10 | [Search Autocomplete](10-search-autocomplete/) | [HLD](10-search-autocomplete/HLD.md) | [LLD](10-search-autocomplete/LLD.md) |
| 11 | [Logging & Monitoring](11-logging-monitoring/) | [HLD](11-logging-monitoring/HLD.md) | [LLD](11-logging-monitoring/LLD.md) |
| 12 | [File Storage System](12-file-storage/) (Google Drive) | [HLD](12-file-storage/HLD.md) | [LLD](12-file-storage/LLD.md) |
| 13 | [E-commerce Checkout](13-ecommerce-checkout/) | [HLD](13-ecommerce-checkout/HLD.md) | [LLD](13-ecommerce-checkout/LLD.md) |
| 14 | [Leaderboard System](14-leaderboard/) | [HLD](14-leaderboard/HLD.md) | [LLD](14-leaderboard/LLD.md) |
| 15 | [Distributed Job Scheduler](15-distributed-job-scheduler/) | [HLD](15-distributed-job-scheduler/HLD.md) | [LLD](15-distributed-job-scheduler/LLD.md) |
| 16 | [Feature Flag System](16-feature-flag/) | [HLD](16-feature-flag/HLD.md) | [LLD](16-feature-flag/LLD.md) |
| 17 | [Metrics & Analytics Platform](17-metrics-analytics/) | [HLD](17-metrics-analytics/HLD.md) | [LLD](17-metrics-analytics/LLD.md) |

---

## Part 2: Case Studies (Company / Product Deep Dives) (18–29)

| # | Case Study | HLD | LLD |
|---|------------|-----|-----|
| 18 | [YouTube: 2.49B Users With MySQL](18-case-youtube-mysql/) | [HLD](18-case-youtube-mysql/HLD.md) | [LLD](18-case-youtube-mysql/LLD.md) |
| 19 | [Uber: Nearby Drivers at 1M RPS](19-case-uber-nearby-drivers/) | [HLD](19-case-uber-nearby-drivers/HLD.md) | [LLD](19-case-uber-nearby-drivers/LLD.md) |
| 20 | [AWS: Scale App to 10M Users](20-case-aws-scale-10m/) | [HLD](20-case-aws-scale-10m/HLD.md) | [LLD](20-case-aws-scale-10m/LLD.md) |
| 21 | [Meta: 11.5M Serverless Calls/s](21-case-meta-serverless/) | [HLD](21-case-meta-serverless/HLD.md) | [LLD](21-case-meta-serverless/LLD.md) |
| 22 | [Google Docs: Real-Time Collaboration](22-case-google-docs/) | [HLD](22-case-google-docs/HLD.md) | [LLD](22-case-google-docs/LLD.md) |
| 23 | [Bluesky: Decentralized Social](23-case-bluesky/) | [HLD](23-case-bluesky/HLD.md) | [LLD](23-case-bluesky/LLD.md) |
| 24 | [Slack: Real-Time Messaging](24-case-slack/) | [HLD](24-case-slack/HLD.md) | [LLD](24-case-slack/LLD.md) |
| 25 | [Spotify: Music Streaming](25-case-spotify/) | [HLD](25-case-spotify/HLD.md) | [LLD](25-case-spotify/LLD.md) |
| 26 | [ChatGPT: How LLMs Work](26-case-chatgpt-llm/) | [HLD](26-case-chatgpt-llm/HLD.md) | [LLD](26-case-chatgpt-llm/LLD.md) |
| 27 | [Twitter: Timeline](27-case-twitter-timeline/) | [HLD](27-case-twitter-timeline/HLD.md) | [LLD](27-case-twitter-timeline/LLD.md) |
| 28 | [Reddit: Communities & Voting](28-case-reddit/) | [HLD](28-case-reddit/HLD.md) | [LLD](28-case-reddit/LLD.md) |
| 29 | [Apache Kafka: Distributed Streaming](29-case-apache-kafka/) | [HLD](29-case-apache-kafka/HLD.md) | [LLD](29-case-apache-kafka/LLD.md) |

---

## Part 3: Additional Important Topics (30–37)

| # | System | HLD | LLD |
|---|--------|-----|-----|
| 30 | [Distributed ID Generator](30-distributed-id-generator/) (Snowflake) | [HLD](30-distributed-id-generator/HLD.md) | [LLD](30-distributed-id-generator/LLD.md) |
| 31 | [Load Balancer](31-load-balancer/) | [HLD](31-load-balancer/HLD.md) | [LLD](31-load-balancer/LLD.md) |
| 32 | [Design Instagram](32-instagram/) | [HLD](32-instagram/HLD.md) | [LLD](32-instagram/LLD.md) |
| 33 | [Search Engine](33-search-engine/) (Google-style) | [HLD](33-search-engine/HLD.md) | [LLD](33-search-engine/LLD.md) |
| 34 | [Web Crawler](34-web-crawler/) | [HLD](34-web-crawler/HLD.md) | [LLD](34-web-crawler/LLD.md) |
| 35 | [Distributed Lock](35-distributed-lock/) | [HLD](35-distributed-lock/HLD.md) | [LLD](35-distributed-lock/LLD.md) |
| 36 | [CDN](36-cdn/) (Content Delivery Network) | [HLD](36-cdn/HLD.md) | [LLD](36-cdn/LLD.md) |
| 37 | [Database Sharding](37-database-sharding/) | [HLD](37-database-sharding/HLD.md) | [LLD](37-database-sharding/LLD.md) |

---

## Part 4: LLD & Design Patterns (38–59)

Every design has both **HLD** and **LLD** (or README for 38).

| # | Topic | HLD | LLD |
|---|--------|-----|-----|
| 38 | [SOLID & Design Patterns](38-design-patterns-solid/) (Strategy, Observer, Factory, etc.) | [HLD](38-design-patterns-solid/HLD.md) | [README](38-design-patterns-solid/README.md) |
| 39 | [Notify-Me Button](39-lld-notify-me/) (Observer) | [HLD](39-lld-notify-me/HLD.md) | [LLD](39-lld-notify-me/LLD.md) |
| 40 | [Pizza Billing System](40-lld-pizza-billing/) (Decorator, Builder) | [HLD](40-lld-pizza-billing/HLD.md) | [LLD](40-lld-pizza-billing/LLD.md) |
| 41 | [Parking Lot](41-lld-parking-lot/) | [HLD](41-lld-parking-lot/HLD.md) | [LLD](41-lld-parking-lot/LLD.md) |
| 42 | [Snake & Ladder](42-lld-snake-ladder/) | [HLD](42-lld-snake-ladder/HLD.md) | [LLD](42-lld-snake-ladder/LLD.md) |
| 43 | [Elevator System](43-lld-elevator/) (State) | [HLD](43-lld-elevator/HLD.md) | [LLD](43-lld-elevator/LLD.md) |
| 44 | [Car Rental](44-lld-car-rental/) (State, Factory) | [HLD](44-lld-car-rental/HLD.md) | [LLD](44-lld-car-rental/LLD.md) |
| 45 | [Logging System (LLD)](45-lld-logging/) (Chain of Responsibility) | [HLD](45-lld-logging/HLD.md) | [LLD](45-lld-logging/LLD.md) |
| 46 | [Tic-Tac-Toe](46-lld-tic-tac-toe/) | [HLD](46-lld-tic-tac-toe/HLD.md) | [LLD](46-lld-tic-tac-toe/LLD.md) |
| 47 | [Chess Game](47-lld-chess/) | [HLD](47-lld-chess/HLD.md) | [LLD](47-lld-chess/LLD.md) |
| 48 | [File System](48-lld-file-system/) (Composite) | [HLD](48-lld-file-system/HLD.md) | [LLD](48-lld-file-system/LLD.md) |
| 49 | [Splitwise & Simplify Algorithm](49-lld-splitwise/) | [HLD](49-lld-splitwise/HLD.md) | [LLD](49-lld-splitwise/LLD.md) |
| 50 | [Vending Machine](50-lld-vending-machine/) (State) | [HLD](50-lld-vending-machine/HLD.md) | [LLD](50-lld-vending-machine/LLD.md) |
| 51 | [ATM](51-lld-atm/) (State) | [HLD](51-lld-atm/HLD.md) | [LLD](51-lld-atm/LLD.md) |
| 52 | [BookMyShow & Concurrency](52-lld-bookmyshow/) | [HLD](52-lld-bookmyshow/HLD.md) | [LLD](52-lld-bookmyshow/LLD.md) |
| 53 | [Library Management System](53-lld-library-management/) | [HLD](53-lld-library-management/HLD.md) | [LLD](53-lld-library-management/LLD.md) |
| 54 | [Traffic Light System](54-lld-traffic-light/) (State) | [HLD](54-lld-traffic-light/HLD.md) | [LLD](54-lld-traffic-light/LLD.md) |
| 55 | [Meeting Scheduler](55-lld-meeting-scheduler/) | [HLD](55-lld-meeting-scheduler/HLD.md) | [LLD](55-lld-meeting-scheduler/LLD.md) |
| 56 | [Online Voting System](56-lld-online-voting/) | [HLD](56-lld-online-voting/HLD.md) | [LLD](56-lld-online-voting/LLD.md) |
| 57 | [Inventory Management System](57-lld-inventory-management/) | [HLD](57-lld-inventory-management/HLD.md) | [LLD](57-lld-inventory-management/LLD.md) |
| 58 | [Restaurant Management System](58-lld-restaurant-management/) | [HLD](58-lld-restaurant-management/HLD.md) | [LLD](58-lld-restaurant-management/LLD.md) |
| 59 | [Bowling Alley Machine](59-lld-bowling-alley/) | [HLD](59-lld-bowling-alley/HLD.md) | [LLD](59-lld-bowling-alley/LLD.md) |

---

## Part 5: More System Designs (60–64)

| # | System | HLD | LLD |
|---|--------|-----|-----|
| 60 | [Pastebin](60-pastebin/) | [HLD](60-pastebin/HLD.md) | [LLD](60-pastebin/LLD.md) |
| 61 | [Ticketmaster](61-ticketmaster/) | [HLD](61-ticketmaster/HLD.md) | [LLD](61-ticketmaster/LLD.md) |
| 62 | [Hotel Booking](62-hotel-booking/) | [HLD](62-hotel-booking/HLD.md) | [LLD](62-hotel-booking/LLD.md) |
| 63 | [Food Delivery](63-food-delivery/) (Swiggy/Zomato) | [HLD](63-food-delivery/HLD.md) | [LLD](63-food-delivery/LLD.md) |
| 64 | [LMS & Calendar](64-lms-calendar/) | [HLD](64-lms-calendar/HLD.md) | [LLD](64-lms-calendar/LLD.md) |

---

## Part 6: HLD Concepts (Theory & When to Use)

| Topic | Doc |
|-------|-----|
| [Network Protocols](65-hld-concepts/README.md#1-network-protocols) (TCP, WebSocket, HTTP) | [HLD Concepts](65-hld-concepts/README.md) |
| [Client-Server vs P2P](65-hld-concepts/README.md#2-client-server-vs-p2p) | |
| [CAP Theorem](65-hld-concepts/README.md#3-cap-theorem) | |
| [Microservices (SAGA, Strangler)](65-hld-concepts/README.md#4-microservices-patterns) | |
| [Scale 0 to Million](65-hld-concepts/README.md#5-scale-0-to-million-users-phases) | |
| [Consistent Hashing](65-hld-concepts/README.md#6-consistent-hashing) | |
| [Back-of-Envelope](65-hld-concepts/README.md#7-back-of-envelope-estimation) | |
| [SQL vs NoSQL](65-hld-concepts/README.md#8-sql-vs-nosql--when-to-use) | |
| [Message Queue / Kafka](65-hld-concepts/README.md#9-message-queue--kafka-when-to-use) | |
| [Proxy, CDN](65-hld-concepts/README.md#10-11) | |
| [Storage Types](65-hld-concepts/README.md#12-storage-types) (Block, File, Object, RAID) | |
| [GFS / HDFS](65-hld-concepts/README.md#13-file-system-gfs-hdfs) | |
| [Bloom Filter](65-hld-concepts/README.md#14-bloom-filter) | |
| [Merkle Tree, Gossip](65-hld-concepts/README.md#15-merkle-tree--gossip) | |
| [Caching](65-hld-concepts/README.md#16-caching) (invalidation, eviction) | |
| [Scale DB](65-hld-concepts/README.md#17-scaling-database), [Indexing](65-hld-concepts/README.md#18-indexing) | |

---

**HLD** = High-Level Design (components, data flow, scaling, trade-offs)  
**LLD** = Low-Level Design (APIs, schemas, algorithms, class/module design)
