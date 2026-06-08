
# Copilot Global Instructions — Hotel Reservation API

## Stack
- .NET 10 · C# 12 · ASP.NET Core Web API
- Entity Framework Core 8 · SQL Server
- No UI — REST API only

## Layer Rules (always enforced)
- Controllers → Services (interface) → AppDbContext → SQL Server
- No business logic in controllers
- No repository pattern — EF Core is the data layer
- Never expose raw EF entities — always map to a DTO record

## Hard Rules
- `decimal` for all money — never float/double
- `DateOnly` for reservation dates — never DateTime
- Soft delete only for Rooms (`IsActive = false`)
- `TotalAmount` stored at booking time, never computed on read
- Status values: `"Confirmed"` | `"Cancelled"` only

## References
- Architecture details → docs/ARD.md
- Product requirements → docs/PRD.md