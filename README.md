<div align="center">

<img src="https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white"/>
<img src="https://img.shields.io/badge/Spring_Boot-3.5.11-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white"/>
<img src="https://img.shields.io/badge/Apache_Kafka-4.1.1-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white"/>
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

<details open>
<summary><strong>System architecture</strong></summary>

  <svg xmlns="http://www.w3.org/2000/svg" width="900" viewBox="0 0 900 1100"
     style="background:#ffffff;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif">

  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#6B7280" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="parr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#B45309" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="garr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#059669" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="varr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#7C3AED" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="rarr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#DC2626" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <rect width="900" height="1100" fill="#ffffff"/>

  <text x="450" y="34" text-anchor="middle" font-size="20" font-weight="600" fill="#111827">OrderNexus — Architecture</text>
  <text x="450" y="56" text-anchor="middle" font-size="12" fill="#6B7280">Java 21 · Spring Boot 3.5 · Apache Kafka · Spring AI · Resilience4j</text>

  <!-- ── TIER 1: Config Server ─────────────────────────────────────────── -->
  <rect x="310" y="76" width="280" height="56" rx="8" fill="#FEF3C7" stroke="#B45309" stroke-width="1"/>
  <text x="450" y="99" text-anchor="middle" font-size="14" font-weight="600" fill="#78350F">Config Server  :8888</text>
  <text x="450" y="118" text-anchor="middle" font-size="11" fill="#92400E">Spring Cloud Config · centralised config for all services</text>

  <line x1="450" y1="132" x2="450" y2="158" stroke="#B45309" stroke-width="1.5" marker-end="url(#parr)"/>

  <!-- ── TIER 2: Eureka Server ─────────────────────────────────────────── -->
  <rect x="310" y="158" width="280" height="56" rx="8" fill="#FEF3C7" stroke="#B45309" stroke-width="1"/>
  <text x="450" y="181" text-anchor="middle" font-size="14" font-weight="600" fill="#78350F">Eureka Server  :8761</text>
  <text x="450" y="200" text-anchor="middle" font-size="11" fill="#92400E">Spring Cloud Netflix · service discovery + registry</text>

  <line x1="450" y1="214" x2="450" y2="240" stroke="#7C3AED" stroke-width="1.5" marker-end="url(#varr)"/>

  <!-- ── TIER 3: API Gateway ───────────────────────────────────────────── -->
  <rect x="310" y="240" width="280" height="56" rx="8" fill="#EDE9FE" stroke="#7C3AED" stroke-width="1"/>
  <text x="450" y="263" text-anchor="middle" font-size="14" font-weight="600" fill="#4C1D95">API Gateway  :8080</text>
  <text x="450" y="282" text-anchor="middle" font-size="11" fill="#6D28D9">JWT validation · Rate Limiter · route filtering</text>

  <!-- Fan out from gateway to 5 services -->
  <path d="M310 268 L100 268 L100 388" fill="none" stroke="#7C3AED" stroke-width="1" marker-end="url(#varr)"/>
  <path d="M390 296 L280 370 L280 388" fill="none" stroke="#7C3AED" stroke-width="1" marker-end="url(#varr)"/>
  <path d="M450 296 L450 388" fill="none" stroke="#7C3AED" stroke-width="1" marker-end="url(#varr)"/>
  <path d="M510 296 L620 370 L620 388" fill="none" stroke="#7C3AED" stroke-width="1" marker-end="url(#varr)"/>
  <path d="M590 268 L800 268 L800 388" fill="none" stroke="#7C3AED" stroke-width="1" marker-end="url(#varr)"/>

  <!-- ── TIER 4: 5 Business Services ──────────────────────────────────── -->
  <rect x="44"  y="388" width="112" height="52" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="100" y="411" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">User Service</text>
  <text x="100" y="428" text-anchor="middle" font-size="10" fill="#065F46">auth · :8081</text>

  <rect x="214" y="388" width="112" height="52" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="270" y="411" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">Order Service</text>
  <text x="270" y="428" text-anchor="middle" font-size="10" fill="#065F46">saga · CB · :8082</text>

  <rect x="384" y="388" width="132" height="52" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="450" y="411" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">Inventory Service</text>
  <text x="450" y="428" text-anchor="middle" font-size="10" fill="#065F46">stock · CB · :8083</text>

  <rect x="574" y="388" width="112" height="52" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="630" y="411" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">Payment</text>
  <text x="630" y="428" text-anchor="middle" font-size="10" fill="#065F46">charge · CB · :8084</text>

  <rect x="744" y="388" width="112" height="52" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="800" y="411" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">Notification</text>
  <text x="800" y="428" text-anchor="middle" font-size="10" fill="#065F46">alerts · BH · :8085</text>

  <!-- Services publish to Kafka -->
  <line x1="270" y1="440" x2="270" y2="510" stroke="#059669" stroke-width="1" marker-end="url(#garr)"/>
  <line x1="450" y1="440" x2="450" y2="510" stroke="#059669" stroke-width="1" marker-end="url(#garr)"/>
  <line x1="630" y1="440" x2="630" y2="510" stroke="#059669" stroke-width="1" marker-end="url(#garr)"/>
  <line x1="800" y1="440" x2="800" y2="510" stroke="#059669" stroke-width="1" marker-end="url(#garr)"/>

  <!-- ── TIER 5: Kafka Message Bus ─────────────────────────────────────── -->
  <rect x="44" y="510" width="812" height="160" rx="12" fill="#F0FDF4" stroke="#16A34A" stroke-width="1" stroke-dasharray="6 3"/>
  <text x="70" y="534" font-size="12" font-weight="600" fill="#15803D">Apache Kafka — message bus</text>

  <!-- Topic pills row 1 -->
  <rect x="64"  y="544" width="128" height="32" rx="16" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="128" y="565" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">order.placed</text>

  <rect x="204" y="544" width="160" height="32" rx="16" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="284" y="565" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">inventory.response</text>

  <rect x="376" y="544" width="148" height="32" rx="16" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="450" y="565" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">payment.response</text>

  <rect x="536" y="544" width="180" height="32" rx="16" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="626" y="565" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">order.status.update</text>

  <!-- Topic pills row 2 — DLT -->
  <rect x="64"  y="590" width="148" height="32" rx="16" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="138" y="611" text-anchor="middle" font-size="12" font-weight="500" fill="#7F1D1D">order.placed.DLT</text>

  <rect x="224" y="590" width="172" height="32" rx="16" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="310" y="611" text-anchor="middle" font-size="12" font-weight="500" fill="#7F1D1D">inventory.response.DLT</text>

  <rect x="408" y="590" width="160" height="32" rx="16" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="488" y="611" text-anchor="middle" font-size="12" font-weight="500" fill="#7F1D1D">payment.response.DLT</text>

  <text x="450" y="656" text-anchor="middle" font-size="11" fill="#6B7280">@RetryableTopic — 3 retries with exponential backoff — then routes to DLT</text>

  <!-- Kafka back to Order Service (responses) -->
  <path d="M284 510 L270 486 L270 440" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#garr)"/>
  <path d="M450 510 L450 486" fill="none" stroke="#059669" stroke-width="0.5" stroke-dasharray="4 3"/>

  <!-- DLT routes down to DLQ Intelligence -->
  <line x1="450" y1="670" x2="450" y2="710" stroke="#DC2626" stroke-width="1.5" marker-end="url(#rarr)"/>

  <!-- ── TIER 6: DLQ Intelligence ──────────────────────────────────────── -->
  <rect x="200" y="710" width="500" height="60" rx="8" fill="#FEE2E2" stroke="#DC2626" stroke-width="1.5"/>
  <text x="450" y="736" text-anchor="middle" font-size="14" font-weight="600" fill="#7F1D1D">DLQ Intelligence Service  :8086</text>
  <text x="450" y="756" text-anchor="middle" font-size="11" fill="#B91C1C">Spring AI · OpenAI GPT-4o-mini · root-cause analysis · Text-to-SQL</text>

  <!-- ── TIER 7: MySQL ──────────────────────────────────────────────────── -->
  <line x1="450" y1="770" x2="450" y2="810" stroke="#6B7280" stroke-width="1" marker-end="url(#arr)"/>

  <rect x="150" y="810" width="600" height="56" rx="8" fill="#F3F4F6" stroke="#D1D5DB" stroke-width="1"/>
  <text x="450" y="834" text-anchor="middle" font-size="13" font-weight="600" fill="#374151">MySQL 8.0 — isolated schemas</text>
  <text x="450" y="852" text-anchor="middle" font-size="11" fill="#6B7280">userdb · orderdb · inventorydb · paymentdb · notificationdb · dlqdb</text>

  <!-- DB writes from services -->
  <path d="M100 440 L100 880 L150 880" fill="none" stroke="#9CA3AF" stroke-width="0.5" stroke-dasharray="3 3" marker-end="url(#arr)"/>
  <path d="M800 440 L800 880 L750 880" fill="none" stroke="#9CA3AF" stroke-width="0.5" stroke-dasharray="3 3" marker-end="url(#arr)"/>

  <!-- ── Resilience4j labels ────────────────────────────────────────────── -->
  <rect x="44" y="888" width="812" height="52" rx="6" fill="#F9FAFB" stroke="#E5E7EB" stroke-width="0.5"/>
  <text x="450" y="908" text-anchor="middle" font-size="11" font-weight="600" fill="#374151">Resilience4j patterns</text>
  <text x="200" y="928" text-anchor="middle" font-size="11" fill="#6B7280">Circuit Breaker</text>
  <text x="200" y="942" text-anchor="middle" font-size="10" fill="#9CA3AF">Order→Payment, Order→Inventory</text>
  <text x="450" y="928" text-anchor="middle" font-size="11" fill="#6B7280">Rate Limiter</text>
  <text x="450" y="942" text-anchor="middle" font-size="10" fill="#9CA3AF">API Gateway per-route</text>
  <text x="700" y="928" text-anchor="middle" font-size="11" fill="#6B7280">Bulkhead</text>
  <text x="700" y="942" text-anchor="middle" font-size="10" fill="#9CA3AF">Notification Service</text>

  <!-- ── Legend ────────────────────────────────────────────────────────── -->
  <rect x="44" y="956" width="12" height="12" rx="2" fill="#FEF3C7" stroke="#B45309" stroke-width="1"/>
  <text x="62" y="966" font-size="11" fill="#374151">config + discovery</text>

  <rect x="185" y="956" width="12" height="12" rx="2" fill="#EDE9FE" stroke="#7C3AED" stroke-width="1"/>
  <text x="203" y="966" font-size="11" fill="#374151">API gateway</text>

  <rect x="305" y="956" width="12" height="12" rx="2" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="323" y="966" font-size="11" fill="#374151">business services</text>

  <rect x="455" y="956" width="20" height="12" rx="6" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="481" y="966" font-size="11" fill="#374151">Kafka topic</text>

  <rect x="565" y="956" width="20" height="12" rx="6" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="591" y="966" font-size="11" fill="#374151">DLT / AI</text>

  <line x1="670" y1="961" x2="700" y2="961" stroke="#059669" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#garr)"/>
  <text x="706" y="966" font-size="11" fill="#374151">Kafka event</text>

  <line x1="790" y1="961" x2="820" y2="961" stroke="#9CA3AF" stroke-width="0.5" stroke-dasharray="3 3" marker-end="url(#arr)"/>
  <text x="826" y="966" font-size="11" fill="#374151">DB write</text>

</svg>
<br/>

<svg xmlns="http://www.w3.org/2000/svg" width="900" viewBox="0 0 900 860" style="background:#ffffff;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif">

  <!-- Background -->
  <rect width="900" height="860" fill="#ffffff"/>

  <!-- Title -->
  <text x="450" y="36" text-anchor="middle" font-size="18" font-weight="600" fill="#111827">OrderNexus — System Architecture</text>
  <text x="450" y="56" text-anchor="middle" font-size="12" fill="#6B7280">Event-driven microservices · Java 21 · Spring Boot 3.5 · Apache Kafka · Spring AI</text>

  <!-- CLIENT -->
  <rect x="360" y="76" width="180" height="44" rx="8" fill="#F3F4F6" stroke="#D1D5DB" stroke-width="1"/>
  <text x="450" y="102" text-anchor="middle" font-size="13" font-weight="600" fill="#374151">Client (REST / Swagger)</text>

  <line x1="450" y1="120" x2="450" y2="144" stroke="#6B7280" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- API GATEWAY -->
  <rect x="285" y="144" width="330" height="56" rx="8" fill="#EDE9FE" stroke="#7C3AED" stroke-width="1"/>
  <text x="450" y="167" text-anchor="middle" font-size="14" font-weight="600" fill="#4C1D95">API Gateway  :8080</text>
  <text x="450" y="186" text-anchor="middle" font-size="11" fill="#6D28D9">Spring Cloud Gateway · JWT validation · route filtering</text>

  <line x1="450" y1="200" x2="450" y2="224" stroke="#7C3AED" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- USER SERVICE -->
  <rect x="285" y="224" width="330" height="56" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="450" y="247" text-anchor="middle" font-size="14" font-weight="600" fill="#1E3A8A">User Service  :8081</text>
  <text x="450" y="266" text-anchor="middle" font-size="11" fill="#1D4ED8">Registration · Login · JWT issuance · BCrypt · RBAC</text>

  <line x1="450" y1="280" x2="450" y2="308" stroke="#2563EB" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- ORDER SERVICE -->
  <rect x="285" y="308" width="330" height="56" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="450" y="331" text-anchor="middle" font-size="14" font-weight="600" fill="#064E3B">Order Service  :8082</text>
  <text x="450" y="350" text-anchor="middle" font-size="11" fill="#065F46">Order lifecycle · Kafka saga orchestrator</text>

  <!-- Dashed lines FROM order-service to Kafka topics -->
  <path d="M285 336 L120 336 L120 490" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <path d="M450 364 L450 460" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <path d="M615 336 L780 336 L780 490" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <path d="M380 364 L200 364 L200 490" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>

  <!-- KAFKA BUS -->
  <rect x="60" y="430" width="780" height="204" rx="12" fill="#F0FDF4" stroke="#16A34A" stroke-width="1" stroke-dasharray="6 3"/>
  <text x="80" y="454" font-size="12" font-weight="600" fill="#15803D">Apache Kafka  —  message bus</text>

  <!-- Topic pills -->
  <rect x="80" y="464" width="170" height="34" rx="17" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="165" y="485" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">order.placed</text>

  <rect x="270" y="464" width="170" height="34" rx="17" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="355" y="485" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">inventory.response</text>

  <rect x="460" y="464" width="170" height="34" rx="17" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="545" y="485" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">payment.response</text>

  <rect x="650" y="464" width="170" height="34" rx="17" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="735" y="485" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">order.status.update</text>

  <rect x="200" y="564" width="140" height="34" rx="17" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="270" y="585" text-anchor="middle" font-size="12" font-weight="500" fill="#7F1D1D">order.placed.DLT</text>

  <rect x="360" y="564" width="180" height="34" rx="17" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="450" y="585" text-anchor="middle" font-size="12" font-weight="500" fill="#7F1D1D">inventory.response.DLT</text>

  <rect x="560" y="564" width="170" height="34" rx="17" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="645" y="585" text-anchor="middle" font-size="12" font-weight="500" fill="#7F1D1D">payment.response.DLT</text>

  <text x="450" y="625" text-anchor="middle" font-size="11" fill="#6B7280">Dead Letter Topics — captured after 3 retries with exponential backoff</text>

  <!-- Arrows from Kafka topics to downstream services -->
  <line x1="165" y1="634" x2="165" y2="680" stroke="#059669" stroke-width="1" marker-end="url(#garr)"/>
  <line x1="355" y1="634" x2="355" y2="680" stroke="#059669" stroke-width="1" marker-end="url(#garr)"/>
  <line x1="545" y1="634" x2="545" y2="680" stroke="#059669" stroke-width="1" marker-end="url(#garr)"/>
  <line x1="450" y1="634" x2="730" y2="680" stroke="#DC2626" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#rarr)"/>

  <!-- DOWNSTREAM SERVICES -->
  <rect x="75" y="680" width="180" height="56" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="165" y="703" text-anchor="middle" font-size="13" font-weight="600" fill="#1E3A8A">Inventory Service</text>
  <text x="165" y="720" text-anchor="middle" font-size="11" fill="#1D4ED8">Stock reservation · :8083</text>

  <rect x="275" y="680" width="180" height="56" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="365" y="703" text-anchor="middle" font-size="13" font-weight="600" fill="#1E3A8A">Notification Service</text>
  <text x="365" y="720" text-anchor="middle" font-size="11" fill="#1D4ED8">Email · SMS alerts · :8085</text>

  <rect x="465" y="680" width="180" height="56" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="555" y="703" text-anchor="middle" font-size="13" font-weight="600" fill="#1E3A8A">Payment Service</text>
  <text x="555" y="720" text-anchor="middle" font-size="11" fill="#1D4ED8">Charge · persist · :8084</text>

  <rect x="660" y="680" width="200" height="56" rx="8" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="760" y="703" text-anchor="middle" font-size="13" font-weight="600" fill="#7F1D1D">DLQ Intelligence</text>
  <text x="760" y="720" text-anchor="middle" font-size="11" fill="#B91C1C">Spring AI · GPT-4o-mini · :8086</text>

  <!-- Database labels -->
  <text x="165" y="750" text-anchor="middle" font-size="10" fill="#9CA3AF">inventorydb (MySQL)</text>
  <text x="365" y="750" text-anchor="middle" font-size="10" fill="#9CA3AF">notificationdb (MySQL)</text>
  <text x="555" y="750" text-anchor="middle" font-size="10" fill="#9CA3AF">paymentdb (MySQL)</text>
  <text x="760" y="750" text-anchor="middle" font-size="10" fill="#9CA3AF">dlqdb (MySQL)</text>

  <!-- Legend -->
  <line x1="60" y1="790" x2="90" y2="790" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <text x="96" y="794" font-size="11" fill="#374151">Kafka event</text>
  <line x1="180" y1="790" x2="210" y2="790" stroke="#374151" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="216" y="794" font-size="11" fill="#374151">HTTP / JWT</text>
  <line x1="296" y1="790" x2="326" y2="790" stroke="#DC2626" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#rarr)"/>
  <text x="332" y="794" font-size="11" fill="#374151">DLT failure path</text>
  <rect x="430" y="783" width="12" height="12" rx="2" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="448" y="794" font-size="11" fill="#374151">service</text>
  <rect x="498" y="783" width="12" height="12" rx="11" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="516" y="794" font-size="11" fill="#374151">Kafka topic</text>
  <rect x="596" y="783" width="12" height="12" rx="2" fill="#FEE2E2" stroke="#DC2626" stroke-width="1"/>
  <text x="614" y="794" font-size="11" fill="#374151">DLT / AI</text>

  <!-- Defs -->
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#6B7280" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="garr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#059669" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="rarr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#DC2626" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>
</svg>

</details>

The platform follows a layered architecture where all external traffic enters through the API Gateway, business logic lives in independent Spring Boot services, and all inter-service communication flows exclusively through Apache Kafka.

<details>
<summary><strong>Kafka event flow</strong></summary>
<br/>

<svg xmlns="http://www.w3.org/2000/svg" width="900" viewBox="0 0 900 760" style="background:#ffffff;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif">

  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#6B7280" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="garr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#059669" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="rarr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#DC2626" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="barr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#2563EB" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <rect width="900" height="760" fill="#ffffff"/>

  <text x="450" y="36" text-anchor="middle" font-size="18" font-weight="600" fill="#111827">Kafka Event Flow — Order Processing Saga</text>
  <text x="450" y="56" text-anchor="middle" font-size="12" fill="#6B7280">Choreography-based saga · no direct service-to-service HTTP calls</text>

  <!-- STEP 1: Customer -->
  <rect x="350" y="76" width="200" height="44" rx="8" fill="#F3F4F6" stroke="#D1D5DB" stroke-width="1"/>
  <text x="450" y="102" text-anchor="middle" font-size="13" font-weight="600" fill="#374151">Customer places order</text>
  <text x="450" y="115" text-anchor="middle" font-size="10" fill="#9CA3AF">POST /api/v1/orders</text>

  <line x1="450" y1="120" x2="450" y2="144" stroke="#6B7280" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- STEP 2: Order Service -->
  <rect x="310" y="144" width="280" height="52" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="450" y="168" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">Order Service  :8082</text>
  <text x="450" y="184" text-anchor="middle" font-size="11" fill="#065F46">creates order · status = PENDING</text>

  <line x1="450" y1="196" x2="450" y2="220" stroke="#059669" stroke-width="1.5" marker-end="url(#garr)"/>

  <!-- EVENT 1: order.placed -->
  <rect x="310" y="220" width="280" height="36" rx="18" fill="#D1FAE5" stroke="#059669" stroke-width="1.5"/>
  <text x="450" y="243" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">  order.placed</text>

  <!-- Fan out to 3 consumers -->
  <path d="M310 238 L130 238 L130 320" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <path d="M450 256 L450 320" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <path d="M590 238 L770 238 L770 320" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>

  <!-- 3 consumers -->
  <rect x="40" y="320" width="180" height="52" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="130" y="343" text-anchor="middle" font-size="13" font-weight="600" fill="#1E3A8A">Inventory Service</text>
  <text x="130" y="359" text-anchor="middle" font-size="11" fill="#1D4ED8">reserves stock · :8083</text>

  <rect x="360" y="320" width="180" height="52" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="450" y="343" text-anchor="middle" font-size="13" font-weight="600" fill="#1E3A8A">Notification Service</text>
  <text x="450" y="359" text-anchor="middle" font-size="11" fill="#1D4ED8">order confirmation · :8085</text>

  <rect x="680" y="320" width="180" height="52" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="770" y="343" text-anchor="middle" font-size="13" font-weight="600" fill="#1E3A8A">Payment Service</text>
  <text x="770" y="359" text-anchor="middle" font-size="11" fill="#1D4ED8">charges customer · :8084</text>

  <!-- Events from inventory and payment -->
  <line x1="130" y1="372" x2="130" y2="396" stroke="#059669" stroke-width="1.5" marker-end="url(#garr)"/>
  <line x1="770" y1="372" x2="770" y2="396" stroke="#059669" stroke-width="1.5" marker-end="url(#garr)"/>

  <rect x="40" y="396" width="180" height="36" rx="18" fill="#D1FAE5" stroke="#059669" stroke-width="1.5"/>
  <text x="130" y="419" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">inventory.response</text>

  <rect x="680" y="396" width="180" height="36" rx="18" fill="#D1FAE5" stroke="#059669" stroke-width="1.5"/>
  <text x="770" y="419" text-anchor="middle" font-size="12" font-weight="500" fill="#064E3B">payment.response</text>

  <!-- Both responses converge to Order Service -->
  <path d="M130 432 L130 476 L310 476" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <path d="M770 432 L770 476 L590 476" fill="none" stroke="#059669" stroke-width="1" stroke-dasharray="5 3" marker-end="url(#garr)"/>

  <rect x="310" y="456" width="280" height="52" rx="8" fill="#D1FAE5" stroke="#059669" stroke-width="1"/>
  <text x="450" y="479" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">Order Service  :8082</text>
  <text x="450" y="495" text-anchor="middle" font-size="11" fill="#065F46">updates status → PROCESSING</text>

  <line x1="450" y1="508" x2="450" y2="532" stroke="#059669" stroke-width="1.5" marker-end="url(#garr)"/>

  <!-- order.status.update -->
  <rect x="300" y="532" width="300" height="36" rx="18" fill="#D1FAE5" stroke="#059669" stroke-width="1.5"/>
  <text x="450" y="555" text-anchor="middle" font-size="13" font-weight="600" fill="#064E3B">order.status.update</text>

  <line x1="450" y1="568" x2="450" y2="592" stroke="#059669" stroke-width="1.5" marker-end="url(#garr)"/>

  <!-- Notification Service final alert -->
  <rect x="360" y="592" width="180" height="52" rx="8" fill="#DBEAFE" stroke="#2563EB" stroke-width="1"/>
  <text x="450" y="615" text-anchor="middle" font-size="13" font-weight="600" fill="#1E3A8A">Notification Service</text>
  <text x="450" y="631" text-anchor="middle" font-size="11" fill="#1D4ED8">status change alert</text>

  <!-- DLT failure path -->
  <path d="M220 390 L220 660 L310 660" fill="none" stroke="#DC2626" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#rarr)"/>
  <text x="170" y="528" text-anchor="middle" font-size="11" fill="#DC2626">on failure</text>
  <text x="170" y="543" text-anchor="middle" font-size="11" fill="#DC2626">(after 3 retries)</text>

  <rect x="310" y="636" width="280" height="52" rx="8" fill="#FEE2E2" stroke="#DC2626" stroke-width="1.5"/>
  <text x="450" y="659" text-anchor="middle" font-size="13" font-weight="600" fill="#7F1D1D">DLQ Intelligence Service</text>
  <text x="450" y="675" text-anchor="middle" font-size="11" fill="#B91C1C">Spring AI · root-cause analysis · :8086</text>

  <!-- Step numbers -->
  <circle cx="450" cy="88" r="10" fill="#6B7280"/>
  <text x="450" y="92" text-anchor="middle" font-size="10" font-weight="600" fill="#ffffff">1</text>

  <circle cx="450" cy="160" r="10" fill="#059669"/>
  <text x="450" y="164" text-anchor="middle" font-size="10" font-weight="600" fill="#ffffff">2</text>

  <circle cx="130" cy="336" r="10" fill="#2563EB"/>
  <text x="130" y="340" text-anchor="middle" font-size="10" font-weight="600" fill="#ffffff">3</text>

  <circle cx="770" cy="336" r="10" fill="#2563EB"/>
  <text x="770" y="340" text-anchor="middle" font-size="10" font-weight="600" fill="#ffffff">4</text>

  <circle cx="450" cy="472" r="10" fill="#059669"/>
  <text x="450" y="476" text-anchor="middle" font-size="10" font-weight="600" fill="#ffffff">5</text>

  <circle cx="450" cy="608" r="10" fill="#2563EB"/>
  <text x="450" y="612" text-anchor="middle" font-size="10" font-weight="600" fill="#ffffff">6</text>

  <!-- Legend -->
  <line x1="60" y1="730" x2="90" y2="730" stroke="#059669" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#garr)"/>
  <text x="96" y="734" font-size="11" fill="#374151">Kafka event</text>
  <line x1="180" y1="730" x2="210" y2="730" stroke="#6B7280" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="216" y="734" font-size="11" fill="#374151">HTTP call</text>
  <line x1="290" y1="730" x2="320" y2="730" stroke="#DC2626" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#rarr)"/>
  <text x="326" y="734" font-size="11" fill="#374151">DLT failure path (after retries)</text>

</svg>

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
