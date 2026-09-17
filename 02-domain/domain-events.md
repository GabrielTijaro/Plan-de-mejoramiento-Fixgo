# Domain Events

> **What to fill in here:** A domain event is a fact that occurred in the business.
> They are the backbone of asynchronous communication between bounded contexts.
> The name is ALWAYS in past tense and in the ubiquitous language of the domain.

---

## What is a domain event?

A **Domain Event** communicates that something important occurred in the business.
It is an immutable message that describes the fact in past tense.

```
✓ OrderCreated
✓ PaymentRejected
✓ UserRegistered
✓ StockDepleted

✗ CreateOrder (this is a command, not an event)
✗ OrderUpdated (too generic — what changed?)
✗ OrderEvent (does not indicate what occurred)
```

### Difference between Command and Event

| Concept | Intent | Tense | Can fail? |
|---------|--------|-------|-----------|
| **Command** | Instruction to do something | Present | Yes |
| **Event** | Notification of something that occurred | Past | No (it already happened) |

```
User → [CreateOrder] → System → [OrderCreated] → Other contexts
          (Command)                  (Event)
```

---

## Event catalog

### Event: DriverRegistered
* **Triggered by:** Driver completes sign-up.
* **Data:** `driverId`, `email`
* **Consumers:** Matchmaking & Dispatch
* **Bounded Context:** User Management
### Event: VehicleRegistered
* **Triggered by:** Driver registers a vehicle.
* **Data:** `vehicleId`, `driverId`, `plate`
* **Consumers:** Matchmaking & Dispatch
* **Bounded Context:** User Management
### Event: ServiceRequested
* **Triggered by:** Driver creates a request.
* **Data:** `requestId`, `driverId`, `vehicleId`, `location`
* **Consumers:** Service Execution
* **Bounded Context:** Matchmaking & Dispatch
### Event: MechanicMatched
* **Triggered by:** Algorithm assigns a mechanic within the SLA.
* **Data:** `requestId`, `mechanicId`, `eta`
* **Consumers:** Service Execution, Driver UI
* **Bounded Context:** Matchmaking & Dispatch
### Event: ServiceCompleted
* **Triggered by:** Mechanic submits the diagnostic.
* **Data:** `requestId`, `diagnosticId`, `completionTime`
* **Consumers:** User Management (history)
* **Bounded Context:** Service Execution
**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "[EventName]",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "[AggregateName]",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "[field1]": "[type and description]",
    "[field2]": "[type and description]"
  },
  "metadata": {
    "correlationId": "550e8400-e29b-41d4-a716-446655440002",
    "causationId": "550e8400-e29b-41d4-a716-446655440003",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "[EventName]",
  "aggregateId": "aggregate-uuid",
  "aggregateType": "[AggregateName]",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "[field1]": "example value",
    "[field2]": 150.00
  }
}
```

**What do consumers do with this event?**

| Event | Consuming context | Action | Idempotent? |
|-------|-------------------|--------|-------------|
| `DriverRegistered` | Matchmaking & Dispatch | Creates the driver's profile shell so a future `ServiceRequested` can reference a known `driverId` | Yes — `driverId` is the natural key |
| `VehicleRegistered` | Matchmaking & Dispatch | Caches the vehicle record so a `ServiceRequest` can validate `vehicleId` without a synchronous call to `auth-service` | Yes — `vehicleId` is the natural key |
| `ServiceRequested` | Service Execution | Reserves a slot for the future `Diagnostic` once a mechanic is matched | Yes — `requestId` is the natural key |
| `MechanicMatched` | Service Execution | Assigns the request to the mechanic's active queue | Yes — `requestId` is the natural key |
| `MechanicMatched` | Driver UI (push) | Triggers the FCM notification "a mechanic is on the way" (HU-11) | Yes — one notification per `requestId` transition |
| `ServiceCompleted` | User Management | Appends the closed request + diagnostic to the vehicle's service history | Yes — `diagnosticId` is the natural key |

---

## Standard fields for all events

All events must include these fields in the envelope:

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | UUID | Unique event ID (for idempotency) |
| `eventType` | string | Event name in PascalCase |
| `aggregateId` | UUID | ID of the aggregate that generated the event |
| `aggregateType` | string | Aggregate type |
| `occurredAt` | ISO 8601 | When the business fact occurred |
| `version` | integer | Schema version (for evolution) |
| `payload` | object | Event data (specific per type) |
| `metadata.correlationId` | UUID | For tracing a transaction across services |
| `metadata.causationId` | UUID | ID of the event or command that caused this event |
| `metadata.userId` | UUID | User who initiated the chain (if applicable) |

---

## Event flow: Service request lifecycle

```

[Driver]                                        [Mechanic]
   │                                                 │
   │ CreateServiceRequest (command)                  │
   ▼                                                 │
[Aggregate: ServiceRequest]                          │
   │                                                 │
   │ ServiceRequested (event)                        │
   ▼                                                 │
[Policy: Matchmaking]                                │
   │ assigns nearest AVAILABLE, verified mechanic    │
   ▼                                                 │
[Aggregate: ServiceRequest]                          │
   │                                                 │
   │ MechanicMatched (event) ───────────────────────▶│
   │                                                  │
   │                              SubmitDiagnostic (command)
   │                                                  ▼
   │                                    [Aggregate: Diagnostic]
   │                                                  │
   │◀───────────────────────── ServiceCompleted (event)
   ▼
[User Management: service history updated]

```

---

## Schema evolution strategy

Events are contracts. Changing them in an incompatible way breaks consumers.

### What is a compatible change (does not break)?

```
✓ Add a new optional field to the payload
✓ Add a new event type
✓ Change a required field → optional
```

### What is an incompatible change (breaks)?

```
✗ Remove a field from the payload
✗ Change the type of a field (string → number)
✗ Change an optional field → required
✗ Change the event name
```

### How to evolve a schema without breaking consumers

**Strategy: Version the event**

```
Step 1: Publish EventV2 (new type with incompatible changes)
Step 2: Publish both EventV1 and EventV2 during the migration period
Step 3: Migrate consumers to V2 one by one
Step 4: Deprecate EventV1 (announce 1 sprint in advance)
Step 5: Stop publishing EventV1
```

---

## Event summary table

| Event | Origin Context | Topic | Consumers | Version |
|-------|---------------|-------|-----------|---------|
| DriverRegistered | User Management | `fixgo.users.driver-registered` | Matchmaking & Dispatch | v1 |
| VehicleRegistered | User Management | `fixgo.users.vehicle-registered` | Matchmaking & Dispatch | v1 |
| ServiceRequested | Matchmaking & Dispatch | `fixgo.services.requested` | Service Execution | v1 |
| MechanicMatched | Matchmaking & Dispatch | `fixgo.services.mechanic-matched` | Service Execution, Driver UI | v1 |
| ServiceCompleted | Service Execution | `fixgo.services.completed` | User Management | v1 | 

---

## Policies — Reactions to events

A **Policy** (or Saga step) describes what happens automatically when an event arrives.
It is the logic of "whenever X occurs, do Y".

```
Event:  OrderCreated
Policy: Whenever an OrderCreated arrives with type=URGENT,
        emit the command NotifyOperationsTeam
```

| Trigger event | Policy | Emitted command | Service |
|---------------|--------|------------------|---------|
| `ServiceRequested` | Whenever a request is created, find the nearest `AVAILABLE` and verified Mechanic within the SLA (`04-requirements/non-functional.md`) | `AssignMechanic` | `service-request` |
| `MechanicMatched` | Whenever a mechanic is assigned, notify the Driver of the ETA (HU-11) | `SendPushNotification` | `service-request` (via FCM) |
| `ServiceCompleted` | Whenever a diagnostic is submitted, close the request and append it to the vehicle's history | `UpdateServiceHistory` | `auth-service` |
 

---

## Resilience patterns for events

### At-least-once delivery + Idempotency

The message broker guarantees the event is delivered **at least once** but it may be
delivered more than once (in case of retries). Consumers must be **idempotent**.

```typescript
// Idempotent consumer — stores the processed eventId
async function processOrderCreatedEvent(event: OrderCreated): Promise<void> {
  // 1. Check if already processed
  if (await isEventAlreadyProcessed(event.eventId)) {
    logger.info(`Event ${event.eventId} already processed, ignoring`);
    return;
  }

  // 2. Process the event
  await updateModel(event.payload);

  // 3. Mark as processed (in the same transaction)
  await markEventProcessed(event.eventId);
}
```

### Dead Letter Queue (DLQ)

When an event fails after N retries, it goes to the DLQ.

| Configuration | Recommended value |
|--------------|------------------|
| Retries before DLQ | 3-5 |
| Backoff | Exponential (1s → 2s → 4s → 8s) |
| DLQ retention | 7 days |
| Alert | When DLQ has > 0 messages |

> See DLQ runbook in `09-microservices/services/XX-service/runbook.md`
