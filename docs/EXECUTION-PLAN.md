# Execution Plan
## Hotel Reservation System API

**Version:** 1.0  
**Date:** 2026-06-08  
**Stack:** .NET 10 · EF Core 8 · SQL Server  
**Team:** 1 Developer  

---

## Overview

```
Phase 1 → Repo & Tooling Setup        (Day 1)
Phase 2 → Database & Domain Layer     (Day 1–2)
Phase 3 → Business Logic (Services)   (Day 2–3)
Phase 4 → API Layer (Controllers)     (Day 3)
Phase 5 → Testing                     (Day 4)
Phase 6 → Documentation & Handoff     (Day 5)
```

Total estimated: **5 working days**

---

## Phase 1 — Repo & Tooling Setup
**Duration:** Half day  
**Goal:** Project skeleton is running locally with Swagger accessible.

### Tasks

| # | Task | Command / Action | Done |
|---|------|-----------------|------|
| 1.1 | Create test project | `dotnet new xunit -n HotelReservation.Tests` | ☐ |
| 1.2 | Add projects to solution | `dotnet sln add **/*.csproj` | ☐ |
| 1.3 | Install NuGet packages (API) | See packages below | ☐ |
| 1.4 | Install NuGet packages (Tests) | See packages below | ☐ |
| 1.5 | Add `.vscode/settings.json` | Enable Copilot instruction files | ☐ |
| 1.6 | Verify project runs | `dotnet run` → Swagger at `/swagger` | ☐ |

### NuGet Packages

**HotelReservation.API:**
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Swashbuckle.AspNetCore
```

**HotelReservation.Tests:**
```bash
dotnet add package Microsoft.EntityFrameworkCore.InMemory
dotnet add package Moq
dotnet add package FluentAssertions
dotnet add reference ../HotelReservation.API/HotelReservation.API.csproj
```

### Deliverable
- Solution compiles and runs
- Swagger UI accessible at `https://localhost:{port}/swagger`
- Git repo initialized with first commit

---

## Phase 2 — Database & Domain Layer
**Duration:** 1 day  
**Goal:** Database created, EF Core connected, migrations applied, seed data loaded.

### Tasks

| # | Task | File | Done |
|---|------|------|------|
| 2.1 | Create `Room` model | `Models/Room.cs` | ☐ |
| 2.2 | Create `Customer` model | `Models/Customer.cs` | ☐ |
| 2.3 | Create `Reservation` model | `Models/Reservation.cs` | ☐ |
| 2.4 | Create `AppDbContext` with Fluent API config | `Data/AppDbContext.cs` | ☐ |
| 2.5 | Register DbContext in DI | `Program.cs` | ☐ |
| 2.6 | Add connection string | `appsettings.json` | ☐ |
| 2.7 | Add connection string (dev override) | `appsettings.Development.json` | ☐ |
| 2.8 | Create initial EF migration | `dotnet ef migrations add 20260608_InitialSchema` | ☐ |
| 2.9 | Review generated migration file | `Migrations/` | ☐ |
| 2.10 | Apply migration to SQL Server | `dotnet ef database update` | ☐ |
| 2.11 | Run SQL seed script | Execute `schema.sql` seed section | ☐ |
| 2.12 | Verify tables in SQL Server | SSMS or Azure Data Studio | ☐ |

### Entity Checklist

**Room**
- [ ] `RoomId` int PK auto-increment
- [ ] `RoomNumber` nvarchar(10) UNIQUE
- [ ] `Type` nvarchar(50)
- [ ] `PricePerNight` decimal(10,2)
- [ ] `Floor` int
- [ ] `IsActive` bit default 1
- [ ] `CreatedAt` datetime2 default GETUTCDATE()

**Customer**
- [ ] `CustomerId` int PK
- [ ] `FullName` nvarchar(100)
- [ ] `Email` nvarchar(150) UNIQUE
- [ ] `Phone` nvarchar(20) nullable
- [ ] `CreatedAt` datetime2 default GETUTCDATE()

**Reservation**
- [ ] `ReservationId` int PK
- [ ] `RoomId` int FK → Rooms
- [ ] `CustomerId` int FK → Customers
- [ ] `CheckInDate` date
- [ ] `CheckOutDate` date (> CheckInDate)
- [ ] `TotalAmount` decimal(10,2)
- [ ] `Status` nvarchar(20) default 'Confirmed'
- [ ] `CreatedAt` datetime2 default GETUTCDATE()

### Deliverable
- All 3 tables created in SQL Server
- Indexes and constraints applied
- Seed data (5 rooms, 2 customers) verified in DB

---

## Phase 3 — Business Logic (Services)
**Duration:** 1 day  
**Goal:** All service interfaces and implementations complete with business rules enforced.

### Tasks

| # | Task | File | Done |
|---|------|------|------|
| 3.1 | Create `RoomDtos` (request + response) | `DTOs/RoomDtos.cs` | ☐ |
| 3.2 | Create `ReservationDtos` (request + response) | `DTOs/ReservationDtos.cs` | ☐ |
| 3.3 | Create `IRoomService` interface | `Services/IRoomService.cs` | ☐ |
| 3.4 | Implement `RoomService` | `Services/RoomService.cs` | ☐ |
| 3.5 | Create `IReservationService` interface | `Services/IReservationService.cs` | ☐ |
| 3.6 | Implement `ReservationService` | `Services/ReservationService.cs` | ☐ |
| 3.7 | Register services as Scoped in DI | `Program.cs` | ☐ |

### Business Rules Verification

**RoomService**
- [ ] `GetAllActiveAsync` filters `IsActive == true` only
- [ ] `CreateAsync` inserts room and returns DTO
- [ ] `DeleteAsync` sets `IsActive = false` (soft delete, not hard delete)
- [ ] `GetByIdAsync` returns null if not found (controller handles 404)

**ReservationService**
- [ ] `CheckOutDate > CheckInDate` validated → `ArgumentException`
- [ ] Room existence checked → `KeyNotFoundException`
- [ ] Room `IsActive` checked → `InvalidOperationException`
- [ ] Customer existence checked → `KeyNotFoundException`
- [ ] Overlap query: `CheckInDate < req.CheckOut && CheckOutDate > req.CheckIn` → `InvalidOperationException`
- [ ] `TotalAmount = (CheckOut.DayNumber - CheckIn.DayNumber) × PricePerNight`
- [ ] `TotalAmount` persisted at booking time (not computed on read)
- [ ] `CancelAsync` checks status != "Cancelled" → `InvalidOperationException`
- [ ] Response includes: nights count, price per night, total amount, status

### Deliverable
- All services compile
- Business rules manually traced through code
- DI registrations in place

---

## Phase 4 — API Layer (Controllers)
**Duration:** Half day  
**Goal:** All endpoints live and tested via Swagger.

### Tasks

| # | Task | File | Done |
|---|------|------|------|
| 4.1 | Create `RoomsController` | `Controllers/RoomsController.cs` | ☐ |
| 4.2 | Create `ReservationsController` | `Controllers/ReservationsController.cs` | ☐ |
| 4.3 | Verify Swagger shows all endpoints | `https://localhost:{port}/swagger` | ☐ |
| 4.4 | Manual smoke test all endpoints | Swagger UI or Postman | ☐ |

### Endpoint Acceptance Criteria

| Endpoint | Input | Expected Response |
|----------|-------|------------------|
| `GET /api/rooms` | — | 200 + array of active rooms |
| `GET /api/rooms/{id}` | valid id | 200 + room detail |
| `GET /api/rooms/{id}` | invalid id | 404 |
| `POST /api/rooms` | valid body | 201 + created room |
| `DELETE /api/rooms/{id}` | valid id | 204 |
| `DELETE /api/rooms/{id}` | invalid id | 404 |
| `POST /api/reservations` | valid body | 201 + reservation + TotalAmount |
| `POST /api/reservations` | overlapping dates | 409 |
| `POST /api/reservations` | invalid room | 404 |
| `POST /api/reservations` | checkout ≤ checkin | 400 |
| `GET /api/reservations/{id}` | valid id | 200 + full detail |
| `GET /api/reservations/{id}` | invalid id | 404 |
| `DELETE /api/reservations/{id}` | confirmed reservation | 204 |
| `DELETE /api/reservations/{id}` | already cancelled | 409 |

### Deliverable
- All 7 endpoints reachable via Swagger
- All acceptance criteria verified manually

---

## Phase 5 — Testing
**Duration:** 1 day  
**Goal:** Automated test suite covering all service and controller logic.

### Tasks

| # | Task | File | Done |
|---|------|------|------|
| 5.1 | Setup test project structure | `HotelReservation.Tests/` | ☐ |
| 5.2 | Write `RoomServiceTests` | `Tests/RoomServiceTests.cs` | ☐ |
| 5.3 | Write `ReservationServiceTests` | `Tests/ReservationServiceTests.cs` | ☐ |
| 5.4 | Write `RoomsControllerTests` | `Tests/RoomsControllerTests.cs` | ☐ |
| 5.5 | Write `ReservationsControllerTests` | `Tests/ReservationsControllerTests.cs` | ☐ |
| 5.6 | Write integration test: full booking flow | `Tests/Integration/BookingFlowTests.cs` | ☐ |
| 5.7 | Run full test suite | `dotnet test` | ☐ |
| 5.8 | Verify 0 failures | CI output | ☐ |

### Minimum Test Coverage

| Class | Min Tests |
|-------|-----------|
| `RoomService` | 5 |
| `ReservationService` | 10 |
| `RoomsController` | 4 |
| `ReservationsController` | 6 |
| Integration (booking flow) | 3 |
| **Total** | **28+** |

### Deliverable
- `dotnet test` passes with 0 failures
- All business rules covered by at least one test
- All exception paths covered by at least one test

---

## Phase 6 — Documentation & Handoff
**Duration:** Half day  
**Goal:** Repo is clean, documented, and ready for the next developer or v2.0.

### Tasks

| # | Task | Done |
|---|------|------|
| 6.1 | Write `README.md` with setup instructions | ☐ |
| 6.2 | Verify `docs/PRD.md` is up to date | ☐ |
| 6.3 | Verify `docs/ARD.md` is up to date | ☐ |
| 6.4 | Verify all `.github/instructions/` files are committed | ☐ |
| 6.5 | Verify all `.github/prompts/` files are committed | ☐ |
| 6.6 | Add `appsettings.Example.json` (no real credentials) | ☐ |
| 6.7 | Confirm `appsettings.json` is in `.gitignore` | ☐ |
| 6.8 | Final `dotnet build` — 0 warnings, 0 errors | ☐ |
| 6.9 | Final `dotnet test` — 0 failures | ☐ |
| 6.10 | Tag release: `git tag v1.0.0` | ☐ |

### README.md Must Include
- [ ] Project description
- [ ] Prerequisites (SDK version, SQL Server)
- [ ] Setup steps (clone → connection string → migrate → run)
- [ ] Endpoint list with example request/response
- [ ] How to run tests
- [ ] Link to PRD and ARD

---

## Final Repository Structure

```
HotelReservation/
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   │   ├── developer.instructions.md
│   │   ├── qa.instructions.md
│   │   ├── database.instructions.md
│   │   └── architecture.instructions.md
│   └── prompts/
│       ├── new-endpoint.prompt.md
│       ├── write-tests.prompt.md
│       ├── code-review.prompt.md
│       ├── add-migration.prompt.md
│       └── debug.prompt.md
├── .vscode/
│   └── settings.json
├── docs/
│   ├── PRD.md
│   ├── ARD.md
│   └── EXECUTION-PLAN.md
├── HotelReservation.API/
│   ├── Controllers/
│   ├── Services/
│   ├── Data/
│   ├── Models/
│   ├── DTOs/
│   ├── Migrations/
│   ├── Program.cs
│   ├── appsettings.json          ← gitignored
│   └── appsettings.Example.json  ← committed (no secrets)
├── HotelReservation.Tests/
│   ├── Integration/
│   └── *.Tests.cs
├── HotelReservation.sln
└── README.md
```

---

## Risk & Mitigation

| Risk | Likelihood | Mitigation |
|------|-----------|-----------|
| SQL Server connection issues locally | Medium | Use `TrustServerCertificate=True` in dev connection string |
| EF migration conflict | Low | Always run `dotnet ef migrations list` before adding new migration |
| Double-booking edge case bug | Medium | Covered by boundary date tests in Phase 5 |
| `DateOnly` serialization issue in older .NET | Low | Ensure .NET 8 — `DateOnly` is natively supported |
| Secrets committed to Git | Low | `appsettings.json` in `.gitignore` from day 1 |