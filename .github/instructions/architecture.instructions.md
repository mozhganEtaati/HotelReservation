---
applyTo: "**/Program.cs, **/Controllers/**, **/Services/**"
---

# Architecture Agent — Hotel Reservation API

## Layer Boundaries (strict)
- **Controllers** → call service interfaces only, no DbContext injection
- **Services** → call DbContext only, no HttpContext or controller concerns
- **DbContext** → EF configuration only, no business logic

## Dependency Injection
All services registered as Scoped:
```csharp
builder.Services.AddScoped<IRoomService, RoomService>();
builder.Services.AddScoped<IReservationService, ReservationService>();
```
Never register as Singleton if the class uses DbContext (DbContext is Scoped).

## Adding a New Domain Entity
Follow this order strictly:
1. `Models/{Entity}.cs` — plain class, no logic
2. `DTOs/{Entity}Dtos.cs` — request + response records
3. `Data/AppDbContext.cs` — add `DbSet<Entity>` + Fluent config
4. `Services/I{Entity}Service.cs` — interface with async methods
5. `Services/{Entity}Service.cs` — implementation
6. `Program.cs` — register scoped service
7. `Controllers/{Entity}Controller.cs` — HTTP layer only

## What Belongs Where

| Code | Belongs In |
|------|-----------|
| Date validation | Service |
| Overlap query | Service |
| TotalAmount calc | Service |
| HTTP status code | Controller |
| try/catch | Controller |
| EF queries | Service (via DbContext) |
| Entity config | AppDbContext.OnModelCreating |
| DTO shape | DTOs folder |

## Response Mapping
Always use a private static method — never inline mapping:
```csharp
private static ReservationResponse ToResponse(Reservation r, Room room, Customer c)
    => new(r.ReservationId, ...);
```

## What Never to Add (v1.0 scope)
- No authentication middleware
- No repository/unit-of-work abstraction
- No CQRS / MediatR
- No background jobs or hosted services
- No caching layer