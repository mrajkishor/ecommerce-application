# 💳 Low-Level Design – Payment Service

This document provides the Low-Level Design (LLD) for the Payment Service in the e-commerce microservices platform. It handles payment processing, transaction status tracking, and integration with third-party gateways.

---

## 🧠 Core Responsibilities

- Process payment requests (card, UPI, wallets)
- Validate payment methods and amounts
- Communicate with external payment gateways
- Persist transaction records
- Notify other services about payment status
- Handle retries, timeouts, and failures

---

## 📐 Architecture Diagram

```
Client → API Gateway → Order Service
                            ↓
                    → Payment Service
                            ↓
            [External Payment Gateway (e.g., Stripe, Razorpay)]
```

---

## 📁 Directory Structure

```
/payment-service
│
├── controller/             # REST API layer
├── service/                # Business logic
├── gateway/                # External provider wrappers
├── repository/             # DB access layer
├── model/                  # DTOs and entities
├── config/                 # Secrets, scheduler configs
└── events/                 # Kafka producers and consumers
```

---

## 📄 API Endpoints

### 1. Initiate Payment

- `POST /api/payments`
```json
{
  "orderId": "abc123",
  "userId": "user789",
  "amount": 499.99,
  "method": "card",
  "cardToken": "tok_visa_xyz"
}
```

- Validates user and order
- Calls payment gateway (Razorpay/Stripe)
- Records transaction as `PENDING`
- Returns a client-side redirect link or confirmation

---

### 2. Get Payment Status

- `GET /api/payments/{paymentId}`

Returns:
```json
{
  "status": "SUCCESS",
  "paymentId": "pay_456",
  "gateway": "Razorpay",
  "timestamp": "2025-04-13T10:00:00Z"
}
```

---

## 🗃️ Database Schema (PostgreSQL)

### `payments`
| Field         | Type        | Description                  |
|---------------|-------------|------------------------------|
| id            | UUID (PK)   | Payment ID                   |
| order_id      | UUID        | Linked order                 |
| user_id       | UUID        | User ID                      |
| amount        | DECIMAL     | Transaction amount           |
| method        | TEXT        | card, upi, wallet, etc.      |
| status        | TEXT        | PENDING, SUCCESS, FAILED     |
| gateway       | TEXT        | Stripe, Razorpay, etc.       |
| created_at    | TIMESTAMP   | Initiation time              |

---

## 📤 Kafka Events

### Produced:
- `PaymentSuccess`
- `PaymentFailed`

### Consumed:
- `OrderCreated`

---

## ⚙️ Gateway Integration

### StripeService / RazorpayService
- Adapter classes
- Abstract retry logic (3 attempts)
- Parse responses into common format

### Signature Verification
- Verifies webhook authenticity
- Prevents spoofing or fake status updates

---

## 🔐 Security

- Sensitive card/token data stored via secure vaults or tokenization
- HTTPS enforced for all endpoints
- HMAC signature check for webhooks
- Access control via JWT token validation

---

## 🧪 Testing Strategy

- Mock gateway API for unit and integration tests
- Test edge cases: duplicate payments, failed network calls
- E2E tests with fake payment gateway sandbox

---

## 🛠 Design Considerations

- Payment service is stateless except DB
- External failures isolated via retries and circuit breakers
- Payment reference ID ensures idempotency
- Failures don't crash system; user notified and retry allowed

---

## 🧠 Future Enhancements

- Add support for EMI and BNPL (Buy Now Pay Later)
- Payment retry and auto-cancel on timeout
- Refund API and reconciliation module
- Multi-currency and region support

---

# ✅ End of LLD