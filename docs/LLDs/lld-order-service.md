# 📦 Low-Level Design – Order Service

This document provides the Low-Level Design (LLD) for the Order Service in the e-commerce microservices architecture. The Order Service handles order placement, tracking, and order history.

---

## 🧠 Core Responsibilities

- Accept order placement requests
- Validate cart and user authentication
- Reserve inventory
- Initiate payment
- Generate order ID and persist order
- Expose order history and status APIs
- Handle async updates (payment success/failure, delivery)

---

## 📐 Architecture Diagram

```
Client → API Gateway → Order Service
                         ↓
             [Inventory Service]
             [Payment Service]
             [Notification Service]
```

---

## 📁 Directory Structure

```
/order-service
│
├── controller/             # REST API layer
├── service/                # Business logic
├── repository/             # DB access (JPA or raw queries)
├── model/                  # DTOs and entities
├── config/                 # App configs, schedulers
└── events/                 # Kafka event producers/consumers
```

---

## 📄 API Endpoints

### 1. Create Order

- `POST /api/orders`
```json
{
  "userId": "123",
  "items": [
    { "productId": "abc", "quantity": 2 },
    { "productId": "xyz", "quantity": 1 }
  ],
  "shippingAddress": "Bhubaneswar",
  "paymentMethod": "card"
}
```

- Steps:
  1. Validate user & inventory
  2. Create order with status `PENDING`
  3. Publish `OrderCreated` event
  4. Await async updates

---

### 2. Get Order by ID

- `GET /api/orders/{orderId}`

Returns full order details including items, payment, and delivery status.

---

### 3. Get Orders by User

- `GET /api/orders/user/{userId}`

Returns paginated order history.

---

## 🗃️ Database Schema (PostgreSQL)

### `orders`
| Field         | Type        | Description            |
|---------------|-------------|------------------------|
| id            | UUID (PK)   | Order ID               |
| user_id       | UUID        | FK to user             |
| total_amount  | DECIMAL     | Total order cost       |
| status        | TEXT        | PENDING, CONFIRMED...  |
| created_at    | TIMESTAMP   | Order placed timestamp |

### `order_items`
| Field         | Type        | Description             |
|---------------|-------------|-------------------------|
| id            | UUID (PK)   |                         |
| order_id      | UUID (FK)   | Parent order            |
| product_id    | UUID        | FK to product service   |
| quantity      | INT         |                         |
| price         | DECIMAL     | Snapshot of product price |

---

## 📤 Kafka Events

### Produced:
- `OrderCreated`
- `OrderConfirmed`
- `OrderCancelled`

### Consumed:
- `PaymentSuccess`
- `PaymentFailed`
- `InventoryReserved`
- `InventoryFailure`

---

## ⚙️ Internal Modules

### InventoryValidator
- Calls Inventory service to lock stock
- Retries on failure
- Circuit breaker fallback

### PaymentInitiator
- Calls Payment service
- Timeout + retry logic
- Updates order status accordingly

### OrderEventHandler
- Listens to Kafka for updates
- Updates DB status fields (confirmed, failed, etc.)

---

## 🔐 Security

- JWT-based auth
- Only authenticated users can create or view orders
- Admin role can view all orders

---

## 🧪 Testing

- Unit: OrderService logic, validations
- Integration: OrderController with mock dependencies
- Contract: API schemas for consumers
- E2E: Full order creation with mocks for inventory/payment

---

## 🛠 Design Decisions

- Asynchronous communication improves responsiveness
- Order service is source of truth for order status
- Final consistency model (eventual updates after creation)
- Idempotency keys used for order creation to avoid duplicates

---

## 🧠 Future Improvements

- Add order cancellation and return modules
- Introduce order status timeline (delivered, shipped, etc.)
- Add support for multiple payment retries
- Split read/write models (CQRS) for scale

```

# End of LLD