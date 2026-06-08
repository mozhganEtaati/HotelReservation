# Architecture Reference Document (ARD)
## Hotel Reservation System API

**Version:** 1.0  
**Date:** 2026-06-08  
**Stack:** .NET 8 · Entity Framework Core · SQL Server  

---

## 1. Architecture Overview

The system is a single-tier REST API following a **Layered Architecture** pattern:

```
Client (HTTP)
     │
     ▼
┌─────────────────────┐
│    Controllers       │  ← HTTP request/response, routing, status codes
├─────────────────────┤
│    Services          │  ← Business logic, validation, calculations
├─────────────────────┤
│    AppDbContext      │  ← EF Core ORM, query execution
├─────────────────────┤
│    SQL Server DB     │  ← Persistent storage
└─────────────────────┘
```

No external messaging, caching, or background services in v1.0.

---

## 2. Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Runtime | .NET | 8.0 |
| Web Framework | ASP.NET Core Web API | 8.0 |
| ORM | Entity Framework Core | 8.x |
| Database | SQL Server | 2019+ |
| API Docs | Swashbuckle (Swagger/OpenAPI) | 6.x |
| Language | C# | 12 |

---

## 3. Project Structure

```
HotelReservation.API/
├── Controllers/
│   ├── RoomsController.cs          # GET, POST, DELETE /api/rooms
│   └── ReservationsController.cs   # GET, POST, DELETE /api/reservations
├── Services/
│   ├── IRoomService.cs
│   ├── RoomService.cs
│   ├── IReservationService.cs
│   └── ReservationService.cs
├── Data/
│   └── AppDbContext.cs             # EF Core DbContext + Fluent API config
├── Models/
│   ├── Room.cs
│   ├── Customer.cs
│   └── Reservation.cs
├── DTOs/
│   ├── RoomDtos.cs                 # CreateRoomRequest, RoomResponse
│   └── ReservationDtos.cs          # CreateReservationRequest, ReservationResponse
├── Program.cs                      # DI registration, middleware pipeline
└── appsettings.json                # Connection string, logging config
```

---

## 4. Data Architecture

### 4.1 Entity Relationship Diagram

```
┌──────────┐         ┌──────────────┐         ┌───────────┐
│  Rooms   │ 1     * │ Reservations │ *     1  │ Customers │
│──────────│─────────│──────────────│──────────│───────────│
│ RoomId   │         │ReservationId │          │CustomerId │
│RoomNumber│         │ RoomId (FK)  │          │ FullName  │
│ Type     │         │CustomerId(FK)│          │ Email     │
│PricePerNt│         │ CheckInDate  │          │ Phone     │
│ Floor    │         │CheckOutDate  │          └───────────┘
│ IsActive │         │ TotalAmount  │
└──────────┘         │ Status       │
                     └──────────────┘
```

### 4.2 Key Constraints
- `Rooms.RoomNumber` — UNIQUE
- `Customers.Email` — UNIQUE
- `Reservations.CheckOutDate > CheckInDate` — CHECK constraint
- Foreign keys with cascading read, no cascade delete (soft delete pattern)

### 4.3 Soft Delete Pattern
Rooms use `IsActive` flag rather than hard deletes. This ensures:
- Reservation history always resolves to a valid room record.
- No orphan foreign key issues.
- Audit trail is preserved.

---

## 5. Business Logic Layer

### 5.1 Reservation Creation Flow

```
POST /api/reservations
        │
        ├─ Validate CheckOut > CheckIn           → 400 if invalid
        ├─ Fetch Room by RoomId                  → 404 if not found
        ├─ Check Room.IsActive == true           → 409 if inactive
        ├─ Fetch Customer by CustomerId          → 404 if not found
        ├─ Query overlapping Confirmed bookings  → 409 if conflict
        ├─ Calculate TotalAmount
        │     = (CheckOut - CheckIn).Days × PricePerNight
        ├─ Insert Reservation (Status=Confirmed)
        └─ Return 201 + ReservationResponse
```

### 5.2 Double-Booking Guard (SQL Logic)
```sql
-- Detects any overlap with existing confirmed reservation
WHERE RoomId = @RoomId
  AND Status = 'Confirmed'
  AND CheckInDate  < @CheckOutDate
  AND CheckOutDate > @CheckInDate
```
This covers all overlap cases: partial overlap, full containment, and exact match.

### 5.3 Cancellation Flow
```
DELETE /api/reservations/{id}
        │
        ├─ Fetch Reservation by Id   → 404 if not found
        ├─ Check Status != Cancelled → 409 if already cancelled
        ├─ Set Status = 'Cancelled'
        └─ Return 204 No Content
```

---

## 6. Dependency Injection

All services are registered as **Scoped** (per HTTP request):

```csharp
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseSqlServer(connectionString));

builder.Services.AddScoped<IRoomService, RoomService>();
builder.Services.AddScoped<IReservationService, ReservationService>();
```

Controllers depend on interfaces, not concrete classes — enabling easy unit testing and future swapping of implementations.

---

## 7. Error Response Contract

All error responses follow this JSON shape:

```json
{
  "error": "Human-readable error message"
}
```

| Exception Type | HTTP Status |
|----------------|-------------|
| `ArgumentException` | 400 Bad Request |
| `KeyNotFoundException` | 404 Not Found |
| `InvalidOperationException` | 409 Conflict |
| Unhandled | 500 Internal Server Error |

---

## 8. Configuration

`appsettings.json` keys:

| Key | Purpose |
|-----|---------|
| `ConnectionStrings:DefaultConnection` | SQL Server connection string |
| `Logging:LogLevel:Default` | Root log level |
| `Logging:LogLevel:Microsoft.AspNetCore` | Framework log level |

---

## 9. Architectural Decisions

### ADR-01: Soft Delete over Hard Delete for Rooms
**Decision:** Set `IsActive = false` instead of `DELETE` row.  
**Reason:** Reservations reference rooms by FK. Hard-deleting a room with existing reservations would break history or require cascading deletes, losing audit data.

### ADR-02: DateOnly for Check-in/Check-out
**Decision:** Use `DateOnly` (not `DateTime`) for reservation dates.  
**Reason:** A reservation is date-scoped, not time-scoped. Using `DateTime` introduces timezone ambiguity with no benefit.

### ADR-03: TotalAmount Stored at Booking Time
**Decision:** Calculate and persist `TotalAmount` when the reservation is created, not computed on read.  
**Reason:** If `PricePerNight` changes later, historical reservation amounts must remain accurate.

### ADR-04: Services Depend on DbContext Directly (No Repository Pattern)
**Decision:** Services call `AppDbContext` directly via EF Core, no additional repository abstraction.  
**Reason:** EF Core's `DbSet<T>` already acts as a repository. Adding another layer in v1.0 is over-engineering. Can be introduced in v2.0 if needed.

---

## 10. Future Architecture Considerations (v2.0+)

| Concern | Recommendation |
|---------|---------------|
| Authentication | Add JWT Bearer middleware + `[Authorize]` on controllers |
| Caching | Add IMemoryCache or Redis for room listings |
| Scalability | Extract to microservices if hotel count grows |
| Migrations | Use EF Core Migrations (`dotnet ef migrations add`) |
| Testing | Add xUnit + Moq unit tests for service layer; use EF InMemory for integration tests |