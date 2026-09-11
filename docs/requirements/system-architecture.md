# LogiTrack — System Architecture

## 1. System Context

LogiTrack is a logistics and shipment tracking platform that allows
customers and logistics operators to create, manage, track, assign,
and complete shipments.

## 2. System Actors

### Customer
- Register and sign in
- Create shipments
- View own shipments
- Track shipments
- Update eligible shipment details
- Cancel eligible shipments
- View notifications

### Operations Manager
- View and manage shipments
- Assign delivery agents
- Monitor shipment lifecycle
- Monitor SLA indicators
- View operational reports

### Delivery Agent
- View assigned shipments
- Update delivery status
- Submit delivery confirmation
- Provide proof-of-delivery reference

### System Administrator
- Manage users
- Manage roles
- View audit activity
- Configure reference data

## 3. High-Level System Flow

Customer / Manager / Agent / Administrator
                    |
                    v
             React Web Application
                    |
                    v
               API Gateway
                    |
       +------------+------------+
       |            |            |
       v            v            v
 Identity      Shipment      Tracking
 Service       Service       Service
       |            |            |
       +------------+------------+
                    |
                    v
                Kafka Event Bus
                    |
       +------------+------------+
       |            |            |
       v            v            v
  Delivery     Notification   Reporting
   Service       Service       Service

Each service owns its data.

PostgreSQL is used for service data and Redis may be used
for caching read-heavy tracking data.

## 4. Communication

### Synchronous Communication
Services expose versioned REST APIs.

### Asynchronous Communication
Kafka events are used to propagate shipment lifecycle changes.

### Frontend Communication
The React application communicates with backend services
through the API Gateway.

## 5. Core Architecture Principles

- API-first development
- Independent microservices
- Database-per-service logical separation
- Eventual consistency for cross-service updates
- Idempotent event consumers
- Correlation IDs for traceability
- Secure access through the API Gateway