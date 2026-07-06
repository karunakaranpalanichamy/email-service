# email-service

# Event-Driven Order Processing System

An event-driven microservices application built using **Java**, **Spring Boot**, and **RabbitMQ** that demonstrates asynchronous communication between distributed services.

The system enables users to create customer orders through REST APIs. Once an order is placed, the application publishes an event to RabbitMQ. The Stock Service reserves the ordered items and, upon successful reservation, another event is published to notify the Email Service, which simulates sending shipment instructions to the shipper.

## Architecture

The application follows an **Event-Driven Architecture (EDA)** where services communicate asynchronously through RabbitMQ events instead of making direct synchronous REST calls.

```text
                    REST API
                       │
                       ▼
               +----------------+
               | Order Service  |
               +----------------+
                       │
         Publish Order Created Event
                       │
                       ▼
                  RabbitMQ Exchange
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
 +----------------+         +----------------+
 | Stock Service  |         | Other Consumers|
 +----------------+         +----------------+
          │
 Reserve Inventory
          │
 Publish Stock Reserved Event
          │
          ▼
      RabbitMQ Exchange
          │
          ▼
 +----------------+
 | Email Service  |
 +----------------+
          │
 Sends Shipment Notification
          │
          ▼
      Shipper System
```

---

# Features

- Create customer orders via REST API
- Reserve product inventory
- Send shipment notification events
- Asynchronous service communication
- Event-driven workflow using RabbitMQ
- Loosely coupled microservices
- RESTful API design
- Message-driven architecture
- Spring Boot based implementation

---

# Microservices

## Order Service

Responsible for:

- Creating customer orders
- Validating order requests
- Persisting order information
- Publishing **Order Created** events to RabbitMQ

### REST Endpoints

| Method | Endpoint | Description |
|---------|----------|-------------|
| POST | `/api/orders` | Create a new order |
| GET | `/api/orders` | Retrieve all orders |
| GET | `/api/orders/{id}` | Retrieve order by ID |

---

## Stock Service

Responsible for:

- Listening for Order Created events
- Reserving inventory
- Updating stock quantity
- Publishing Stock Reserved events

No public REST endpoints are required for the event processing flow, although administrative APIs may be exposed.

Example:

| Method | Endpoint | Description |
|---------|----------|-------------|
| GET | `/api/stocks` | View inventory |
| PUT | `/api/stocks/{productId}` | Update stock |

---

## Email Service

Responsible for:

- Listening for Stock Reserved events
- Sending shipment notification
- Triggering shipper workflow
- Logging email status

Example endpoint (optional):

| Method | Endpoint | Description |
|---------|----------|-------------|
| GET | `/api/emails` | View sent notifications |

---

# Event Flow

```text
Client
   │
POST /api/orders
   │
   ▼
Order Service
   │
Store Order
   │
Publish OrderCreated Event
   │
   ▼
RabbitMQ
   │
   ▼
Stock Service
   │
Reserve Inventory
   │
Publish StockReserved Event
   │
   ▼
RabbitMQ
   │
   ▼
Email Service
   │
Notify Shipper
   │
Shipment Initiated
```

---

# Technology Stack

- Java 17+
- Spring Boot
- Spring Web
- Spring Data JPA
- RabbitMQ
- Spring AMQP
- Maven
- REST APIs
- H2 / MySQL / PostgreSQL
- Jackson JSON

---

# Project Structure

```
event-driven-order-system
│
├── order-service
│   ├── controller
│   ├── service
│   ├── repository
│   ├── entity
│   ├── event
│   └── config
│
├── stock-service
│   ├── listener
│   ├── service
│   ├── repository
│   ├── entity
│   └── config
│
├── email-service
│   ├── listener
│   ├── service
│   ├── email
│   └── config
│
└── common
    ├── events
    ├── dto
    └── messaging
```

---

# Prerequisites

- Java 17 or later
- Maven 3.8+
- RabbitMQ Server
- MySQL, PostgreSQL, or H2 Database

---

# Running RabbitMQ

Using Docker:

```bash
docker run -d \
--hostname rabbitmq \
--name rabbitmq \
-p 5672:5672 \
-p 15672:15672 \
rabbitmq:3-management
```

RabbitMQ Management Console

```
http://localhost:15672
```

Default credentials

```
Username: guest
Password: guest
```

---

# Running the Application

Clone the repository

```bash
git clone https://github.com/yourusername/event-driven-order-system.git
```

Navigate to the project

```bash
cd event-driven-order-system
```

Build the project

```bash
mvn clean install
```

Run each service independently

```bash
mvn spring-boot:run
```

or

```bash
java -jar target/order-service.jar
```

Repeat for:

- order-service
- stock-service
- email-service

---

# Configuration

Example `application.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/orders
spring.datasource.username=root
spring.datasource.password=password

spring.jpa.hibernate.ddl-auto=update

spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

---

# Sample API

## Create Order

**POST**

```
/api/orders
```

Request Body

```json
{
  "customerName": "John Doe",
  "productCode": "LAPTOP-001",
  "quantity": 2
}
```

Response

```json
{
  "orderId": 1001,
  "status": "CREATED"
}
```

---

# Messaging Events

## OrderCreatedEvent

```json
{
  "orderId": 1001,
  "productCode": "LAPTOP-001",
  "quantity": 2
}
```

Published by:

- Order Service

Consumed by:

- Stock Service

---

## StockReservedEvent

```json
{
  "orderId": 1001,
  "status": "RESERVED"
}
```

Published by:

- Stock Service

Consumed by:

- Email Service

---

# RabbitMQ Messaging Topology

```
Order Service
      │
      ▼
order.exchange
      │
      ▼
order.created.queue
      │
      ▼
Stock Service
      │
      ▼
stock.exchange
      │
      ▼
stock.reserved.queue
      │
      ▼
Email Service
```

---

# HTTP Status Codes

| Status | Description |
|---------|-------------|
| 200 OK | Request processed successfully |
| 201 Created | Order created successfully |
| 202 Accepted | Event accepted for processing |
| 400 Bad Request | Invalid request |
| 404 Not Found | Resource not found |
| 409 Conflict | Inventory unavailable |
| 500 Internal Server Error | Unexpected server error |

---

# Benefits of Event-Driven Architecture

- Loose coupling between services
- High scalability
- Improved fault tolerance
- Independent deployment of services
- Asynchronous processing
- Better system resilience
- Easy integration with additional consumers
- Supports horizontal scaling

---

# Future Enhancements

- Payment Service
- Shipping Service
- Notification Service (SMS/Push)
- Dead Letter Queues (DLQ)
- Retry Mechanism
- Distributed Tracing (Zipkin/OpenTelemetry)
- API Gateway
- Service Discovery
- Docker & Docker Compose
- Kubernetes Deployment
- Swagger/OpenAPI Documentation
- Spring Cloud Config
- Kafka support
- Circuit Breaker using Resilience4j
- Monitoring with Prometheus and Grafana

---

# Testing

Run all tests

```bash
mvn test
```

Run integration tests

```bash
mvn verify
```

---



# Author Karunakaran Palanichamy


Developed using **Java**, **Spring Boot**, **RabbitMQ**, and **Event-Driven Microservices Architecture**.
