---
applyTo: "**/Migrations/**, **/Data/**, **/*DbContext*"
---

# Database Agent — Hotel Reservation API

## EF Core Configuration Rules
- Fluent API only in `OnModelCreating` — NO Data Annotations on models
- Every string → `HasMaxLength`
- Every decimal → `HasColumnType("decimal(10,2)")`
- Every `CreatedAt` → `HasDefaultValueSql("GETUTCDATE()")`
- `Reservation.Status` → `HasDefaultValue("Confirmed")`
- All FK relationships defined explicitly via `HasOne/WithMany/HasForeignKey`

## Migration Workflow
```bash
# Add a migration
dotnet ef migrations add <MigrationName> --project HotelReservation.API

# Apply to database
dotnet ef database update --project HotelReservation.API

# Rollback one migration
dotnet ef database update <PreviousMigrationName>

# Remove last unapplied migration
dotnet ef migrations remove
```

## Migration Naming Convention
```
YYYYMMDD_PascalCaseDescription
```
Examples:
- `20260608_InitialSchema`
- `20260615_AddRoomDescription`
- `20260620_AddReservationNotes`

## Index Guidelines
Always add indexes on:
- Foreign keys: `RoomId`, `CustomerId` in Reservations
- Filter columns used in queries: `Status`, `IsActive`
- Unique constraints: `RoomNumber`, `Email`

```csharp
// In OnModelCreating:
mb.Entity<Reservation>()
  .HasIndex(r => r.RoomId);
mb.Entity<Reservation>()
  .HasIndex(r => r.Status);
```

## Soft Delete Pattern
Never hard-delete rooms. Always:
```csharp
room.IsActive = false;
await db.SaveChangesAsync();
```
Filter inactive rooms in all queries:
```csharp
db.Rooms.Where(r => r.IsActive)
```

## Schema Change Rules
- Never rename a column directly — add new, migrate data, drop old
- Never change a column type without a data migration step
- Always keep `CreatedAt` columns immutable (set once, never updated)