# Product Requirements Document (PRD)
## Hotel Reservation System API

**Version:** 1.0  
**Date:** 2026-06-08  
**Status:** Approved  

---

## 1. Overview

### 1.1 Purpose
The Hotel Reservation System is a backend REST API that enables hotel staff and integrated front-ends to manage rooms, customers, and reservations programmatically. It exposes no UI of its own.

### 1.2 Goals
- Allow hotel operators to add and remove rooms from inventory.
- Allow customers (or staff on their behalf) to reserve available rooms for specific date ranges.
- Prevent double-bookings automatically.
- Calculate and expose the exact amount a customer owes for a reservation.
- Provide room availability and detail at any time.

### 1.3 Non-Goals (v1.0)
- No authentication / authorization (planned for v2.0).
- No payment processing.
- No front-end or admin dashboard.
- No email / SMS notification system.
- No multi-property / multi-hotel support.

---

## 2. Stakeholders

| Role | Responsibility |
|------|---------------|
| Hotel Manager | Manages room inventory (add/remove) |
| Front Desk Staff | Creates and cancels reservations |
| Integrating Systems | Any third-party front-end or PMS consuming the API |

---

## 3. Functional Requirements

### 3.1 Room Management

| ID | Requirement |
|----|-------------|
| RM-01 | The system shall allow adding a new room with: room number, type, price per night, and floor. |
| RM-02 | Room numbers shall be unique across the system. |
| RM-03 | The system shall allow soft-deleting a room (marking it inactive), preserving historical reservation records. |
| RM-04 | The system shall return a list of all active rooms. |
| RM-05 | The system shall return full details of a single room by its ID. |
| RM-06 | Inactive (deleted) rooms shall not appear in availability listings. |

### 3.2 Reservation Management

| ID | Requirement |
|----|-------------|
| RS-01 | The system shall allow creating a reservation for a valid room and customer with check-in and check-out dates. |
| RS-02 | Check-out date must be strictly after check-in date. |
| RS-03 | The system shall reject a reservation if the room is already confirmed-booked for any overlapping date range. |
| RS-04 | The system shall automatically calculate `TotalAmount = PricePerNight × NumberOfNights` at booking time. |
| RS-05 | The system shall allow cancelling a reservation by ID, setting its status to `Cancelled`. |
| RS-06 | Cancelling an already-cancelled reservation shall return an error. |
| RS-07 | The system shall return full reservation details including: nights count, price per night, total amount, and current status. |

### 3.3 Customer Management

| ID | Requirement |
|----|-------------|
| CM-01 | Customers are pre-existing records identified by `CustomerId`. |
| CM-02 | Each customer has: full name, email (unique), and optional phone. |

---

## 4. Non-Functional Requirements

| ID | Requirement |
|----|-------------|
| NFR-01 | API response time shall be under 300ms for all endpoints under normal load. |
| NFR-02 | The system shall use SQL Server as the relational database. |
| NFR-03 | All date handling shall be timezone-aware (UTC stored, `DateOnly` for check-in/out). |
| NFR-04 | The system shall return structured JSON error responses on all failure paths. |
| NFR-05 | The codebase shall follow clean separation: Controllers → Services → DbContext. |
| NFR-06 | Swagger/OpenAPI documentation shall be auto-generated and accessible in development. |

---

## 5. API Endpoints Summary

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/rooms` | List all active rooms |
| GET | `/api/rooms/{id}` | Get room details |
| POST | `/api/rooms` | Add a new room |
| DELETE | `/api/rooms/{id}` | Soft-delete a room |
| POST | `/api/reservations` | Create a reservation |
| GET | `/api/reservations/{id}` | Get reservation details + bill |
| DELETE | `/api/reservations/{id}` | Cancel a reservation |

---

## 6. Data Models

### Room
| Field | Type | Notes |
|-------|------|-------|
| RoomId | int | PK, auto-increment |
| RoomNumber | string | Unique |
| Type | string | Single, Double, Suite… |
| PricePerNight | decimal | > 0 |
| Floor | int | |
| IsActive | bool | Soft delete flag |

### Customer
| Field | Type | Notes |
|-------|------|-------|
| CustomerId | int | PK |
| FullName | string | |
| Email | string | Unique |
| Phone | string? | Optional |

### Reservation
| Field | Type | Notes |
|-------|------|-------|
| ReservationId | int | PK |
| RoomId | int | FK → Rooms |
| CustomerId | int | FK → Customers |
| CheckInDate | DateOnly | |
| CheckOutDate | DateOnly | > CheckInDate |
| TotalAmount | decimal | Auto-calculated |
| Status | string | Confirmed / Cancelled |

---

## 7. Error Handling

| HTTP Status | When Used |
|-------------|-----------|
| 200 OK | Successful GET |
| 201 Created | Successful POST |
| 204 No Content | Successful DELETE |
| 400 Bad Request | Invalid input (e.g. bad dates) |
| 404 Not Found | Resource does not exist |
| 409 Conflict | Double-booking, already cancelled |

---

## 8. Future Roadmap (v2.0+)
- JWT-based authentication and role management (Admin, Staff).
- Customer self-service endpoints (register, view own reservations).
- Room availability search by date range.
- Partial refund calculation on cancellation.
- Pagination and filtering on list endpoints.
- Integration with payment gateway.