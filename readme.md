
# 🛒 Amazon-Scale E-Commerce Platform – Polyglot Microservices Architecture

This project is a full-fledged, scalable, polyglot **e-commerce backend system**, architected using **Spring Boot**, **Node.js**, **Docker**, and **Kubernetes** for cloud-native, distributed, and highly available deployments. Inspired by large-scale e-commerce systems, this project is a fun and evolving playground for exploring modern backend architecture, distributed systems, and machine learning integration.

---

## 📘 Table of Contents

- [🛒 Amazon-Scale E-Commerce Platform – Polyglot Microservices Architecture](#-amazon-scale-e-commerce-platform--polyglot-microservices-architecture)
  - [📘 Table of Contents](#-table-of-contents)
  - [🧠 Overview](#-overview)
  - [🛠 Tech Stack](#-tech-stack)
  - [🧩 Microservices](#-microservices)
  - [🔧 Architecture](#-architecture)
  - [🧭 **End-to-End Project Flow:**](#-end-to-end-project-flow)
    - [🛒 1. **User Browses Products**](#-1-user-browses-products)
    - [👤 2. **User Registers / Logs In**](#-2-user-registers--logs-in)
    - [🧺 3. **User Adds Items to Cart**](#-3-user-adds-items-to-cart)
    - [🧾 4. **User Places an Order**](#-4-user-places-an-order)
    - [🧠 5. **Backend Reacts to Order Event (Kafka Flow)**](#-5-backend-reacts-to-order-event-kafka-flow)
    - [🤖 6. **ML Service (Optional Calls or Events)**](#-6-ml-service-optional-calls-or-events)
    - [📊 7. **Analytics Service**](#-7-analytics-service)
    - [🔐 8. **Security \& Role-Based Access**](#-8-security--role-based-access)
    - [📈 9. **Monitoring, Logging, and Tracing**](#-9-monitoring-logging-and-tracing)
  - [🌐 Summary of Tech Roles in Flow:](#-summary-of-tech-roles-in-flow)
  - [🚀 Key Features](#-key-features)
  - [🌍 High Availability \& Scalability](#-high-availability--scalability)
  - [🔐 Security](#-security)
  - [📊 Monitoring \& Observability](#-monitoring--observability)
  - [⚙️ Setup \& Deployment](#️-setup--deployment)
  - [📈 Load Testing \& Benchmarking](#-load-testing--benchmarking)
  - [🔁 DevOps \& CI/CD](#-devops--cicd)
  - [🤖 AI/ML Integration](#-aiml-integration)
    - [📦 `ml-service` (Python + FastAPI)](#-ml-service-python--fastapi)
    - [🔍 Features Provided by ML Service](#-features-provided-by-ml-service)
    - [⚙️ Technologies Used](#️-technologies-used)
    - [📁 Suggested Directory Structure](#-suggested-directory-structure)
  - [🤝 Contributions](#-contributions)
  - [📄 License](#-license)
  - [📬 Contact](#-contact)

---

## 🧠 Overview

> This project simulates an Amazon.com-like e-commerce backend ecosystem using loosely coupled, highly cohesive microservices. Built with a **polyglot architecture**, it leverages **Java** for transactional services and **Node.js** for high-throughput, asynchronous tasks.

---

## 🛠 Tech Stack

| Layer                 | Tech                                                  |
|----------------------|-------------------------------------------------------|
| Backend               | Java (Spring Boot), Node.js (Express.js), Python (FastAPI for analytics) |
| API Gateway           | Spring Cloud Gateway, NGINX                          |
| Databases             | MySQL, PostgreSQL, MongoDB, Redis                     |
| Messaging/Eventing    | Kafka, RabbitMQ                                       |
| Caching               | Redis, Local LRU (for Node.js)                        |
| Authentication        | JWT (Node.js), Spring Security + OAuth2 (Java)       |
| Deployment            | Docker, Kubernetes, Helm                             |
| CI/CD                 | GitHub Actions, Jenkins                              |
| Observability         | ELK (Elasticsearch, Logstash, Kibana), Prometheus, Grafana, Jaeger |
| Testing               | JUnit, Mockito, Jest, Supertest                      |
| Load Testing          | k6, JMeter                                            |
| Cloud Ready           | AWS/GCP/Azure                                         |

---

## 🧩 Microservices

| Microservice         | Language  | Description                                       |
|----------------------|-----------|---------------------------------------------------|
| **User Service**     | Java      | Handles user registration, profile, sessions      |
| **Auth Service**     | Node.js   | Stateless JWT authentication                      |
| **Product Service**  | Java      | Manages catalog, search, filters, categories      |
| **Review Service**   | Node.js   | Handles product reviews, ratings                  |
| **Order Service**    | Java      | Processes orders, history, statuses               |
| **Payment Service**  | Java      | Integrates with Stripe/PayPal gateways            |
| **Inventory Service**| Java      | Tracks stock levels, replenishments               |
| **Notification Service** | Node.js | Sends email/SMS notifications asynchronously     |
| **Analytics Service**| Python    | Tracks user behavior, trends, dashboards, and runs ML/statistical models |
| **API Gateway**      | Node.js or Spring | Routes requests, enforces security        |

Each service has:
- Its **own database**
- **Health checks**
- **Rate limiting**
- **Retry/fallback logic**

---

## 🔧 Architecture


**(ASCII Diagram)**

```
                                   +-------------------------+
                                   |      Web / Mobile UI    |
                                   | (React / Flutter / etc) |
                                   +-----------+-------------+
                                               |
                                      HTTPS / REST API Calls
                                               |
                                  +------------v-------------+
                                  |       API Gateway        |
                                  | (Spring Gateway / NGINX) |
                                  +------------+-------------+
                                               |
        +---------------------------+----------+-----------+-------------------------+
        |                           |                      |                         |
+-------v-------+         +---------v--------+   +---------v--------+       +--------v--------+
| Auth Service  |         |  User Service    |   | Product Service  |       | Review Service  |
|  (Node.js)    |         |   (Java)         |   |   (Java)         |       |   (Node.js)     |
+-------+-------+         +---------+--------+   +---------+--------+       +--------+--------+
        |                           |                      |                         |
        |              +------------v-------------+        |                         |
        |              |       Redis (Sessions)   |        |                         |
        |              +--------------------------+        |                         |
        |                                                  |                         |
+-------v-------+         +---------v--------+   +---------v--------+       +--------v--------+
| Order Service |         | Inventory Service|   | Payment Service  |       | Notification Svc|
|   (Java)      |         |     (Java)       |   |    (Java)        |       |   (Node.js)     |
+-------+-------+         +---------+--------+   +---------+--------+       +--------+--------+
        |                           |                      |                         |
        |              +------------v-------------+        |                         |
        |              | Kafka Event Streaming    |<-------+-------------------------+
        |              +------------+-------------+        |
        |                           |                      |
+-------v-------+         +---------v--------+   +---------v--------+
|  ML Service   |         |  Analytics Svc   |   |     MongoDB       |
| (Python/ML)   |         |   (Python)       |   |   Orders / Reviews|
+---------------+         +------------------+   +-------------------+

```


- **Polyglot Microservices** – Node.js for non-blocking services, Java for high-transactional systems
- **API Gateway** for routing, throttling, auth
- **Kafka-based Event Bus** for decoupling and eventual consistency
- **Elastic scaling** via Kubernetes HPA
- **Distributed Tracing** using Jaeger
- **Rate-limiting and Circuit Breakers** for resilience

---


## 🧭 **End-to-End Project Flow:**


### 🛒 1. **User Browses Products**
- User opens the app or website
- Sends request → `GET /products` to **API Gateway**
- API Gateway routes to **Product Service**
- Product Service queries **PostgreSQL**
- Frequently viewed products may be fetched from **Redis cache**
- Response is sent back with product listings

---

### 👤 2. **User Registers / Logs In**
- User sends `POST /auth/login` with credentials
- API Gateway routes to **Auth Service (Node.js)**
- Auth Service validates credentials against **User Service**
- If valid:
  - A **JWT token** is generated
  - Session data may be cached in **Redis**
- JWT is returned to user for future authenticated requests

---

### 🧺 3. **User Adds Items to Cart**
- Client stores cart locally or communicates with a **Cart Service** (optional)
- This part can be a microservice or handled on client side

---

### 🧾 4. **User Places an Order**
- User clicks “Place Order” → `POST /orders`
- Request hits **API Gateway**, which routes to **Order Service**
- Order Service:
  - Saves order in **MongoDB**
  - Publishes `OrderCreatedEvent` to **Kafka**

---

### 🧠 5. **Backend Reacts to Order Event (Kafka Flow)**

**a. Inventory Service**
- Listens to `OrderCreatedEvent`
- Decreases stock count in **PostgreSQL**
- If stock insufficient, publishes `InventoryFailedEvent`

**b. Payment Service**
- Listens to `OrderCreatedEvent`
- Processes mock payment
- On success/failure, sends `PaymentProcessedEvent`

**c. Notification Service**
- Listens to order/payment events
- Sends email/SMS using **Nodemailer/Twilio**

---

### 🤖 6. **ML Service (Optional Calls or Events)**
- When a user logs in, their ID may be sent to:
  - `GET /recommendations/{userId}` → from **ML Service**
  - ML service uses LightFM/KMeans models to recommend
- Or, events like "product viewed" or "order placed" are sent to Kafka
- ML Service processes them periodically and updates models

---

### 📊 7. **Analytics Service**
- Listens to Kafka events (product views, order placed, user registered)
- Aggregates data:
  - Daily sales, top products, abandoned carts
- Can expose dashboard via:
  - REST APIs (`/stats/top-products`)
  - Grafana (Prometheus metrics)

---

### 🔐 8. **Security & Role-Based Access**
- JWT is passed with every request in `Authorization` header
- API Gateway or downstream services validate it
- Admin users can:
  - Add/edit/delete products
  - View analytics

---

### 📈 9. **Monitoring, Logging, and Tracing**
- Each service emits:
  - Logs → **Logstash → Elasticsearch → Kibana**
  - Metrics → **Prometheus → Grafana**
  - Traces → **Jaeger or Zipkin**
- Helps identify bottlenecks, failed requests, latency, etc.



```sql
                                  +---------------------------+
                                  |      Web / Mobile App     |
                                  +------------+--------------+
                                               |
                                   HTTPS Requests (REST + JWT)
                                               |
                                  +------------v-------------+
                                  |        API Gateway        |
                                  | (Spring Cloud / NGINX)    |
                                  +------------+-------------+
                                               |
     +-----------+----------+-----------+------------+------------+----------+
     |           |          |           |            |            |          |
+----v--+   +----v--+  +----v--+   +-----v----+  +-----v----+  +----v--+  +---v---+
| Auth |   |  User |  |Product|   |  Review  |  |  Order   |  |Payment|  |Inventory|
|Node.js| | Java   |  | Java  |   | Node.js  |  |  Java    |  | Java  |  |  Java   |
+---+---+   +--+---+  +--+---+   +-----+-----+  +-----+----+  +--+----+  +---+----+
    |           |         |            |             |           |         |
    | Redis     |         |   Redis    |             |           |         |
    | (Sessions)|         | (Product)  |             |           |         |
    +-----------+         +------------+             |           |         |
                                                      \___________|_________/
                                                                 |
                                            Kafka Event Stream (OrderCreated, PaymentStatus)
                                                                 |
                            +------------------------+-----------+------------+
                            |                        |                        |
                    +-------v-------+        +-------v--------+       +-------v--------+
                    | Notification  |        |   ML Service   |       |  Analytics Svc |
                    |   Node.js     |        |   Python       |       |    Python      |
                    +---------------+        +----------------+       +----------------+
                    | Email/SMS via          | Recos, Segments        | Metrics, Trends
                    | Kafka + SMTP/Twilio    | Forecast, ETA          | Dashboard API
                    +------------------------+------------------------+----------------
```




## 🌐 Summary of Tech Roles in Flow:

| Component         | Purpose |
|------------------|---------|
| API Gateway       | Entry point, routing, security |
| Node.js Services  | Lightweight services (Auth, Reviews, Notifications) |
| Java Services     | Business logic heavy services (User, Product, Order, Payment, Inventory) |
| Python Services   | AI/ML, analytics, forecasting |
| Kafka             | Event-driven, decoupled communication |
| Redis             | Fast caching for products, sessions, and stock |
| MongoDB           | Flexible schema for orders, reviews |
| SQL Databases     | Structured product, user, and payment data |
| ELK + Prometheus  | Monitoring, logging, alerts |







---

## 🚀 Key Features

- JWT-based Authentication
- Product Search + Filters (by category, price, rating)
- Inventory synchronization via Kafka
- Order lifecycle: Place, Cancel, Track, History
- Payment gateway integration (Stripe/PayPal mock)
- Review system with ratings & abuse checks
- Caching for products, user sessions, and stock levels
- CI/CD pipeline with full code coverage + linting

---

## 🌍 High Availability & Scalability

- Kubernetes with **HPA (Horizontal Pod Autoscaler)** based on CPU/memory
- Stateless services using REST, event-based communication
- **Caching layer (Redis)** for performance optimization
- Rate limiting & retries to ensure graceful degradation
- **Zero downtime deployments** with rolling updates
- **Blue/Green deployment readiness**

---

## 🔐 Security

- OAuth2 + JWT authentication
- Role-based access control (admin, seller, customer)
- HTTPS with TLS termination at gateway level
- SQL/NoSQL injection protection
- Rate limiting + brute-force protection at API Gateway

---

## 📊 Monitoring & Observability

- **Prometheus** + **Grafana** for real-time metrics
- **ELK stack** for centralized logs
- **Jaeger** or **Zipkin** for distributed tracing
- Custom health endpoints for all services
- Alerts for resource spikes, downtime, failed jobs

---

## ⚙️ Setup & Deployment

```bash
# 1. Start Kubernetes locally
minikube start

# 2. Build Docker images for each service
cd ecommerce-user-service
docker build -t user-service:latest .

# 3. Deploy using Helm or YAML
kubectl apply -f k8s/user-service.yaml

# 4. Access via Ingress
http://ecommerce.local/users/register
```

> You can also deploy to cloud using Helm charts + GitHub Actions workflow.

---

## 📈 Load Testing & Benchmarking

> Simulated 10M requests/day using **k6** and **JMeter**

- Average response time (95th percentile): **<250ms**
- System remained stable under 1K concurrent users per pod
- Redis cache reduced DB hits by ~60%
- Kafka-based order handling achieved **100K+ messages/min**

---

## 🔁 DevOps & CI/CD

- GitHub Actions for test → build → Docker → deploy
- Docker images stored in GitHub Container Registry
- Kubernetes manifests + Helm for deployment
- Environment-specific configs using Spring Config Server & dotenv

---


## 🤖 AI/ML Integration

To explore how intelligent features can be embedded in real-world applications, this project includes an optional AI/ML microservice that adds a layer of smart behavior using Python and FastAPI.

### 📦 `ml-service` (Python + FastAPI)

This microservice hosts and serves pre-trained ML models and exposes REST APIs for predictions and recommendations.

### 🔍 Features Provided by ML Service

| Feature                         | Description |
|----------------------------------|-------------|
| 🔮 **Recommendation Engine**     | Suggests products based on user behavior and purchase history |
| 🧠 **Customer Segmentation**     | Clusters users into personas (e.g., high-spenders, window shoppers) |
| 📈 **Sales Forecasting**         | Predicts future product demand using time-series models |
| 🚨 **Fraud Detection**           | Identifies anomalous payment or login activity |
| 💬 **Smart Search (NLP)**        | Enhances product search relevance with intent recognition |
| 📦 **Delivery ETA Prediction**   | Estimates delivery time using historical data and optional external APIs |

### ⚙️ Technologies Used

- **FastAPI** – Lightweight Python web framework
- **scikit-learn**, **XGBoost**, **LightFM**, **Prophet** – ML model training and inference
- **pandas**, **NumPy**, **joblib** – Data processing and model serialization
- **Docker** – Containerized deployment
- **Kafka Integration** – For event-triggered model inference (optional)

### 📁 Suggested Directory Structure

```
ml-service/
├── app/
│   ├── main.py          # FastAPI entry point
│   ├── models/          # ML model files (.pkl, .joblib)
│   ├── services/        # Business logic for model loading/inference
│   └── routers/         # Route definitions for REST APIs
├── requirements.txt
├── Dockerfile
└── README.md
```

> Other microservices (e.g., Product, Order, or User) can call this ML service via REST or trigger it asynchronously using Kafka.


## 🤝 Contributions

We welcome PRs to improve performance, refactor code, or add new features like:
- Product recommendations
- Fraud detection microservice
- Integration with actual payment gateways (Stripe test mode)

---

## 📄 License

This project is licensed under the **Apache 2.0 License**.

---

## 📬 Contact

Maintainer: [Raj Kishor Maharana](mailto:mrajkishor331@gmail.com)  
GitHub: [@mrajkishor](https://github.com/mrajkishor)
