## 1. User Authentication
Overview

This feature handles secure user registration, login, and session management. Users can register as guests or hosts and must authenticate before accessing protected resources.

API Endpoints

POST /api/auth/register → Register a new user

POST /api/auth/login → Login and receive a JWT

GET /api/auth/profile → Retrieve logged-in user profile (protected)

POST /api/auth/logout → Invalidate token/session

Input/Output

Register Input (JSON):

{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "SecurePass123",
  "role": "guest"
}


Register Output:

{
  "user_id": "uuid",
  "email": "john@example.com",
  "role": "guest",
  "created_at": "2025-08-27T12:00:00Z"
}


Login Input:

{
  "email": "john@example.com",
  "password": "SecurePass123"
}


Login Output:

{
  "access_token": "jwt-token",
  "expires_in": 3600
}

Validation Rules

Email must be unique and valid format.

Password must be ≥ 8 characters, contain letters + numbers.

Role must be guest | host | admin.

Hash all passwords before storing.

Performance Criteria

Authentication requests < 200ms on average.

Token expiry configurable (default: 1 hour).

High concurrency support (up to 10k logins/day).

## 2. Property Management
Overview

Hosts can add, update, and delete properties. Guests can view/search properties.

API Endpoints

POST /api/properties → Add new property (host only)

GET /api/properties → List all properties with filters

GET /api/properties/{id} → Get property details

PUT /api/properties/{id} → Update property (host only)

DELETE /api/properties/{id} → Delete property (host only)

Input/Output

Create Property Input:

{
  "name": "Seaside Villa",
  "description": "A cozy villa near the beach.",
  "location": "Lagos, Nigeria",
  "price_per_night": 150.00
}


Create Property Output:

{
  "property_id": "uuid",
  "host_id": "uuid",
  "name": "Seaside Villa",
  "location": "Lagos, Nigeria",
  "price_per_night": 150.00,
  "created_at": "2025-08-27T12:00:00Z"
}

Validation Rules

Property name: max 200 chars, required.

Price: must be > 0.

Location: required, max 255 chars.

Description: required.

Performance Criteria

Must return property search results < 500ms with filters (location, price range).

Index location and price_per_night for fast queries.

Support pagination (default 20 per page).

## 3. Booking System
Overview

Guests can book available properties. The system ensures no overlapping bookings and calculates total price.

API Endpoints

POST /api/bookings → Create booking (guest only)

GET /api/bookings → List user bookings

GET /api/bookings/{id} → View booking details

PUT /api/bookings/{id}/cancel → Cancel booking (guest/admin)

Input/Output

Create Booking Input:

{
  "property_id": "uuid",
  "start_date": "2025-09-10",
  "end_date": "2025-09-15"
}


Create Booking Output:

{
  "booking_id": "uuid",
  "property_id": "uuid",
  "user_id": "uuid",
  "start_date": "2025-09-10",
  "end_date": "2025-09-15",
  "total_price": 750.00,
  "status": "pending"
}

Validation Rules

Dates must be valid and end_date > start_date.

No overlapping bookings for the same property.

Total price = price_per_night * nights.

Only guests can book, only admins/guests can cancel.

Performance Criteria

Booking creation < 300ms.

Enforce ACID transactions (prevent double-booking).

Scale for 1,000+ concurrent bookings.
