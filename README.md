# 🛒 E-Commerce Order Management Platform

![Java](https://img.shields.io/badge/Java-21-orange)
![Spring Boot](https://img.shields.io/badge/SpringBoot-4.x-brightgreen)
![Kafka](https://img.shields.io/badge/Apache-Kafka-black)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue)
![License](https://img.shields.io/badge/License-MIT-green)

An **event-driven microservices backend** for managing the lifecycle of e-commerce orders, built with **Java, Spring Boot, Apache Kafka, Spring Security, and Spring AI**.

The platform demonstrates modern backend engineering practices including:

* Distributed microservices architecture
* Asynchronous messaging with Kafka
* Fault-tolerant event processing
* Secure APIs using JWT authentication
* AI-assisted observability and operational insights

---

# 🏗️ Architecture Overview

The system follows an **event-driven microservices architecture** where services communicate asynchronously through Kafka events.

Order workflows such as **order creation, inventory reservation, payment processing, and notifications** are coordinated through an event pipeline to ensure:

* Scalability
* Resilience
* Eventual consistency across distributed services

### Key Architectural Principles

* Domain-driven microservices
* Event-driven communication using Kafka
* Independent database per service
* JWT-secured APIs
* Retry mechanisms and Dead Letter Queues
* AI-powered operational insights

---

# 🧩 Microservices

| Service                      | Responsibility                               |
| ---------------------------- | -------------------------------------------- |
| **Auth Service**             | Authentication, JWT generation, RBAC         |
| **Order Service**            | Order lifecycle management                   |
| **Inventory Service**        | Product inventory validation and reservation |
| **Payment Service**          | Payment processing                           |
| **Notification Service**     | Customer notifications                       |
| **AI Observability Service** | AI analysis of system failures               |

---

## 🔐 Auth Service

Handles **authentication and authorization**.

**Responsibilities**

* User registration
* User login
* JWT token generation
* Role-based access control (RBAC)

**APIs**

```http
POST /auth/register
POST /auth/login
```

---

## 📦 Order Service

Manages the **order lifecycle**.

**Responsibilities**

* Create and track orders
* Publish order events
* Maintain order state

**APIs**

```http
POST /orders
GET /orders/{id}
GET /orders/user/{userId}
```

**Published Events**

```
ORDER_CREATED
ORDER_COMPLETED
ORDER_FAILED
```

---

## 📊 Inventory Service

Handles **inventory validation and reservation**.

**Responsibilities**

* Validate product availability
* Reserve inventory
* Publish inventory events

**Consumed Events**

```
ORDER_CREATED
```

**Published Events**

```
INVENTORY_RESERVED
INVENTORY_FAILED
```

---

## 💳 Payment Service

Processes **order payments**.

**Responsibilities**

* Payment processing
* Payment record persistence
* Publish payment events

**Consumed Events**

```
INVENTORY_RESERVED
```

**Published Events**

```
PAYMENT_COMPLETED
PAYMENT_FAILED
```

---

## 🔔 Notification Service

Handles **customer notifications**.

**Responsibilities**

* Order confirmation notifications
* Payment failure alerts
* Messaging integrations

**Consumed Events**

```
ORDER_COMPLETED
PAYMENT_FAILED
```

---

## 🤖 AI Observability Service

Provides **AI-assisted operational intelligence**.

**Responsibilities**

* Analyze events from Dead Letter Queues
* Generate root-cause explanations
* Convert natural language queries into SQL/analytics queries

**Example APIs**

```http
POST /ai/analyze-error
POST /ai/query
```

Example AI Query

```
"Show failed payments today"
```

AI converts this into a SQL analytics query.

---

# 🔄 Event Flow

Example order processing pipeline:

```
User creates order
        │
        ▼
Order Service
        │
        ▼
Publish ORDER_CREATED
        │
        ▼
Inventory Service
        │
        ▼
Publish INVENTORY_RESERVED
        │
        ▼
Payment Service
        │
        ▼
Publish PAYMENT_COMPLETED
        │
        ▼
Order Service updates order
        │
        ▼
Notification Service sends confirmation
```

Failures are routed to **Dead Letter Queues (DLQ)** for analysis.

---

# 🧰 Technology Stack

| Category         | Technology                      |
| ---------------- | ------------------------------- |
| Language         | Java 21                         |
| Framework        | Spring Boot                     |
| Messaging        | Apache Kafka                    |
| Security         | Spring Security + JWT           |
| Database         | MySQL                           |
| AI Integration   | Spring AI                       |
| Containerization | Docker                          |
| Observability    | Micrometer, Prometheus, Grafana |

---

# 📁 Repository Structure

```
ecommerce-order-management-platform
│
├── services
│   ├── auth-service
│   ├── order-service
│   ├── inventory-service
│   ├── payment-service
│   ├── notification-service
│   └── ai-observability-service
│
├── shared-libraries
│   ├── common-dto
│   ├── event-models
│   └── security-utils
│
├── infrastructure
│   ├── docker
│   ├── database
│   └── kafka
│
├── docs
│   ├── architecture.md
│   └── event-flow.md
│
├── scripts
│   ├── start-local.sh
│   └── setup-topics.sh
```

---

# 🚀 Running the Platform Locally

### Prerequisites

* Java 21
* Maven
* Docker
* Docker Compose

---

### Start Infrastructure

```bash
docker-compose up -d
```

---

### Build Services

```bash
mvn clean install
```

---

### Run Services

```bash
mvn spring-boot:run
```

Once running, all services will be available locally via their configured ports.

---

# 📡 Kafka Topics

```
order-events
inventory-events
payment-events
notification-events
dlq-events
```

These topics enable asynchronous communication across microservices.

---

# 🔐 Security

The platform uses **JWT-based authentication** with **role-based authorization**.

### Roles

```
ROLE_USER
ROLE_ADMIN
ROLE_SUPPORT
```

Protected endpoints require a valid JWT token.

---

# 🤖 AI Capabilities

The **AI Observability Service** integrates with **Spring AI** to enhance operational intelligence.

### Features

* AI analysis of failed events
* Root-cause explanations for incidents
* Natural language analytics queries
* Operational summaries for support teams

Example

```
User Query:
"Show failed payments today"
```

AI generates and executes the appropriate SQL query.

---

# 🔮 Future Enhancements

* API Gateway
* Distributed tracing with OpenTelemetry
* Kubernetes deployment
* Automated CI/CD pipelines
* Advanced monitoring dashboards

---

# 🎯 Learning Goals

This project demonstrates hands-on experience with:

* Distributed microservices architecture
* Event-driven systems using Kafka
* Secure REST APIs with JWT
* Fault-tolerant messaging and DLQ
* AI integration in backend platforms

---

# 📄 License

MIT License
