# LogiTrack — Microservice Boundaries

## 1. Architecture Overview

LogiTrack follows an API-first, event-driven microservice architecture.

Each service owns its own data and exposes versioned REST APIs.
Kafka is used for asynchronous shipment lifecycle events.

The system contains:

1. API Gateway
2. Identity Service
3. Shipment Service
4. Tracking Service
5. Delivery Service
6. Notification Service
7. Reporting Service


## 2. API Gateway

### Responsibility

The API Gateway is the single entry point for the React frontend.

### Responsibilities

- Route requests to backend services
- Validate authentication tokens
- Handle CORS
- Generate/propagate request correlation IDs
- Optional rate limiting
- Provide secure access to backend services

### Owned Data

None.

The API Gateway does not own business data.


## 3. Identity Service

### Responsibility

Manages users, authentication, authorization, profiles, and roles.

### Responsibilities

- User registration
- User login
- JWT authentication
- Refresh tokens
- User profiles
- Role management
- User status management

### Owned Data

- users
- roles
- refresh_tokens

### Main Roles

- CUSTOMER
- OPERATIONS_MANAGER
- DELIVERY_AGENT
- SYSTEM_ADMINISTRATOR


## 4. Shipment Service

### Responsibility

Owns the main shipment lifecycle and shipment master data.

### Responsibilities

- Create shipments
- Update permitted shipment details
- Retrieve shipments
- Search shipments
- Generate tracking numbers
- Cancel eligible shipments
- Validate shipment status transitions

### Owned Data

- shipments
- addresses
- packages

### Shipment Lifecycle

CREATED
→ PICKUP_SCHEDULED
→ PICKED_UP
→ IN_WAREHOUSE
→ IN_TRANSIT
→ OUT_FOR_DELIVERY
→ DELIVERED

Alternative terminal states:

- CANCELLED
- RETURNED

The Shipment Service is the authoritative owner of shipment lifecycle state.


## 5. Tracking Service

### Responsibility

Provides a read-optimized shipment tracking timeline.

### Responsibilities

- Maintain tracking events
- Consume shipment lifecycle events
- Build chronological tracking history
- Provide tracking information
- Support read-optimized tracking queries

### Owned Data

- tracking_events

### Important Rule

The Tracking Service does not become the owner of shipment status.

Shipment lifecycle state is owned by the Shipment Service.
Tracking Service maintains the tracking history based on events.


## 6. Delivery Service

### Responsibility

Manages delivery agents, assignments, and delivery confirmation.

### Responsibilities

- Manage delivery agents
- Track agent availability
- Assign agents to shipments
- Update delivery status
- Record delivery confirmation
- Record proof-of-delivery reference

### Owned Data

- delivery_agents
- assignments
- delivery_confirmations

### Delivery Agent Workflow

Available Agent
      ↓
Assigned to Shipment
      ↓
Update Delivery Status
      ↓
Confirm Delivery


## 7. Notification Service

### Responsibility

Processes shipment-related events and generates notifications.

### Responsibilities

- Consume Kafka events
- Generate email or simulated notifications
- Record notification history
- Manage notification templates

### Owned Data

- notification_log
- templates

### Example Events

- ShipmentCreated
- ShipmentStatusChanged
- DeliveryAgentAssigned
- ShipmentDelivered


## 8. Reporting Service

### Responsibility

Provides operational reporting and dashboard data.

### Responsibilities

- Calculate shipment aggregates
- Provide operational dashboard data
- Track shipment status distribution
- Track delivery completion
- Identify SLA exceptions
- Maintain reporting projections/read models

### Owned Data

- reporting projections
- reporting read model


## 9. Data Ownership Rules

Each microservice owns its own data.

Services must not directly access another service's database.

Example:

Shipment Service
      ↓
PostgreSQL
      ↓
shipments

Tracking Service
      ↓
PostgreSQL
      ↓
tracking_events

Delivery Service
      ↓
PostgreSQL
      ↓
delivery_agents
assignments
delivery_confirmations


## 10. Communication Rules

### Synchronous Communication

REST APIs are used when an immediate response is required.

Example:

React
  ↓
API Gateway
  ↓
Shipment Service
  ↓
Create Shipment Response


### Asynchronous Communication

Kafka events are used to propagate shipment lifecycle changes.

Example:

Shipment Service
      ↓
ShipmentCreated
      ↓
Kafka
      ↓
Tracking Service
      ↓
Notification Service


## 11. Core Events

### ShipmentCreated

Producer:
- Shipment Service

Consumers:
- Tracking Service
- Notification Service

Purpose:
- Initialize tracking timeline
- Notify customer


### ShipmentStatusChanged

Producer:
- Shipment Service
- Delivery Service

Consumers:
- Tracking Service
- Notification Service
- Reporting Service

Purpose:
- Propagate shipment lifecycle changes


### DeliveryAgentAssigned

Producer:
- Delivery Service

Consumers:
- Notification Service
- Reporting Service

Purpose:
- Record assignment
- Notify relevant users


### ShipmentDelivered

Producer:
- Delivery Service

Consumers:
- Tracking Service
- Notification Service
- Reporting Service

Purpose:
- Close shipment lifecycle
- Update reporting data


## 12. Service Dependency Principle

Services should remain loosely coupled.

A service should communicate with another service through:

1. Versioned REST APIs when synchronous communication is required.
2. Kafka events when asynchronous communication is appropriate.

A service must not directly access another service's database.


## 13. Implementation Priority

Because this is a single-student internship project, implementation will be prioritized.

### Phase 1 — Core Services

1. API Gateway
2. Identity Service
3. Shipment Service
4. Tracking Service
5. Delivery Service

### Phase 2 — Lightweight Services

6. Notification Service
7. Reporting Service

Notification and Reporting will initially remain lightweight and will be expanded after the core shipment workflow is working.