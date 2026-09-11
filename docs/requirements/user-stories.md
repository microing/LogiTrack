# LogiTrack — User Stories

## 1. Customer User Stories

### US-C01 — User Registration
As a customer, I want to register an account so that I can use LogiTrack.

### US-C02 — User Login
As a customer, I want to log in securely so that I can access my shipments.

### US-C03 — Create Shipment
As a customer, I want to create a shipment so that I can send a package.

### US-C04 — Tracking Number
As a customer, I want to receive a unique tracking number so that I can track my shipment.

### US-C05 — View My Shipments
As a customer, I want to view my shipments so that I can monitor my deliveries.

### US-C06 — Track Shipment
As a customer, I want to search for a shipment using its tracking number so that I can see its current status.

### US-C07 — Tracking Timeline
As a customer, I want to view the tracking timeline so that I can see the shipment's history.

### US-C08 — Update Shipment
As a customer, I want to update eligible shipment details before pickup so that incorrect information can be corrected.

### US-C09 — Cancel Shipment
As a customer, I want to cancel an eligible shipment so that I can stop a shipment before pickup.

### US-C10 — Notifications
As a customer, I want to receive shipment notifications so that I know when important shipment events occur.


## 2. Operations Manager User Stories

### US-M01 — View Shipments
As an operations manager, I want to view all shipments so that I can monitor logistics operations.

### US-M02 — Search and Filter
As an operations manager, I want to search and filter shipments so that I can find shipments quickly.

### US-M03 — Assign Delivery Agent
As an operations manager, I want to assign an available delivery agent to a shipment so that the shipment can be delivered.

### US-M04 — Update Shipment
As an operations manager, I want to update permitted shipment information so that operational data remains accurate.

### US-M05 — Monitor Shipment Status
As an operations manager, I want to monitor shipment statuses so that I can identify delays.

### US-M06 — Monitor SLA
As an operations manager, I want to view SLA indicators so that I can identify SLA exceptions.

### US-M07 — Operational Reports
As an operations manager, I want to view operational reports so that I can understand delivery performance.


## 3. Delivery Agent User Stories

### US-A01 — View Assigned Shipments
As a delivery agent, I want to view shipments assigned to me so that I know which deliveries I need to complete.

### US-A02 — Update Shipment Status
As a delivery agent, I want to update shipment status so that the system reflects the current delivery progress.

### US-A03 — Record Receiver
As a delivery agent, I want to record receiver information so that delivery completion is documented.

### US-A04 — Delivery Details
As a delivery agent, I want to record delivery time and remarks so that the delivery has proper confirmation details.

### US-A05 — Proof of Delivery
As a delivery agent, I want to submit proof-of-delivery information so that successful delivery can be verified.


## 4. System Administrator User Stories

### US-AD01 — Manage Users
As a system administrator, I want to manage users so that the platform remains controlled.

### US-AD02 — Manage Roles
As a system administrator, I want to manage user roles so that users have appropriate permissions.

### US-AD03 — View Audit Activity
As a system administrator, I want to view audit activity so that security-sensitive actions can be traced.

### US-AD04 — Configure Reference Data
As a system administrator, I want to configure reference data so that the system can maintain required configuration.


## 5. Shipment Lifecycle

The shipment follows this lifecycle:

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

Invalid status transitions must be rejected by the backend without changing the existing shipment status.