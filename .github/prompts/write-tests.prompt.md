---
mode: agent
tools: [codebase, editFiles]
description: Generate full xUnit test coverage for a service or controller
---

# Skill: Write Tests

You are a QA engineer working on the Hotel Reservation System API.

## Your Task
Write complete xUnit tests for: **${input:targetClass}**

## Setup Rules
- Framework: xUnit
- Mocking: Moq
- DB: EF Core InMemory (`UseInMemoryDatabase`)
- Assertions: FluentAssertions (`result.Should()...`)
- Test project name: `HotelReservation.Tests`

## Test File Location
`HotelReservation.Tests/{input:targetClass}Tests.cs`

## Before Writing Tests
1. Read the target class: find it in `Services/` or `Controllers/`
2. Read its interface to understand the contract
3. Identify all public methods and their possible outcomes

## Test Naming Convention
```
MethodName_Scenario_ExpectedResult
```

## Test Structure (AAA strictly)
```csharp
[Fact]
public async Task MethodName_Scenario_ExpectedResult()
{
    // Arrange

    // Act

    // Assert
}
```

## Coverage Requirements

### For Service classes
Cover every method with:
- ✅ Happy path (valid input, correct output)
- ❌ Entity not found → `KeyNotFoundException`
- ❌ Business rule violation → `InvalidOperationException`
- ❌ Invalid input → `ArgumentException`
- 🔢 Calculation accuracy (TotalAmount, nights count)
- 🔒 Constraint check (double-booking, soft delete, already cancelled)

### For Controller classes
Cover every action with:
- ✅ 200/201/204 on success
- ❌ 404 when service throws `KeyNotFoundException`
- ❌ 409 when service throws `InvalidOperationException`
- ❌ 400 when service throws `ArgumentException`

## Edge Cases to Always Include
- Boundary dates (checkout = day after checkin)
- Same checkin and checkout → invalid
- Checkout before checkin → invalid
- Booking on exact boundary of existing reservation → valid
- Soft-deleted room → booking rejected

## Definition of Done
- [ ] All public methods covered
- [ ] All exception paths covered
- [ ] No magic strings — use constants or variables for status values
- [ ] Tests are independent (no shared state between tests)
- [ ] All tests pass: `dotnet test`