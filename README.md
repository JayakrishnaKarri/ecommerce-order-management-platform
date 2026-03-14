# 🛒 E-Commerce Order Management Platform

![Java](https://img.shields.io/badge/Java-21-orange)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5-brightgreen)
![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-black)
![Spring AI](https://img.shields.io/badge/Spring%20AI-1.0.2-blue)
![MySQL](https://img.shields.io/badge/MySQL-8.3.0-blue)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)

A production-grade, event-driven microservices backend for asynchronous order processing — built with **Spring Boot**, **Apache Kafka**, **Spring Security (JWT/RBAC)**, and **Spring AI** for intelligent DLQ failure analysis.

---

## Architecture

```
                    ┌──────────────────────────────────────┐
                    │         API GATEWAY  :8080           │
                    │   JWT validation · Route filtering   │
                    └────┬────┬────┬────┬────┬─────────────┘
                         │    │    │    │    │
          ┌──────────────┘    │    │    │    └──────────────┐
          │                   │    │    │                    │
   ┌──────▼──────┐  ┌─────────▼┐  │  ┌─▼────────┐  ┌──────▼───────┐
   │    User     │  │  Order   │  │  │Inventory │  │   Payment    │
   │  Service   │  │ Service  │  │  │ Service  │  │   Service    │
   │   :8081    │  │  :8082   │  │  │  :8083   │  │    :8084     │
   └─────────────┘  └────┬─────┘  │  └────┬─────┘  └──────┬───────┘
                         │        │       │                  │
              ┌──────────▼────────▼───────▼──────────────────▼──────┐
              │                  APACHE KAFKA                        │
              │  order.placed | inventory.response | payment.response│
              │  order.status.update | *.DLT (Dead Letter Topics)    │
              └──────────┬──────────────────────────┬────────────────┘
                         │                          │
            ┌────────────▼──────────┐  ┌────────────▼────────────────┐
            │  Notification Service │  │  DLQ Intelligence Service   │
            │        :8085          │  │          :8086               │
            │   Order event alerts  │  │  Spring AI + GPT-4o-mini    │
            └───────────────────────┘  │  Automated root-cause RCA   │
                                       └─────────────────────────────┘
```

## Services

| Service | Port | Responsibility |
|---|---|---|
| **api-gateway** | 8080 | JWT validation, route filtering, single entry point |
| **user-service** | 8081 | Registration, login, JWT issuance, RBAC |
| **order-service** | 8082 | Order CRUD, Kafka saga orchestrator |
| **inventory-service** | 8083 | Stock management, inventory reservation |
| **payment-service** | 8084 | Payment processing (pluggable gateway) |
| **notification-service** | 8085 | Event-driven order status notifications |
| **dlq-intelligence-service** | 8086 | AI-powered Dead Letter Queue analysis |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Spring Boot 3.2, Spring Cloud Gateway |
| Messaging | Apache Kafka 7.5 (Confluent) |
| Security | Spring Security 6 + JWT (JJWT 0.11) |
| AI | Spring AI 0.8 + OpenAI GPT-4o-mini |
| Database | MySQL 8.0 (isolated schema per service) |
| Container | Docker + Docker Compose |
| Build | Maven 3.9, Java 17 |

---

## Quick Start

### Option A — Docker (no Java/Maven needed)
```bash
git clone https://github.com/YOUR_USERNAME/orderflow.git
cd orderflow

cp .env.example .env
# Edit .env and set OPENAI_API_KEY=sk-...

./start.sh
```

### Option B — Local IDE (IntelliJ / VS Code)
```bash
# 1. Start only infrastructure
docker-compose up -d mysql zookeeper kafka kafka-ui

# 2. Open project in IntelliJ → Load Maven Projects
# 3. Run each Application.java in this order:
#    UserServiceApplication      → :8081
#    InventoryServiceApplication → :8083
#    PaymentServiceApplication   → :8084
#    NotificationServiceApplication → :8085
#    OrderServiceApplication     → :8082
#    DlqIntelligenceServiceApplication → :8086
#    ApiGatewayApplication       → :8080
```

---

## API Usage

### Register & Login
```bash
# Register
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"john","email":"john@example.com","password":"Password123","role":"CUSTOMER"}'

# Login → copy the accessToken
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"john","password":"Password123"}'
```

### Add Inventory (ADMIN role)
```bash
curl -X POST http://localhost:8080/api/v1/inventory/products \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"sku":"PROD-001","name":"MacBook Pro","price":2499.99,"stockQuantity":50}'
```

### Place an Order
```bash
curl -X POST http://localhost:8080/api/v1/orders \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [{"productId":1,"productName":"MacBook Pro","quantity":1,"unitPrice":2499.99}],
    "shippingAddress": "123 Main St, Chennai"
  }'
```

### View DLQ AI Analysis (ADMIN)
```bash
curl http://localhost:8080/api/v1/dlq \
  -H "Authorization: Bearer $ADMIN_TOKEN"
```

---

## Kafka Event Flow

```
Customer places order
  └─► [order.placed]
        ├─► inventory-service  →  [inventory.response]  →  order-service
        ├─► payment-service    →  [payment.response]    →  order-service
        └─► notification-service (order confirmation email)

order-service  →  [order.status.update]  →  notification-service

On consumer failure (after 3 retries):
  any-service  →  [topic.DLT]  →  dlq-intelligence-service (AI root-cause analysis)
```

---

## Roles & Permissions

| Role | Access |
|---|---|
| `CUSTOMER` | Place orders, view own orders and notifications |
| `ADMIN` | All endpoints, DLQ analysis dashboard |
| `INVENTORY_MANAGER` | Manage products and stock levels |
| `PAYMENT_PROCESSOR` | View payment records |

---

## Project Structure

```
orderflow/
├── docker-compose.yml          # Full stack orchestration
├── init-db.sql                 # Creates all 6 databases
├── start.sh / stop.sh          # Convenience scripts
├── .env.example                # Environment variable template
├── api-gateway/                # Spring Cloud Gateway + JWT filter
├── user-service/               # Auth, JWT issuance
├── order-service/              # Saga orchestrator
├── inventory-service/          # Stock reservation
├── payment-service/            # Payment processing
├── notification-service/       # Event-driven alerts
└── dlq-intelligence-service/   # Spring AI failure analysis
```

---

## Environment Variables

| Variable | Description |
|---|---|
| `OPENAI_API_KEY` | OpenAI API key for DLQ AI analysis |
| `MYSQL_ROOT_PASSWORD` | MySQL root password |
| `MYSQL_USER` | MySQL application user |
| `MYSQL_PASSWORD` | MySQL application password |
| `JWT_SECRET` | Base64-encoded HS256 signing key |

Generate a strong JWT secret:
```bash
openssl rand -base64 32 | tr -d '\n' | base64
```

---

## Monitoring

| Tool | URL | Purpose |
|---|---|---|
| Kafka UI | http://localhost:8090 | Inspect topics, messages, consumer groups |
| Actuator (per service) | http://localhost:808x/actuator/health | Service health |

---

## License

MIT
