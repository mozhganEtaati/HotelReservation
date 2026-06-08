---
mode: agent
tools: [codebase, editFiles, runCommands]
description: Safely add an EF Core migration for a schema change
---

# Skill: Add EF Core Migration

You are a database engineer on the Hotel Reservation System API.

## Your Task
Create a migration for: **${input:changeDescription}**

## Before Creating the Migration

1. Read `Data/AppDbContext.cs` — understand current schema
2. Read `Models/` — check if model changes are needed
3. Check `Migrations/` — verify no unapplied migrations exist first

## Rules to Follow

### Fluent API Only
Configure everything in `OnModelCreating` — never add Data Annotations to models.

Required config for any new entity:
```csharp
mb.Entity<NewEntity>(e =>
{
    e.HasKey(x => x.Id);
    e.Property(x => x.Name).IsRequired().HasMaxLength(100);
    e.Property(x => x.Amount).HasColumnType("decimal(10,2)");
    e.Property(x => x.CreatedAt).HasDefaultValueSql("GETUTCDATE()");
});
```

### Naming Convention
```
YYYYMMDD_PascalCaseDescription
```
Example: `20260608_AddRoomDescription`

### Index Rules
Add indexes for:
- Every new FK column
- Every column used in a WHERE clause
- Unique constraints (`HasIndex(...).IsUnique()`)

## Steps

1. Apply model + DbContext changes
2. Run migration:
```bash
dotnet ef migrations add ${input:migrationName} --project HotelReservation.API
```
3. Review the generated migration file in `Migrations/` — verify it matches intent
4. Apply to DB:
```bash
dotnet ef database update --project HotelReservation.API
```

## Safety Rules
- Never rename a column directly — add new column, migrate data, drop old column in separate migrations
- Never change a column type without a data migration step
- Never delete a migration that has been applied to any environment
- `CreatedAt` is immutable — never update it after insert

## Definition of Done
- [ ] Migration file generated in `Migrations/`
- [ ] Migration content reviewed and matches intent
- [ ] `dotnet ef database update` runs without errors
- [ ] No existing data is lost