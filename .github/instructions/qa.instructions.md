---
applyTo: "**/*Tests.cs, **/*Test.cs, **/*Spec.cs"
---

# QA Agent — Hotel Reservation API

## Test Framework
- xUnit for all tests
- Moq for mocking service interfaces
- EF Core InMemory provider for integration tests
- FluentAssertions for readable assertions (optional but preferred)

## Test Naming Convention
```
MethodName_Scenario_ExpectedResult
```
Examples:
- `CreateAsync_RoomNotFound_ThrowsKeyNotFoundException`
- `CreateAsync_OverlappingDates_ThrowsInvalidOperationException`
- `CancelAsync_AlreadyCancelled_ThrowsInvalidOperationException`
- `CreateAsync_ValidRequest_ReturnsTotalAmountCorrect`

## Test Structure (AAA)
```csharp
[Fact]
public async Task MethodName_Scenario_ExpectedResult()
{
    // Arrange
    var mockService = new Mock<IRoomService>();
    mockService.Setup(...).Returns(...);

    // Act
    var result = await sut.MethodName(request);

    // Assert
    result.Should().NotBeNull();
}
```

## What to Test per Layer

### Service Layer (unit tests — mock DbContext or use InMemory)
- Happy path: valid inputs produce correct output
- `TotalAmount` calculation accuracy (nights × price)
- Double-booking rejection (overlapping dates)
- Cancellation of already-cancelled reservation → exception
- Room not found → `KeyNotFoundException`
- Customer not found → `KeyNotFoundException`
- Inactive room → `InvalidOperationException`
- `CheckOut <= CheckIn` → `ArgumentException`

### Controller Layer (unit tests — mock service)
- 201 returned on successful POST
- 204 returned on successful DELETE
- 404 returned when service throws `KeyNotFoundException`
- 409 returned when service throws `InvalidOperationException`
- 400 returned when service throws `ArgumentException`

### Integration Tests (EF InMemory)
- Full reservation flow: create room → create customer → reserve → verify TotalAmount
- Soft delete: deleted room does not appear in GET /api/rooms
- Cancel: reservation status becomes "Cancelled"

## Edge Cases to Always Cover
- Same check-in and check-out date → invalid
- Check-out before check-in → invalid
- Booking on exact boundary dates of existing reservation → should succeed
- Booking a soft-deleted room → rejected
- Zero-price room (if allowed) → TotalAmount = 0