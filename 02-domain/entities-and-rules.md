# Entities, Value Objects, and Business Rules

> **What to fill in here:** The building blocks of the domain following the DDD tactical model.
> This document translates domain knowledge (obtained in Event Storming) into code models.

> **Stack note:** The concepts of Entity, Value Object, and Aggregate are language-independent.
> Code examples (classes, interfaces, decorators) are written in pseudo-TypeScript
> to illustrate the idea. To see the implementation in your technology:
> [`_stacks/node-typescript.md`](../_stacks/node-typescript.md) ·
> [`_stacks/java-spring.md`](../_stacks/java-spring.md) ·
> [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md) ·
> [`_stacks/go.md`](../_stacks/go.md)

---

## Tactical DDD concepts

### Entity
An **Entity** is an object defined by its identity, not its attributes.
Two entities are equal if they have the same ID, even if all their other attributes differ.

```
✓ Entity: User (two users with different emails are still distinct by their ID)
✓ Entity: Order (changes state but remains the same order)
✗ Not an entity: Money (10 USD == 10 USD regardless of which bill)
```

### Value Object (VO)
A **Value Object** is an object defined by its attributes; it has no identity of its own.
It is immutable — if an attribute changes, it is a new VO.

```
✓ Value Object: Address (5th Street #10-20, Neiva, Huila)
✓ Value Object: Money (USD 150.00)
✓ Value Object: Email (user@example.com)
✓ Value Object: DateRange (2024-01-01 → 2024-01-31)
```

### Aggregate
An **Aggregate** is a cluster of entities and VOs treated as a unit.
It has an **Aggregate Root** which is the entry point — internal objects can only be
accessed through the root.

```
Order (Aggregate Root)
  ├── OrderItems[] (Entities inside the aggregate)
  ├── DeliveryAddress (Value Object)
  └── OrderTotal (Calculated Value Object)
```

**Golden rule of the Aggregate:** Transactions do not cross aggregate boundaries.
If you need to modify two aggregates in one operation, use a Domain Event and a Saga.

### Business Rules
**Business Rules** (invariants) are the constraints the domain must always satisfy.
They live in the Aggregate Root and are validated on every operation.

---

## System entities

### Entity: Driver
 
**Context:** User Management
 
**Description:** A registered user who owns at least one vehicle and can create service requests.
 
**Attributes:**
 
| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| userId | UUID | Reference to the base authenticated account | Yes | Must reference a valid Firebase Auth user |
| createdAt | DateTime | Creation date | Yes | Immutable, set on creation |
| updatedAt | DateTime | Last modification | Yes | Updated automatically |
 
**Lifecycle / States:**
 
No formal state machine — a Driver account is either created or not; deactivation happens
at the underlying `userId` account level (SRS Module 1), not on the Driver entity itself.
 
**Invariants (Business rules that MUST ALWAYS hold):**
 
```
INV-005: Cannot request service without an active vehicle
  - Rule: A Driver cannot create a ServiceRequest without at least one Vehicle in ACTIVE status
  - Violation: DomainException is thrown when creating a ServiceRequest if the driver has zero active vehicles
  - Implementation: Validated in ServiceRequest.create(), which queries the Driver's active vehicles first
```
 
**Code example (TypeScript/Java):**
 
```typescript
// TypeScript — Entity
class Driver {
  private constructor(
    private readonly id: DriverId,
    private readonly userId: UserId,
    private readonly createdAt: Date,
  ) {}
 
  static register(userId: UserId): Driver {
    return new Driver(DriverId.new(), userId, new Date());
  }
}
```
 
---
 
### Entity: Vehicle
 
**Context:** User Management
 
**Description:** A car or motorcycle registered by a Driver, associated with service requests.
 
**Attributes:**
 
| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| driverId | UUID | Owning Driver | Yes | Must reference a valid Driver |
| plate | String | License plate | Yes | Unique across the whole system (SRS RF2.1) |
| brand | String | Vehicle brand | Yes | Non-empty |
| model | String | Vehicle model | Yes | Non-empty |
| type | Enum (CAR, MOTORCYCLE) | Vehicle type | Yes | SRS RF2.1 |
| status | Enum (ACTIVE, INACTIVE) | Current status | Yes | Defaults to ACTIVE |
| createdAt | DateTime | Creation date | Yes | Immutable, set on creation |
| updatedAt | DateTime | Last modification | Yes | Updated automatically |
 
**Lifecycle / States:**
 
```
ACTIVE ──(deactivate)──▶ INACTIVE
```
 
| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| ACTIVE | Available to use in a new service request | → INACTIVE |
| INACTIVE | Deactivated, kept for history, cannot be used in a new request | → ACTIVE (can be reactivated) |
 
**Invariants (Business rules that MUST ALWAYS hold):**
 
```
INV-006: Unique plate
  - Rule: No two Vehicle records may share the same plate value
  - Violation: DomainException is thrown on registration if the plate already exists
  - Implementation: Unique constraint at the database level (see 06-data/models.md) plus a domain check on creation
 
INV-007: Deactivation preserves history
  - Rule: Deactivating a Vehicle never deletes it or its past ServiceRequests
  - Violation: A hard delete of a Vehicle with existing ServiceRequests is rejected
  - Implementation: deactivate() only flips status to INACTIVE, never removes the record
```
 
**Code example (TypeScript/Java):**
 
```typescript
// TypeScript — Entity with invariants
class Vehicle {
  private constructor(
    private readonly id: VehicleId,
    private readonly driverId: DriverId,
    private readonly plate: Plate,
    private status: VehicleStatus,
  ) {}
 
  static register(driverId: DriverId, plate: Plate, brand: string, model: string): Vehicle {
    return new Vehicle(VehicleId.new(), driverId, plate, VehicleStatus.ACTIVE);
  }
 
  deactivate(): void {
    if (this.status === VehicleStatus.INACTIVE) {
      throw new DomainException('Vehicle is already inactive');
    }
    this.status = VehicleStatus.INACTIVE;
  }
}
```
 
---
 
### Entity: Mechanic
 
**Context:** User Management
 
**Description:** A verified technician or workshop that can receive and fulfill service requests.
 
**Attributes:**
 
| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| userId | UUID | Reference to the base authenticated account | Yes | Must reference a valid Firebase Auth user |
| workshopName | String | Name of the workshop/technician | Yes | Non-empty |
| verificationStatus | Enum (PENDING_VERIFICATION, VERIFIED, REJECTED) | Admin verification state | Yes | Defaults to PENDING_VERIFICATION |
| status | Enum (AVAILABLE, BUSY, OFFLINE) | Real-time dispatch status | Yes | Defaults to OFFLINE |
| location | GPSLocation | Current live position | No | Required only while status = AVAILABLE |
| createdAt | DateTime | Creation date | Yes | Immutable, set on creation |
| updatedAt | DateTime | Last modification | Yes | Updated automatically |
 
**Lifecycle / States:**
 
```
PENDING_VERIFICATION ──(admin approves)──▶ VERIFIED ──(goes online)──▶ AVAILABLE
                    │                                      │
              (admin rejects)                        (accepts request)
                    ▼                                      ▼
                REJECTED                                 BUSY
                                                            │
                                                     (completes/cancels)
                                                            ▼
                                                       AVAILABLE
```
 
| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| PENDING_VERIFICATION | Just registered, awaiting admin approval | → VERIFIED, → REJECTED |
| VERIFIED | Approved but not accepting requests yet | → AVAILABLE (goes online), → OFFLINE |
| AVAILABLE | Verified and eligible for matching | → BUSY, → OFFLINE |
| BUSY | Currently assigned to a service request | → AVAILABLE |
| OFFLINE | Verified but not currently accepting requests | → AVAILABLE |
| REJECTED | Verification denied | Terminal state |
 
**Invariants (Business rules that MUST ALWAYS hold):**
 
```
INV-001: Verification required before dispatch
  - Rule: A Mechanic must have verificationStatus = VERIFIED before it can appear in matching results
  - Violation: The matching query excludes any Mechanic whose verificationStatus is not VERIFIED
  - Implementation: Filtered directly in the MatchingService query, not left to the client
 
INV-002: Location must be dynamic while available
  - Rule: A Mechanic with status = AVAILABLE must have a location updated within the last 60 seconds
  - Violation: A stale location excludes the Mechanic from matching results
  - Implementation: Checked by MatchingService against location.updatedAt
```
 
**Code example (TypeScript/Java):**
 
```typescript
// TypeScript — Entity with invariants
class Mechanic {
  private constructor(
    private readonly id: MechanicId,
    private verificationStatus: VerificationStatus,
    private status: MechanicStatus,
    private location: GPSLocation | null,
  ) {}
 
  goAvailable(location: GPSLocation): void {
    if (this.verificationStatus !== VerificationStatus.VERIFIED) {
      throw new DomainException('INV-001: Mechanic must be verified before going available');
    }
    this.status = MechanicStatus.AVAILABLE;
    this.location = location;
  }
}
```
 
---
 
### Entity: ServiceRequest
 
**Context:** Matchmaking & Dispatch
 
**Description:** A driver's request for roadside assistance, from creation to completion.
 
**Attributes:**
 
| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| driverId | UUID | Requesting Driver | Yes | Must reference a valid Driver |
| vehicleId | UUID | Vehicle needing assistance | Yes | Must reference an ACTIVE Vehicle owned by driverId (SRS RF2.3) |
| mechanicId | UUID | Assigned Mechanic | No | Set only after matching succeeds |
| pickupLocation | GPSLocation | Where the driver is located | Yes | Set on creation |
| status | Enum (PENDING, ACCEPTED, ON_THE_WAY, IN_PROGRESS, COMPLETED, CANCELLED) | Current status | Yes | Must follow the defined transition order (SRS RF3.5) |
| createdAt | DateTime | Creation date | Yes | Immutable, set on creation |
| updatedAt | DateTime | Last modification | Yes | Updated automatically |
 
**Lifecycle / States:**
 
```
PENDING ──(matched)──▶ ACCEPTED ──(mechanic departs)──▶ ON_THE_WAY ──(arrives)──▶ IN_PROGRESS ──(diagnostic submitted)──▶ COMPLETED
   │            │
(cancel)    (cancel)
   ▼            ▼
CANCELLED   CANCELLED
```
 
| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| PENDING | Just created, matching in progress | → ACCEPTED, → CANCELLED |
| ACCEPTED | A Mechanic has been matched | → ON_THE_WAY, → CANCELLED |
| ON_THE_WAY | Mechanic is traveling to the pickup location | → IN_PROGRESS |
| IN_PROGRESS | Mechanic is on-site working | → COMPLETED |
| COMPLETED | Diagnostic registered, service finished | Terminal state |
| CANCELLED | Cancelled by the driver before completion | Terminal state |
 
**Invariants (Business rules that MUST ALWAYS hold):**
 
```
INV-003: Matching SLA
  - Rule: The transition from PENDING to ACCEPTED must occur within 3 seconds (P95) of creation (NFR-01)
  - Violation: Logged as an SLA breach for monitoring; does not block the transition itself
  - Implementation: Measured by MatchingService, timestamped at ServiceRequested and MechanicMatched events
 
INV-004: Cannot complete without a diagnostic
  - Rule: A ServiceRequest cannot move to COMPLETED without an associated Diagnostic
  - Violation: DomainException is thrown if complete() is called with no diagnostic attached
  - Implementation: Validated in the complete() method before the status transition
```
 
**Code example (TypeScript/Java):**
 
```typescript
// TypeScript — Aggregate Root with invariants
class ServiceRequest {
  private constructor(
    private readonly id: ServiceRequestId,
    private readonly driverId: DriverId,
    private readonly vehicleId: VehicleId,
    private mechanicId: MechanicId | null,
    private status: ServiceRequestStatus,
    private diagnostic: Diagnostic | null,
  ) {}
 
  static create(driverId: DriverId, vehicleId: VehicleId, pickupLocation: GPSLocation): ServiceRequest {
    return new ServiceRequest(ServiceRequestId.new(), driverId, vehicleId, null, ServiceRequestStatus.PENDING, null);
  }
 
  complete(diagnostic: Diagnostic): void {
    if (this.status !== ServiceRequestStatus.IN_PROGRESS) {
      throw new DomainException('Only an IN_PROGRESS request can be completed');
    }
    if (!diagnostic) {
      throw new DomainException('INV-004: Cannot complete without a diagnostic');
    }
    this.diagnostic = diagnostic;
    this.status = ServiceRequestStatus.COMPLETED;
    this.addEvent(new ServiceCompletedEvent(this.id, diagnostic.id));
  }
}
```
 
---
 
### Entity: Diagnostic
 
**Context:** Service Execution
 
**Description:** The record a Mechanic submits when closing a ServiceRequest.
 
**Attributes:**
 
| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| requestId | UUID | The ServiceRequest it closes | Yes | One-to-one — a request has at most one Diagnostic |
| findings | String | Description of the repair/diagnosis performed | Yes | Non-empty |
| finalStatus | Enum (RESOLVED, PARTIALLY_RESOLVED, REFERRED_TO_WORKSHOP) | Outcome of the service | Yes | SRS RF4.4 |
| createdAt | DateTime | Creation date | Yes | Immutable, set on creation |
 
**Lifecycle / States:**
 
No formal state machine — a Diagnostic is created once, at the moment a ServiceRequest
is completed, and is never modified afterward (immutable historical record).
 
**Invariants (Business rules that MUST ALWAYS hold):**
 
```
INV-008: One diagnostic per request
  - Rule: A ServiceRequest can have at most one Diagnostic
  - Violation: DomainException is thrown if a second Diagnostic is submitted for the same requestId
  - Implementation: Enforced by a unique constraint on requestId at the database level
```
 
**Code example (TypeScript/Java):**
 
```typescript
// TypeScript — Entity, immutable after creation
class Diagnostic {
  private constructor(
    private readonly id: DiagnosticId,
    private readonly requestId: ServiceRequestId,
    private readonly findings: string,
    private readonly finalStatus: DiagnosticOutcome,
    private readonly createdAt: Date,
  ) {}
 
  static submit(requestId: ServiceRequestId, findings: string, finalStatus: DiagnosticOutcome): Diagnostic {
    if (!findings.trim()) {
      throw new DomainException('Diagnostic findings cannot be empty');
    }
    return new Diagnostic(DiagnosticId.new(), requestId, findings, finalStatus, new Date());
  }
}
```
 
---
 
## System Value Objects
 
### Value Object: GPSLocation
 
**Description:** An immutable geographic coordinate, used both for a Mechanic's live
position and a ServiceRequest's pickup location.
 
**Attributes:**
 
| Attribute | Type | Description |
|-----------|------|-------------|
| latitude | Number | Latitude in decimal degrees |
| longitude | Number | Longitude in decimal degrees |
| updatedAt | DateTime | When this coordinate was captured |
 
**Validation rules:**
 
```
- latitude must be between -90 and 90
- longitude must be between -180 and 180
- Accuracy relied upon by the system is 15 meters (NFR-02) — the source (device GPS) must meet this
```
 
**Example:**
 
```typescript
// Value Object — Immutable, validated in the constructor
class GPSLocation {
  private readonly latitude: number;
  private readonly longitude: number;
  private readonly updatedAt: Date;
 
  constructor(latitude: number, longitude: number) {
    if (latitude < -90 || latitude > 90) {
      throw new DomainException(`Invalid latitude: ${latitude}`);
    }
    if (longitude < -180 || longitude > 180) {
      throw new DomainException(`Invalid longitude: ${longitude}`);
    }
    this.latitude = latitude;
    this.longitude = longitude;
    this.updatedAt = new Date();
  }
 
  equals(other: GPSLocation): boolean {
    return this.latitude === other.latitude && this.longitude === other.longitude;
  }
}
```
  
---
 
## System Aggregates
 
### Aggregate: Driver
 
**Aggregate Root:** Driver
 
**Internal entities:**
- Vehicle — a Vehicle has no independent lifecycle from its owning Driver; it is always
  accessed and modified through the Driver aggregate
**Value Objects:**
- None specific to this aggregate
**Aggregate invariants:**
 
```
AGGR-INV-001: A Vehicle's driverId must always match the ID of the Driver aggregate it belongs to
AGGR-INV-002: A Driver can have zero or more Vehicles, but a ServiceRequest requires at least one ACTIVE one (INV-005)
```
 
**Why do these objects form an aggregate?**
> A Vehicle only makes sense in the context of the Driver who registered it — it is never
> queried, modified, or deleted independently of its owner. Keeping them as one aggregate
> guarantees that INV-006 (unique plate) and INV-007 (deactivation preserves history) are
> always enforced together, in one transaction.
 
---
 
### Aggregate: ServiceRequest
 
**Aggregate Root:** ServiceRequest
 
**Internal entities:**
- Diagnostic — only ever created and read through its parent ServiceRequest; it never
  exists independently (INV-008)
**Value Objects:**
- GPSLocation (pickupLocation)
**Aggregate invariants:**
 
```
AGGR-INV-003: A ServiceRequest cannot reach status COMPLETED without exactly one associated Diagnostic (INV-004, INV-008)
AGGR-INV-004: mechanicId is null while status = PENDING, and non-null for every other status
```
 
**Why do these objects form an aggregate?**
> The Diagnostic is the closing record of one specific ServiceRequest — it has no meaning
> on its own and must never be created without the request transitioning to COMPLETED in
> the same operation. Treating them as one aggregate guarantees INV-004 cannot be bypassed
> by writing a Diagnostic directly.
 
---
 
### Aggregate: Mechanic
 
**Aggregate Root:** Mechanic
 
**Internal entities:**
- None — Mechanic has no child entities
**Value Objects:**
- GPSLocation (location)
**Aggregate invariants:**
 
```
AGGR-INV-005: location can only be non-null while status is AVAILABLE or BUSY (INV-002)
```
 
**Why do these objects form an aggregate?**
> Mechanic is a simple aggregate with a single root and no internal entities — its own
> verification and dispatch status are enough to enforce INV-001 and INV-002 without
> needing to coordinate with any other object.
 
---
 
## Summary table of tactical building blocks
 
| Name | Type | Bounded Context | Aggregate Root? |
|------|------|----------------|----------------|
| Driver | Entity | User Management | Yes |
| Vehicle | Entity | User Management | No (inside Driver) |
| Mechanic | Entity | User Management | Yes |
| ServiceRequest | Entity | Matchmaking & Dispatch | Yes |
| Diagnostic | Entity | Service Execution | No (inside ServiceRequest) |
| GPSLocation | Value Object | Shared | N/A |
| MatchingService | Domain Service | Matchmaking & Dispatch | N/A |
 
---
 
## Domain Services
 
A **Domain Service** is business logic that does not naturally belong to any entity.
Use it when:
- The operation involves multiple entities or aggregates
- It would be unnatural for the operation to belong to a single entity
- The logic does not need its own state
```typescript
// Domain Service — Stateless, orchestrates logic between the ServiceRequest and Mechanic aggregates
class MatchingService {
  findNearestAvailableMechanic(request: ServiceRequest, mechanics: Mechanic[]): Mechanic | null {
    const eligible = mechanics.filter(m =>
      m.verificationStatus === VerificationStatus.VERIFIED &&
      m.status === MechanicStatus.AVAILABLE
    );
    if (eligible.length === 0) return null;
 
    return eligible.reduce((nearest, current) =>
      distance(current.location, request.pickupLocation) < distance(nearest.location, request.pickupLocation)
        ? current
        : nearest
    );
  }
}
```
 
---
 
## Correlation with code
 
| Domain artifact | Package / folder in code | File |
|----------------|--------------------------|------|
| Aggregate Root `Driver` | `src/domain/driver/` | `Driver.ts` |
| Entity `Vehicle` | `src/domain/driver/` | `Vehicle.ts` |
| Aggregate Root `Mechanic` | `src/domain/mechanic/` | `Mechanic.ts` |
| Aggregate Root `ServiceRequest` | `src/domain/service-request/` | `ServiceRequest.ts` |
| Entity `Diagnostic` | `src/domain/service-request/` | `Diagnostic.ts` |
| Value Object `GPSLocation` | `src/domain/shared/value-objects/` | `GPSLocation.ts` |
| Domain Service `MatchingService` | `src/domain/service-request/services/` | `MatchingService.ts` |
 
> See hexagonal structure in `05-architecture/hexagonal-architecture.md`