---
applyTo: "**/*.cs"
---

# Developer Agent — Hotel Reservation API

## Code Style
- Primary constructors for DI: `public class RoomService(AppDbContext db)`
- `record` types for all DTOs (immutable, value-based equality)
- `async/await` on every DB call — never `.Result` or `.Wait()`
- `null!` for EF navigation properties
- `static` private method `ToResponse(entity)` for entity→DTO mapping

## Naming
| Artifact | Pattern | Example |
|----------|---------|---------|
| Controller | `{Entity}Controller` | `RoomsController` |
| Interface | `I{Entity}Service` | `IRoomService` |
| Service | `{Entity}Service` | `RoomService` |
| Request DTO | `Create{Entity}Request` | `CreateReservationRequest` |
| Response DTO | `{Entity}Response` | `ReservationResponse` |
| Async method | suffix `Async` | `GetByIdAsync` |

## Error Handling in Controllers
```csharp
catch (KeyNotFoundException ex)      { return NotFound(new { error = ex.Message }); }
catch (InvalidOperationException ex) { return Conflict(new { error = ex.Message }); }
catch (ArgumentException ex)         { return BadRequest(new { error = ex.Message }); }
```

## New Feature Checklist
- [ ] Model in `Models/`
- [ ] DTOs (request + response) in `DTOs/`
- [ ] Interface in `Services/I{Entity}Service.cs`
- [ ] Implementation in `Services/{Entity}Service.cs`
- [ ] Register as Scoped in `Program.cs`
- [ ] Controller in `Controllers/{Entity}Controller.cs`
- [ ] Correct HTTP status codes: 201 create · 204 delete · 404 not found · 409 conflict

## Business Logic Reminders
- Double-booking check: `CheckInDate < req.CheckOut && CheckOutDate > req.CheckIn`
- Night calculation: `CheckOutDate.DayNumber - CheckInDate.DayNumber`
- Validate `CheckOut > CheckIn` → throw `ArgumentException`
- Inactive room → throw `InvalidOperationException`