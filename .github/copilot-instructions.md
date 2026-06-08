# GitHub Copilot Instructions
## Hotel Reservation System API

This file guides Copilot's suggestions to match the conventions and architecture of this project.

---

## Project Context

This is a **.NET 8 ASP.NET Core Web API** for a hotel reservation system.  
It has **no UI** — only REST API endpoints consumed by external clients.  
The database is **SQL Server** accessed via **Entity Framework Core 8**.

---

## Stack & Versions

- **Runtime:** .NET 8
- **Language:** C# 12
- **ORM:** Entity Framework Core 8 (Code-First, Fluent API configuration in `AppDbContext`)
- **Database:** SQL Server
- **API Docs:** Swashbuckle / Swagger (auto-generated, dev only)
- **DI Lifetime:** Services are `Scoped`

---

## Architecture Rules

### Layer Responsibilities
- **Controllers** — Handle HTTP only: routing, request binding, status codes, calling services.  
  Never put business logic in controllers.
- **Services** — All business logic lives here. Validate inputs, enforce rules, calculate values.  
  Services talk to `AppDbContext` directly (no repository pattern).
- **AppDbContext** — EF Core DbContext. Entity configuration via Fluent API in `OnModelCreating`.  
  Never use Data Annotations on models.
- **Models** — Plain entity classes. No business logic. No attributes.
- **DTOs** — Use C# `record` types for all request/response objects. Never expose raw models to the API.

### Dependency Direction
```
Controllers → Services (via interface) → AppDbContext → SQL Server
```
Controllers must depend on **interfaces** (`IRoomService`, `IReservationService`), never concrete classes.

---

## Naming Conventions

| Artifact | Convention | Example |
|----------|-----------|---------|
| Controllers | `{Entity}Controller` | `RoomsController` |
| Service interfaces | `I{Entity}Service` | `IRoomService` |
| Service classes | `{Entity}Service` | `RoomService` |
| Request DTOs | `Create{Entity}Request` | `CreateRoomRequest` |
| Response DTOs | `{Entity}Response` | `RoomResponse` |
| DbContext | `AppDbContext` | — |
| Async methods | Suffix `Async` | `GetByIdAsync` |

---

## Code Style

- Use **primary constructors** for constructor injection (C# 12):
  ```csharp
  public class RoomService(AppDbContext db) : IRoomService { }
  ```
- Use `null!` for navigation properties that EF Core will always populate.
- Use `async/await` on all database calls — never `.Result` or `.Wait()`.
- Use `DateOnly` for reservation dates, not `DateTime`.
- Use `decimal` (not `double` or `float`) for all monetary values.
- Prefer `record` for DTOs:
  ```csharp
  public record CreateRoomRequest(string RoomNumber, string Type, decimal PricePerNight, int Floor);
  ```
- Prefer `static` helper methods for entity → DTO mapping (`ToResponse`).

---

## Business Rules Copilot Must Respect

1. **Double-booking prevention:** Before inserting a reservation, always query for overlapping confirmed reservations:
   ```csharp
   r.Status == "Confirmed" &&
   r.CheckInDate  < req.CheckOutDate &&
   r.CheckOutDate > req.CheckInDate
   ```

2. **Total amount calculation:** Always compute at booking time and persist it:
   ```csharp
   int nights = req.CheckOutDate.DayNumber - req.CheckInDate.DayNumber;
   decimal total = nights * room.PricePerNight;
   ```

3. **Soft delete for rooms:** Never hard-delete a room. Set `IsActive = false`.

4. **Date validation:** `CheckOutDate` must be strictly after `CheckInDate`. Throw `ArgumentException` if not.

5. **Status values:** Use only `"Confirmed"` and `"Cancelled"` as string literals for `Reservation.Status`.

---

## Error Handling Pattern

Throw typed exceptions in services; catch them in controllers:

| Exception | HTTP Status | When to throw |
|-----------|-------------|---------------|
| `ArgumentException` | 400 | Invalid input (bad dates, negative price) |
| `KeyNotFoundException` | 404 | Entity not found by ID |
| `InvalidOperationException` | 409 | Business rule violation (double-book, already cancelled, inactive room) |

Controller pattern:
```csharp
try { /* call service */ }
catch (KeyNotFoundException ex)      { return NotFound(new { error = ex.Message }); }
catch (InvalidOperationException ex) { return Conflict(new { error = ex.Message }); }
catch (ArgumentException ex)         { return BadRequest(new { error = ex.Message }); }
```

---

## EF Core Conventions

- Configure all entities in `OnModelCreating` using Fluent API only.
- Always set `HasMaxLength` on string properties.
- Always set `HasColumnType("decimal(10,2)")` on decimal properties.
- Use `HasDefaultValueSql("GETUTCDATE()")` for `CreatedAt` fields.
- Use `HasDefaultValue("Confirmed")` for `Reservation.Status`.
- Always define foreign key constraints explicitly via `HasOne/WithMany/HasForeignKey`.

---

## What Copilot Should NOT Do

- Do not add `[Required]`, `[MaxLength]`, or other Data Annotations to model classes.
- Do not add a repository pattern or unit-of-work layer — EF Core is the data layer.
- Do not add authentication/authorization middleware (planned for v2.0).
- Do not use `DateTime` for reservation check-in/check-out dates.
- Do not use `float` or `double` for prices or monetary values.
- Do not put business logic in controllers.
- Do not return raw EF Core entity objects from controllers — always map to a DTO.
- Do not call `.Result` or `.Wait()` on async methods.

---

## Adding New Features — Checklist

When Copilot helps add a new endpoint, follow this checklist:

- [ ] Add entity to `Models/` (if new table needed)
- [ ] Add DTO records to `DTOs/`
- [ ] Configure entity in `AppDbContext.OnModelCreating`
- [ ] Define interface in `Services/I{Entity}Service.cs`
- [ ] Implement service in `Services/{Entity}Service.cs`
- [ ] Register service as Scoped in `Program.cs`
- [ ] Add controller in `Controllers/{Entity}Controller.cs`
- [ ] Handle all exception types in controller catch blocks
- [ ] Return correct HTTP status codes (201 for create, 204 for delete, 404 for not found)