# 🛒 E-Commerce Order Management Platform

<p align="center">

![Java](https://img.shields.io/badge/Java-21-orange?style=for-the-badge\&logo=openjdk)
![Spring Boot](https://img.shields.io/badge/SpringBoot-3.x-6DB33F?style=for-the-badge\&logo=springboot)
![Apache Kafka](https://img.shields.io/badge/ApacheKafka-000000?style=for-the-badge\&logo=apachekafka)
![Spring Security](https://img.shields.io/badge/SpringSecurity-JWT-green?style=for-the-badge)
![Spring AI](https://img.shields.io/badge/SpringAI-LLM-blue?style=for-the-badge)
![MySQL](https://img.shields.io/badge/MySQL-8.x-00758F?style=for-the-badge\&logo=mysql)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge\&logo=docker)

</p>

<p align="center">
Production-grade <b>event-driven microservices backend</b> for asynchronous order processing  
built using <b>Spring Boot, Apache Kafka, Spring Security (JWT/RBAC)</b>, and <b>Spring AI</b> for intelligent DLQ failure analysis.
</p>

---

# 📚 Table of Contents

* Architecture
* Services
* Technology Stack
* Quick Start
* API Usage
* Kafka Event Flow
* Roles & Permissions
* Project Structure
* Environment Variables
* Monitoring
* License

---

# 🏗 Architecture

```
                    ┌──────────────────────────────────────┐
                    │         API GATEWAY  :8080           │
                    │   JWT validation · Route filtering   │
                    └────┬────┬────┬────┬────┬─────────────┘
                         │    │    │    │    │
          ┌──────────────┘    │    │    │    └──────────────┐
          │                   │    │    │                    │
   ┌──────▼──────┐  ┌─────────▼┐  │  ┌─▼────────┐  ┌──────▼───────┐
   │  User       │  │  Order   │  │  │Inventory │  │   Payment    │
   │  Service    │  │ Service  │  │  │ Service  │  │   Service    │
   │   :8081     │  │  :8082   │  │  │  :8083   │  │    :8084     │
   └─────────────┘  └────┬─────┘  │  └────┬─────┘  └──────┬───────┘
                         │        │       │                  │
              ┌──────────▼────────▼───────▼──────────────────▼──────┐
              │                  APACHE KAFKA                        │
              │  order.placed | inventory.response | payment.response│
              │  order.status.update | *.DLT (Dead Letter Topics)    │
              └──────────┬──────────────────────────┬────────────────┘
                         │                          │
            ┌────────────▼──────────┐  ┌────────────▼────────────────┐
            │ Notification Service  │  │ DLQ Intelligence Service    │
            │        :8085          │  │        :8086                 │
            │ Event driven alerts   │  │ Spring AI + GPT analysis    │
            └───────────────────────┘  │ Automated root cause RCA    │
                                       └─────────────────────────────┘
```

---

# 🧩 Services

| Service                      | Port | Responsibility                              |
| ---------------------------- | ---- | ------------------------------------------- |
| **api-gateway**              | 8080 | Single entry point, JWT validation, routing |
| **user-service**             | 8081 | User registration, login, JWT generation    |
| **order-service**            | 8082 | Order orchestration and saga management     |
| **inventory-service**        | 8083 | Product inventory management                |
| **payment-service**          | 8084 | Payment processing                          |
| **notification-service**     | 8085 | Event-driven order notifications            |
| **dlq-intelligence-service** | 8086 | AI-based failure analysis                   |

---

# ⚙️ Technology Stack

| Layer                | Technology                            |
| -------------------- | ------------------------------------- |
| **Framework**        | Spring Boot 3.x, Spring Cloud Gateway |
| **Messaging**        | Apache Kafka                          |
| **Security**         | Spring Security + JWT                 |
| **AI Integration**   | Spring AI + OpenAI                    |
| **Database**         | MySQL (schema per service)            |
| **Containerization** | Docker & Docker Compose               |
| **Build Tool**       | Maven                                 |
| **Language**         | Java 17                               |

---

# 🚀 Quick Start

## Option 1 — Docker (Recommended)

Run the full platform with Docker.

```bash
git clone https://github.com/YOUR_USERNAME/orderflow.git
cd orderflow

cp .env.example .env

# set OPENAI_API_KEY in .env
./start.sh
```

---

## Option 2 — Run Services Locally

Start infrastructure first.

```bash
docker-compose up -d mysql zookeeper kafka kafka-ui
```

Run services in this order:

```
UserServiceApplication
InventoryServiceApplication
PaymentServiceApplication
NotificationServiceApplication
OrderServiceApplication
DlqIntelligenceServiceApplication
ApiGatewayApplication
```

---

# 📡 API Usage

## Register User

```bash
curl -X POST http://localhost:8080/api/v1/auth/register \
-H "Content-Type: application/json" \
-d '{"username":"john","email":"john@example.com","password":"Password123","role":"CUSTOMER"}'
```

---

## Login

```bash
curl -X POST http://localhost:8080/api/v1/auth/login \
-H "Content-Type: application/json" \
-d '{"username":"john","password":"Password123"}'
```

Copy the returned `accessToken`.

---

## Add Inventory

```bash
curl -X POST http://localhost:8080/api/v1/inventory/products \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{"sku":"PROD-001","name":"MacBook Pro","price":2499.99,"stockQuantity":50}'
```

---

## Place Order

```bash
curl -X POST http://localhost:8080/api/v1/orders \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json"
```

---

# 🔄 Kafka Event Flow

```
Customer places order
      │
      ▼
[order.placed]
      │
      ├── inventory-service → inventory.response
      ├── payment-service   → payment.response
      └── notification-service

order-service → order.status.update

On failure:
service → topic.DLT → dlq-intelligence-service
```

---

# 🔐 Roles & Permissions

| Role                  | Access                    |
| --------------------- | ------------------------- |
| **CUSTOMER**          | Place orders, view orders |
| **ADMIN**             | Access all endpoints      |
| **INVENTORY_MANAGER** | Manage products           |
| **PAYMENT_PROCESSOR** | Payment operations        |

---

# 📂 Project Structure

```
orderflow
│
├── docker-compose.yml
├── init-db.sql
├── start.sh / stop.sh
├── .env.example
│
├── api-gateway
├── user-service
├── order-service
├── inventory-service
├── payment-service
├── notification-service
└── dlq-intelligence-service
```

---

# ⚙️ Environment Variables

| Variable            | Description            |
| ------------------- | ---------------------- |
| OPENAI_API_KEY      | OpenAI API key         |
| MYSQL_ROOT_PASSWORD | MySQL root password    |
| MYSQL_USER          | MySQL application user |
| MYSQL_PASSWORD      | MySQL password         |
| JWT_SECRET          | Base64 JWT signing key |

Generate JWT secret:

```bash
openssl rand -base64 32
```

---

# 📊 Monitoring

| Tool            | URL                                                                            |
| --------------- | ------------------------------------------------------------------------------ |
| Kafka UI        | http://localhost:8090                                                          |
| Spring Actuator | [http://localhost:808x/actuator/health](http://localhost:808x/actuator/health) |

---

# 📜 License

MIT
