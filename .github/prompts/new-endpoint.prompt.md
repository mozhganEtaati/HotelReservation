---
mode: agent
tools: [codebase, editFiles, runCommands]
description: Scaffold a complete new API endpoint following project conventions
---

# Skill: Add New Endpoint

You are a senior .NET 8 developer working on the Hotel Reservation System API.

## Your Task
Scaffold a complete new endpoint for the entity: **${input:entityName}**  
Endpoint purpose: **${input:endpointDescription}**

## Steps to Follow (in order)

### 1. Check existing code
- Read `Models/` to understand existing entity patterns
- Read `DTOs/` to understand DTO record conventions
- Read `Services/` to understand service interface + implementation patterns
- Read `Controllers/` to understand controller patterns

### 2. Create Model (if new entity)
File: `Models/${input:entityName}.cs`
- Plain class, no Data Annotations
- Include `Id`, `CreatedAt`, navigation properties if needed

### 3. Create DTOs
File: `DTOs/${input:entityName}Dtos.cs`
- `Create${input:entityName}Request` record
- `${input:entityName}Response` record
- Include all fields the API consumer needs

### 4. Update AppDbContext
File: `Data/AppDbContext.cs`
- Add `DbSet<${input:entityName}>` property
- Add Fluent API config in `OnModelCreating`:
  - `HasMaxLength` on all strings
  - `HasColumnType("decimal(10,2)")` on all decimals
  - `HasDefaultValueSql("GETUTCDATE()")` on `CreatedAt`
  - Define FK relationships explicitly

### 5. Create Service Interface
File: `Services/I${input:entityName}Service.cs`
- Async methods only
- Return DTOs, not entities

### 6. Create Service Implementation
File: `Services/${input:entityName}Service.cs`
- Primary constructor injection of `AppDbContext`
- Business logic and validation here
- Throw typed exceptions:
  - `KeyNotFoundException` → not found
  - `InvalidOperationException` → business rule violation
  - `ArgumentException` → invalid input
- Private static `ToResponse()` for mapping

### 7. Register in DI
File: `Program.cs`
- `builder.Services.AddScoped<I${input:entityName}Service, ${input:entityName}Service>();`

### 8. Create Controller
File: `Controllers/${input:entityName}Controller.cs`
- `[ApiController]` + `[Route("api/[controller]")]`
- Primary constructor injection of interface
- try/catch pattern:
  ```csharp
  catch (KeyNotFoundException ex)      { return NotFound(new { error = ex.Message }); }
  catch (InvalidOperationException ex) { return Conflict(new { error = ex.Message }); }
  catch (ArgumentException ex)         { return BadRequest(new { error = ex.Message }); }
  ```
- HTTP status codes: 201 create · 204 delete · 200 get · 404 not found

### 9. Create EF Migration
Run:
```bash
dotnet ef migrations add Add${input:entityName} --project HotelReservation.API
```

## Definition of Done
- [ ] All 7 files created/updated
- [ ] No business logic in controller
- [ ] No raw entities returned from API
- [ ] Migration created
- [ ] Compiles without errors