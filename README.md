# E-Commerce Order Management Platform

Event-driven **microservices backend** for managing the lifecycle of e-commerce orders, built with **Java, Spring Boot, Apache Kafka, Spring Security, and Spring AI**.

The platform demonstrates modern backend engineering practices including **distributed systems, asynchronous messaging, fault-tolerant processing, JWT security, and AI-assisted observability**.

---

# Architecture Overview

The system follows an **event-driven microservices architecture** where services communicate asynchronously through Kafka events.

Order workflows such as **order creation, inventory reservation, payment processing, and notifications** are coordinated through an event pipeline to ensure **scalability, resilience, and eventual consistency**.

Key architectural characteristics:

• Domain-driven microservices
• Event-driven communication using Kafka
• Independent databases per service
• Secure APIs with JWT authentication
• Fault tolerance using retries and Dead Letter Queues
• AI-powered operational insights

---

# Microservices

### Auth Service

Handles authentication and authorization.

Responsibilities

* User registration and login
* JWT token generation
* Role-based access control (RBAC)

Main APIs

POST /auth/register
POST /auth/login

---

### Order Service

Manages the lifecycle of orders.

Responsibilities

* Create and track orders
* Publish order events
* Maintain order state

Main APIs

POST /orders
GET /orders/{id}
GET /orders/user/{userId}

Published events

ORDER_CREATED
ORDER_COMPLETED
ORDER_FAILED

---

### Inventory Service

Manages product inventory and stock reservation.

Responsibilities

* Validate product availability
* Reserve inventory
* Publish inventory events

Consumed events

ORDER_CREATED

Published events

INVENTORY_RESERVED
INVENTORY_FAILED

---

### Payment Service

Processes payments for orders.

Responsibilities

* Handle payment processing
* Persist payment records
* Publish payment events

Consumed events

INVENTORY_RESERVED

Published events

PAYMENT_COMPLETED
PAYMENT_FAILED

---

### Notification Service

Sends notifications to customers or internal systems.

Responsibilities

* Order completion notifications
* Payment failure alerts
* Email / message integrations

Consumed events

ORDER_COMPLETED
PAYMENT_FAILED

---

### AI Observability Service

Provides **AI-assisted operational insights**.

Responsibilities

* Analyze failed events from Dead Letter Queues
* Generate root-cause explanations
* Convert natural language queries into analytics queries

Example features

AI explanation of payment failures
AI-generated SQL queries for analytics
Operational summaries for support teams

Example API

POST /ai/analyze-error
POST /ai/query

---

# Event Flow

Example order processing flow:

1. Order Service publishes **ORDER_CREATED**
2. Inventory Service consumes event and validates stock
3. Inventory Service publishes **INVENTORY_RESERVED**
4. Payment Service processes payment
5. Payment Service publishes **PAYMENT_COMPLETED**
6. Order Service updates order status
7. Notification Service sends confirmation

Failure scenarios are routed to **Dead Letter Queues (DLQ)** for later analysis.

---

# Technology Stack

**Language**
Java 21

**Frameworks**
Spring Boot
Spring Security
Spring AI

**Messaging**
Apache Kafka

**Database**
MySQL

**Containerization**
Docker
Docker Compose

**Observability**
Micrometer
Prometheus
Grafana

**Security**
JWT Authentication
Role-Based Access Control

---

# Repository Structure

**ecommerce-order-management-platform**

services
 auth-service
 order-service
 inventory-service
 payment-service
 notification-service
 ai-observability-service

shared-libraries
 common-dto
 event-models
 security-utils

infrastructure
 docker
 database
 kafka

docs
 architecture.md
 event-flow.md

scripts
 start-local.sh
 setup-topics.sh

---

# Running the Platform Locally

Prerequisites

Java 21
Docker
Docker Compose
Maven

Start infrastructure

docker-compose up -d

Build services

mvn clean install

Run services

mvn spring-boot:run

Once running, services will be available locally through their configured ports.

---

# Kafka Topics

order-events
inventory-events
payment-events
notification-events
dlq-events

These topics enable asynchronous communication between services.

---

# Security

The platform uses **JWT-based authentication** with role-based authorization.

Roles

ROLE_USER
ROLE_ADMIN
ROLE_SUPPORT

Protected endpoints require valid JWT tokens.

---

# AI Capabilities

The AI service integrates with **Spring AI** to provide intelligent operational tooling.

Features

AI analysis of failed events
Root-cause explanations for incidents
Natural language analytics queries
Operational summaries for support teams

Example

User query

"Show failed payments today"

AI converts it to an SQL query executed on analytics data.

---

# Future Enhancements

Distributed tracing (OpenTelemetry)
Kubernetes deployment
Automated CI/CD pipelines

---

# Learning Goals

This project demonstrates practical experience with:

Distributed microservices architecture
Event-driven systems using Kafka
Secure REST APIs with JWT
Fault-tolerant messaging with DLQ
AI integration into backend platforms

---

# License

MIT License
