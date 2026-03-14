<div align="center">

<img src="https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white"/>
<img src="https://img.shields.io/badge/Spring_Boot-3.5.11-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white"/>
<img src="https://img.shields.io/badge/Apache_Kafka-7.5-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white"/>
<img src="https://img.shields.io/badge/Spring_AI-1.0.3-6DB33F?style=for-the-badge&logo=spring&logoColor=white"/>
<img src="https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white"/>
<img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white"/>

<br/>
<br/>

# OrderNexus

### E-Commerce Order Management Platform

*A production-grade event-driven microservices backend built with Spring Boot · Apache Kafka · Spring Security JWT · Spring AI*

<br/>

[Getting Started](#getting-started) •
[Architecture](#architecture) •
[Services](#microservices) •
[API Reference](#api-reference) •
[Kafka Events](#kafka-event-flow) •
[AI Module](#ai-observability-module) •
[Security](#security-model)

</div>

---

## Overview

OrderNexus is a **fully event-driven microservices platform** that demonstrates modern backend engineering practices across the complete lifecycle of an e-commerce order — from placement through inventory reservation, payment processing, customer notification, and AI-assisted failure analysis.

The system is built around an **asynchronous Kafka saga** where services communicate exclusively through domain events, with no direct HTTP calls between services. This ensures each service is fully decoupled, independently deployable, and resilient to partial failures.

| Design Goal | Implementation |
|---|---|
| Loose coupling | Services communicate via Kafka events only — zero inter-service HTTP |
| Fault tolerance | `@RetryableTopic` with exponential backoff + Dead Letter Topics on every consumer |
| Observability | Spring AI analyzes every failed Kafka message automatically |
| Security | JWT authentication + role-based authorization on every endpoint |
| Scalability | Java 21 Virtual Threads + stateless services |

---

## Architecture

![OrderNexus System Architecture](docs/architecture.svg)

The platform follows a layered architecture where all external traffic enters through the API Gateway, business logic lives in independent Spring Boot services, and all inter-service communication flows exclusively through Apache Kafka.

<details>
<summary>View Kafka event flow diagram</summary>
<br/>

![Kafka Event Flow](docs/kafka-event-flow.svg)

</details>

### Key architectural principles

| Principle | Implementation |
|---|---|
| Domain-driven microservices | Each service owns its bounded context and database schema |
| Event-driven communication | All inter-service communication via Kafka — zero HTTP calls between services |
| Database per service | 6 isolated MySQL schemas — no shared tables, no cross-schema JOINs |
| Saga pattern (choreography) | Order flow coordinated by Kafka events, not a central orchestrator |
| Fault tolerance | `@RetryableTopic` with exponential backoff + Dead Letter Topics on every consumer |
| Stateless APIs | JWT in every request — no server-side sessions ever created |
| Java 21 Virtual Threads | `spring.threads.virtual.enabled=true` across all services |

---

## Microservices

| Service | Port | Responsibility |
|---|---|---|
| **api-gateway** | 8080 | JWT validation, route filtering, single entry point for all clients |
| **user-service** | 8081 | User registration, login, JWT issuance, BCrypt password hashing |
| **order-service** | 8082 | Order lifecycle management, Kafka saga orchestrator |
| **inventory-service** | 8083 | Product catalogue, transactional stock reservation |
| **payment-service** | 8084 | Payment processing with simulated gateway (pluggable) |
| **notification-service** | 8085 | Event-driven customer alerts (email/SMS ready) |
| **dlq-intelligence-service** | 8086 | AI root-cause analysis + Text-to-SQL natural language queries |

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Language | Java | 21 LTS |
| Framework | Spring Boot | 3.5.11 |
| Messaging | Apache Kafka (Confluent) | 7.5 |
| Security | Spring Security + jjwt | 6.x / 0.12.6 |
| AI | Spring AI + OpenAI GPT-4o-mini | 1.0.3 |
| Gateway | Spring Cloud Gateway (WebFlux) | 2025.0.0 |
| Database | MySQL | 8.0 |
| ORM | Spring Data JPA + Hibernate | 6.x |
| API Docs | SpringDoc OpenAPI (Swagger UI) | 2.8.9 |
| Containers | Docker + Docker Compose | — |
| Build | Maven | 3.9.x |

---

## Getting Started

### Prerequisites

| Tool | Version | Check |
|---|---|---|
| Java | 21+ | `java -version` |
| Maven | 3.8+ | `mvn -version` |
| Docker Desktop | Latest | `docker -v` |

### Option A — Docker (no Java/Maven needed locally)

```bash
# 1. Clone the repository
git clone https://github.com/YOUR_USERNAME/ordernexus.git
cd ordernexus

# 2. Set your environment variables
cp .env.example .env
# Edit .env and set: OPENAI_API_KEY=sk-your-key-here

# 3. Start everything with one command
./start.sh
```

### Option B — Local IDE (IntelliJ / VS Code)

```bash
# 1. Start only infrastructure via Docker
docker-compose up -d mysql zookeeper kafka kafka-ui

# 2. Open project in IntelliJ → click "Load Maven Projects" when prompted

# 3. Run each Application.java in this exact order:
#    UserServiceApplication         → :8081
#    InventoryServiceApplication    → :8083
#    PaymentServiceApplication      → :8084
#    NotificationServiceApplication → :8085
#    OrderServiceApplication        → :8082
#    DlqIntelligenceServiceApplication → :8086
#    ApiGatewayApplication          → :8080
```

### Verify everything is running

```bash
# Health check all 7 services
for port in 8080 8081 8082 8083 8084 8085 8086; do
  echo -n "Port $port: "
  curl -s http://localhost:$port/actuator/health     | python3 -c "import sys,json; print(json.load(sys.stdin).get('status','?'))"
done
```

### Useful URLs

| Resource | URL |
|---|---|
| API Gateway | http://localhost:8080 |
| Kafka UI (topics & messages) | http://localhost:8090 |
| User Service Swagger | http://localhost:8081/swagger-ui.html |
| Order Service Swagger | http://localhost:8082/swagger-ui.html |
| Inventory Service Swagger | http://localhost:8083/swagger-ui.html |
| Payment Service Swagger | http://localhost:8084/swagger-ui.html |
| Notification Service Swagger | http://localhost:8085/swagger-ui.html |
| DLQ Intelligence Swagger | http://localhost:8086/swagger-ui.html |

---

## API Reference

### Authentication

```bash
# Register — choose role: CUSTOMER | ADMIN | INVENTORY_MANAGER | PAYMENT_PROCESSOR
curl -X POST http://localhost:8081/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"john","email":"john@example.com","password":"Password@123","role":"CUSTOMER"}'

# Login — returns accessToken (24h) and refreshToken (7d)
curl -X POST http://localhost:8081/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"john","password":"Password@123"}'
```

Save your token as a shell variable for convenience:

```bash
TOKEN=$(curl -s -X POST http://localhost:8081/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"john","password":"Password@123"}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['accessToken'])")
```

### Orders

```bash
POST   /api/v1/orders                     # CUSTOMER or ADMIN — triggers Kafka saga
GET    /api/v1/orders/{orderNumber}       # any authenticated user
GET    /api/v1/orders/customer/{id}       # CUSTOMER or ADMIN
GET    /api/v1/orders                     # ADMIN only
PATCH  /api/v1/orders/{orderNumber}/cancel
```

**Example — place an order:**

```bash
curl -X POST http://localhost:8082/api/v1/orders \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [{"productId":1,"productName":"MacBook Pro","quantity":1,"unitPrice":2499.99}],
    "shippingAddress": "123 Main St, Chennai"
  }'
```

### Inventory

```bash
POST   /api/v1/inventory/products              # ADMIN or INVENTORY_MANAGER
GET    /api/v1/inventory/products              # any authenticated user
GET    /api/v1/inventory/products/{id}         # any authenticated user
PATCH  /api/v1/inventory/products/{id}/stock   # ADMIN or INVENTORY_MANAGER
```

### Payments

```bash
GET    /api/v1/payments                        # ADMIN only
GET    /api/v1/payments/order/{orderNumber}    # any authenticated user
```

### Notifications

```bash
GET    /api/v1/notifications                        # ADMIN only
GET    /api/v1/notifications/order/{orderNumber}    # any authenticated user
GET    /api/v1/notifications/customer/{id}          # any authenticated user
```

### AI Observability

```bash
# Natural language → SQL → live results from MySQL
POST   /api/v1/ai/query
{"query": "Show failed payments today"}
{"query": "Which products have less than 10 items in stock?"}
{"query": "Top 5 customers by total order value"}
{"query": "Show DLQ failures that have not been analyzed yet"}

# Ad-hoc error analysis (no DLQ required)
POST   /api/v1/ai/analyze-error
{"exceptionMessage": "...", "stackTrace": "...", "payload": "..."}

# DLQ failure records with AI root-cause
GET    /api/v1/dlq
GET    /api/v1/dlq/recent
GET    /api/v1/dlq/{id}
POST   /api/v1/dlq/{id}/reanalyze
POST   /api/v1/dlq/analyze-pending
```

---

## Kafka Event Flow

The order processing saga is choreographed entirely through Kafka events. No service calls another service directly over HTTP.

```
Customer places order
        │
        ▼
  order-service  ──────────────►  [order.placed]
                                         │
                        ┌────────────────┼────────────────┐
                        ▼                ▼                ▼
               inventory-service   notification-service  payment-service
               (reserve stock)     (order confirmation)  (charge customer)
                        │                                  │
                        ▼                                  ▼
               [inventory.response]              [payment.response]
                        │                                  │
                        └──────────────┬───────────────────┘
                                       ▼
                                 order-service
                              (update order status)
                                       │
                                       ▼
                             [order.status.update]
                                       │
                                       ▼
                             notification-service
                             (status change alert)

  On consumer failure (after 3 retries with exponential backoff):
        any-service  ──►  [topic.DLT]  ──►  dlq-intelligence-service
                                            (Spring AI root-cause + fix)
```

### Topic ownership

| Topic | Owned by | Consumed by |
|---|---|---|
| `order.placed` | order-service | inventory-service, payment-service, notification-service |
| `order.status.update` | order-service | notification-service |
| `inventory.response` | inventory-service | order-service |
| `payment.response` | payment-service | order-service |
| `*.DLT` | auto-created by `@RetryableTopic` | dlq-intelligence-service |

---

## AI Observability Module

The `dlq-intelligence-service` provides two AI-powered capabilities built with **Spring AI 1.0 + OpenAI GPT-4o-mini**.

### 1. Automated DLQ failure analysis

When any Kafka consumer fails after all retries are exhausted, the message is automatically routed to a Dead Letter Topic (`*.DLT`). The DLQ service captures every failure and calls the OpenAI API to generate a structured analysis:

```json
{
  "topic": "order.placed.DLT",
  "exceptionClass": "ProductNotFoundException",
  "aiRootCauseSummary": "The inventory reservation failed because the product ID
    referenced in the order does not exist in the inventory database. This is
    likely a race condition where the order was placed before the product
    catalogue was synced.",
  "aiSuggestedFix": "1. Add product existence validation in order-service before
    publishing ORDER_PLACED\n2. Implement compensating transaction to cancel
    the order if inventory returns NOT_FOUND\n3. Add circuit breaker pattern...",
  "analysisStatus": "COMPLETED"
}
```

### 2. Text-to-SQL natural language queries

Ask questions in plain English — the AI generates the SQL, executes it against live MySQL data, and returns the results with full transparency.

```bash
POST /api/v1/ai/query
{"query": "Show failed payments today"}

# Response
{
  "naturalLanguageQuery": "Show failed payments today",
  "generatedSql": "SELECT * FROM paymentdb.payments WHERE status = 'FAILED'
                   AND DATE(created_at) = CURDATE() ORDER BY created_at DESC LIMIT 100;",
  "explanation": "Retrieves all failed payment records from today, ordered by most recent.",
  "results": [...],
  "rowCount": 3,
  "executionTimeMs": 42,
  "success": true
}
```

---

## Security Model

All APIs are secured with **stateless JWT authentication** and **role-based authorization**.

### Authentication flow

```
Client  ──POST /auth/login──►  user-service  ──►  JWT (HS256, 24h expiry)
  │
  └──►  All subsequent requests:
        Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
                                │
                         API Gateway validates JWT
                         Injects X-Auth-Username + X-Auth-Roles headers
                                │
                         Downstream service reads roles from JWT claims
                         @PreAuthorize enforces method-level access control
```

### Roles and permissions

| Role | Permissions |
|---|---|
| `CUSTOMER` | Place orders · View own orders · View own notifications |
| `ADMIN` | All endpoints · DLQ analysis · User management · AI queries |
| `INVENTORY_MANAGER` | Add/edit products · Update stock levels |
| `PAYMENT_PROCESSOR` | View payment records |

### Security features implemented

- Passwords hashed with **BCrypt** (strength 10) — never stored in plain text
- JWT signed with **HS256** — secret loaded from environment variable `JWT_SECRET`
- Access tokens expire after **24 hours**, refresh tokens after **7 days**
- CSRF disabled — correct for stateless token-based APIs
- `SessionCreationPolicy.STATELESS` — no `HttpSession` ever created
- `@EnableMethodSecurity` — `@PreAuthorize` enforced on every controller method
- Username enumeration prevented — `UsernameNotFoundException` mapped to `BadCredentialsException`
- Auto-generated Spring Security password disabled via `spring.security.user.name=disabled`

---

## Project Structure

```
ordernexus/
├── docker-compose.yml                  # Full stack with health checks + resource limits
├── init-db.sql                         # Creates all 6 MySQL databases + grants
├── start.sh / stop.sh                  # One-command startup scripts
├── reset-kafka-topics.sh               # Fixes partition mismatch on fresh runs
├── .env.example                        # Environment variable template
├── .gitignore
├── docs/
│   ├── architecture.svg                # System architecture diagram
│   └── kafka-event-flow.svg            # Kafka saga event flow diagram
│
├── api-gateway/                        # Spring Cloud Gateway + JWT filter
├── user-service/                       # Auth, JWT issuance, RBAC
│   └── src/main/java/com/orderflow/user/
│       ├── config/     SecurityConfig · OpenApiConfig
│       ├── controller/ AuthController
│       ├── entity/     User · Role
│       ├── exception/  GlobalExceptionHandler
│       ├── security/   JwtUtil · JwtAuthenticationFilter
│       └── service/    AuthService · UserDetailsServiceImpl
│
├── order-service/                      # Saga orchestrator
│   └── src/main/java/com/orderflow/order/
│       ├── config/     SecurityConfig · KafkaConfig · KafkaProducerConfig · KafkaConsumerConfig
│       ├── controller/ OrderController
│       ├── entity/     Order · OrderItem · OrderStatus
│       ├── event/      OrderPlacedEvent · OrderStatusUpdateEvent · InventoryResponseEvent · PaymentResponseEvent
│       ├── kafka/      OrderEventProducer · OrderEventConsumer
│       └── service/    OrderService
│
├── inventory-service/                  # Stock reservation
├── payment-service/                    # Payment processing
├── notification-service/               # Event-driven alerts
│
└── dlq-intelligence-service/           # Spring AI failure analysis
    └── src/main/java/com/orderflow/dlq/
        ├── config/     SecurityConfig · AiConfig · AsyncConfig
        ├── controller/ DlqController · AiQueryController
        ├── entity/     DlqFailureRecord · AnalysisStatus
        ├── kafka/      DlqEventConsumer
        └── service/    AiAnalysisService · TextToSqlService
```

---

## Environment Variables

| Variable | Description | Required |
|---|---|---|
| `OPENAI_API_KEY` | OpenAI API key for DLQ AI analysis | Yes (for DLQ service) |
| `JWT_SECRET` | Base64-encoded HS256 signing key | Yes |
| `MYSQL_ROOT_PASSWORD` | MySQL root password | Yes |
| `MYSQL_USER` | MySQL application username | Yes |
| `MYSQL_PASSWORD` | MySQL application password | Yes |

Copy `.env.example` to `.env` and fill in all values. The `.env` file is excluded from Git via `.gitignore`.

Generate a strong JWT secret:

```bash
openssl rand -base64 32 | tr -d '\n' | base64
```

---

## Running Tests

```bash
# All tests
mvn test

# Specific service
cd order-service && mvn test

# With coverage report
mvn test jacoco:report
```

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

Built with Java 21 · Spring Boot 3.5 · Apache Kafka · Spring AI 1.0

</div>
