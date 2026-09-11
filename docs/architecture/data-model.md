# LogiTrack — Data Model

## 1. Database Strategy

LogiTrack follows a database-per-service logical separation.

Each microservice owns and manages its own data.

Services must not directly access another service's database.

Primary database technology:

- PostgreSQL

Optional caching:

- Redis

Redis may be used for read-heavy tracking data.


## 2. Identity Service Database

### User

Stores user account information.

| Field | Description |
|---|---|
| user_id | Unique user identifier |
| name | User's name |
| email | User's unique email |
| password_hash | BCrypt-hashed password |
| status | User account status |
| created_at | Account creation timestamp |

### Role

Stores application roles.

| Field | Description |
|---|---|
| role_id | Unique role identifier |
| role_name | Name of the role |

### Refresh Token

Stores refresh token information.

| Field | Description |
|---|---|
| refresh_token_id | Unique refresh token identifier |
| user_id | Associated user |
| token | Refresh token |
| created_at | Token creation time |
| expires_at | Token expiration time |


## 3. Shipment Service Database

### Shipment

Stores the master shipment information.

| Field | Description |
|---|---|
| shipment_id | Unique shipment identifier |
| tracking_number | Unique tracking number |
| customer_id | Customer associated with shipment |
| sender_address_id | Sender address reference |
| receiver_address_id | Receiver address reference |
| package_type | Type of package |
| weight | Package weight |
| status | Current shipment status |
| created_at | Shipment creation timestamp |
| updated_at | Last update timestamp |
| version | Version for data consistency |

### Address

Stores shipment address information.

| Field | Description |
|---|---|
| address_id | Unique address identifier |
| name | Associated person's name |
| city | City |
| postal_code | Postal code |

### Package

Stores package information.

| Field | Description |
|---|---|
| package_id | Unique package identifier |
| package_type | Type of package |
| weight | Package weight |


## 4. Tracking Service Database

### Tracking Event

Stores the chronological shipment tracking history.

| Field | Description |
|---|---|
| event_id | Unique event identifier |
| shipment_id | Associated shipment |
| status | Shipment status at the event |
| location | Event location |
| event_time | Time of tracking event |
| remarks | Additional remarks |
| correlation_id | Request/event correlation identifier |

Tracking Service is responsible for tracking event data.

It does not become the owner of the shipment's current lifecycle state.


## 5. Delivery Service Database

### Delivery Agent

Stores delivery agent information.

| Field | Description |
|---|---|
| agent_id | Unique agent identifier |
| name | Agent name |
| phone | Agent phone number |
| vehicle_number | Assigned vehicle |
| availability_status | Current availability |

### Assignment

Stores delivery-agent assignments.

| Field | Description |
|---|---|
| assignment_id | Unique assignment identifier |
| shipment_id | Associated shipment |
| agent_id | Assigned delivery agent |
| assigned_at | Assignment timestamp |
| status | Assignment status |

### Delivery Confirmation

Stores successful delivery confirmation.

| Field | Description |
|---|---|
| confirmation_id | Unique confirmation identifier |
| shipment_id | Associated shipment |
| receiver_name | Person receiving shipment |
| delivered_at | Delivery timestamp |
| proof_reference | Optional proof-of-delivery reference |
| remarks | Delivery remarks |


## 6. Notification Service Database

### Notification Log

Stores generated notification records.

| Field | Description |
|---|---|
| notification_id | Unique notification identifier |
| event_type | Event that triggered notification |
| recipient | Notification recipient |
| status | Notification processing status |
| created_at | Notification creation timestamp |

### Notification Template

Stores notification templates.

| Field | Description |
|---|---|
| template_id | Unique template identifier |
| event_type | Associated event |
| template_content | Notification content |
| status | Template status |


## 7. Reporting Service Database

The Reporting Service maintains operational reporting projections
or a read model.

The SRS specifies reporting projections/read-model data rather than
a detailed table structure.

The reporting model will be designed during implementation based on
the dashboard and reporting requirements.


## 8. Service Data Ownership

| Service | Owned Data |
|---|---|
| API Gateway | None |
| Identity Service | users, roles, refresh_tokens |
| Shipment Service | shipments, addresses, packages |
| Tracking Service | tracking_events |
| Delivery Service | delivery_agents, assignments, delivery_confirmations |
| Notification Service | notification_log, templates |
| Reporting Service | reporting projections / read model |


## 9. Logical Relationships

### Shipment → Customer

A shipment is associated with a customer through customer_id.

The customer itself is owned by the Identity Service.

The Shipment Service should not directly access the Identity Service
database.

### Shipment → Addresses

A shipment references:

- sender_address_id
- receiver_address_id

Both addresses belong to the Shipment Service.

### Assignment → Shipment

An assignment references a shipment through shipment_id.

### Assignment → Delivery Agent

An assignment references an agent through agent_id.

### Delivery Confirmation → Shipment

A delivery confirmation references the completed shipment through
shipment_id.

### Tracking Event → Shipment

A tracking event references a shipment through shipment_id.

The tracking service receives shipment lifecycle information through
events rather than directly accessing the Shipment Service database.


## 10. Cross-Service Data Rule

Cross-service references such as customer_id, shipment_id, and agent_id
are identifiers only.

They do not create direct database relationships across services.

For example:

Shipment Service
    |
    | customer_id
    |
    X
    |
Identity Service Database

The Shipment Service stores the identifier but does not use a foreign
key to the Identity Service database.


## 11. Shipment Status

The Shipment Service owns the authoritative shipment status.

Valid lifecycle:

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

Invalid transitions must be rejected.


## 12. Data Consistency

Cross-service lifecycle updates use eventual consistency.

Kafka events propagate changes between services.

Example:

Shipment Service
      |
      | ShipmentStatusChanged
      v
Kafka Event Bus
      |
      +------> Tracking Service
      |
      +------> Notification Service
      |
      +------> Reporting Service


## 13. Caching

Redis may be used by the Tracking Service for read-heavy tracking
queries.

Example:

Client
  |
  v
Tracking Service
  |
  +----> Redis Cache
  |
  +----> PostgreSQL