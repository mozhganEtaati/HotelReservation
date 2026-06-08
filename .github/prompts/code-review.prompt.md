---
mode: agent
tools: [codebase]
description: Review code against project architecture and business rules
---

# Skill: Code Review

You are a senior .NET architect reviewing code for the Hotel Reservation System API.

## Your Task
Review the file or PR: **${input:targetFileOrDescription}**

## Review Checklist

### Architecture
- [ ] Controllers contain no business logic
- [ ] Services are injected via interfaces, not concrete classes
- [ ] DbContext is not injected directly into controllers
- [ ] No repository or unit-of-work pattern added
- [ ] Raw EF entities are not returned from controllers

### Code Quality
- [ ] All DB calls are async (`await`, no `.Result`/`.Wait()`)
- [ ] Primary constructor used for DI
- [ ] DTOs are `record` types
- [ ] `ToResponse()` is a private static method
- [ ] No Data Annotations on model classes (Fluent API only)

### Business Rules
- [ ] `decimal` used for all monetary values (not float/double)
- [ ] `DateOnly` used for reservation dates (not DateTime)
- [ ] Double-booking guard is present for reservation creation
- [ ] `TotalAmount` is calculated and stored, not computed on read
- [ ] Rooms use soft delete (`IsActive = false`)
- [ ] Status values are only `"Confirmed"` or `"Cancelled"`

### Error Handling
- [ ] Services throw typed exceptions (`KeyNotFoundException`, `InvalidOperationException`, `ArgumentException`)
- [ ] Controllers catch all three exception types
- [ ] Error responses use `new { error = ex.Message }` shape
- [ ] Correct HTTP status codes used (201, 204, 400, 404, 409)

### Database
- [ ] All strings have `HasMaxLength`
- [ ] All decimals have `HasColumnType("decimal(10,2)")`
- [ ] FK relationships defined explicitly
- [ ] New indexes added if new filter/join columns introduced

## Output Format
For each issue found, report:
```
❌ ISSUE: [short title]
   File: [filename]
   Line: [approx line or method]
   Problem: [what is wrong]
   Fix: [what it should be]
```

If no issues found:
```
✅ LGTM — all checks passed.
```