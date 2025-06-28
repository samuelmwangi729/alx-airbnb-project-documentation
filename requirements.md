# 📄 Backend Feature Specifications for Airbnb Clone

## 🔐 1. User Authentication

### Objective

Enable secure user registration and login functionality for guests and hosts using JWT and optional OAuth.

### API Endpoints

#### `POST /api/auth/register`

Registers a new user (guest or host).

**Input:**

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "SecurePass123",
  "role": "host"
}
```

**Validation Rules:**

- Email must be unique and valid.
- Password must be at least 8 characters.
- Role must be either `guest` or `host`.

**Output:**

```json
{
  "message": "Registration successful",
  "token": "JWT_TOKEN"
}
```

---

#### `POST /api/auth/login`

Authenticates user and returns a JWT token.

**Input:**

```json
{
  "email": "john@example.com",
  "password": "SecurePass123"
}
```

**Output:**

```json
{
  "message": "Login successful",
  "token": "JWT_TOKEN"
}
```

---

#### `GET /api/profile`

Returns the authenticated user’s profile.

**Headers:**`Authorization: Bearer <token>`

**Output:**

```json
{
  "id": "user_id",
  "name": "John Doe",
  "email": "john@example.com",
  "role": "host",
  "profilePhoto": "url"
}
```

### Performance Criteria

- Token generation and authentication must complete within 300ms.
- Support 500+ concurrent logins using horizontal scaling.

---

## 🏡 2. Property Management

### Objective

Allow hosts to create, update, and delete property listings, including details such as title, location, price, and amenities.

### API Endpoints

#### `POST /api/properties`

Creates a new property listing.

**Headers:**`Authorization: Bearer <token>`

**Input:**

```json
{
  "title": "Seaside Villa",
  "description": "Beautiful villa with ocean view",
  "location": "Mombasa, Kenya",
  "price": 120.00,
  "guests": 4,
  "amenities": ["wifi", "pool", "air-conditioning"],
  "availability": {
    "start": "2025-07-01",
    "end": "2025-08-01"
  }
}
```

**Validation Rules:**

- Price must be a positive number.
- Availability dates must be in the future.
- Only authenticated hosts can create listings.

**Output:**

```json
{
  "message": "Property created successfully",
  "propertyId": "abc123"
}
```

---

#### `PUT /api/properties/:id`

Updates an existing property listing.**Note:** Only the owner host can perform this action.

---

#### `DELETE /api/properties/:id`

Deletes a property listing.**Note:** Only the owner host or an admin can perform this action.

### Performance Criteria

- Creation and updates should respond within 500ms.
- Index `location`, `price`, and `availability` fields for search performance.

---

## 🗕️ 3. Booking System

### Objective

Allow guests to book available properties for a specific date range and manage booking statuses.

### API Endpoints

#### `POST /api/bookings`

Creates a booking for a property.

**Headers:**`Authorization: Bearer <token>`

**Input:**

```json
{
  "propertyId": "abc123",
  "startDate": "2025-07-05",
  "endDate": "2025-07-10"
}
```

**Validation Rules:**

- Dates must be within the property’s availability range.
- No overlapping bookings allowed.
- Start date must be before end date.

**Output:**

```json
{
  "message": "Booking successful",
  "bookingId": "book_123",
  "status": "pending"
}
```

---

#### `GET /api/bookings/:id`

Returns details for a specific booking.

---

#### `PATCH /api/bookings/:id/cancel`

Cancels a booking.

**Validation Rules:**

- Only the guest or the host can cancel.
- Must comply with the cancellation policy.

---

#### `GET /api/bookings/status`

Returns all bookings for the current user with statuses.

### Booking Status Values

- `pending`: Awaiting confirmation
- `confirmed`: Approved and locked
- `cancelled`: Cancelled by user or host
- `completed`: Stay is finished

### Performance Criteria

- Prevent double bookings via atomic DB transactions or locks.
- Return booking status within 300ms.
- Use background jobs to send confirmation emails.

---

## 🔐 Security and Authorization

All protected routes require JWT tokens.

### Roles

- **Guest**: Can search, book, cancel.
- **Host**: Can create, update, delete listings.
- **Admin**: Full access to system data and actions.

---

## 🧰 Testing Strategy

- 90%+ unit test coverage for all modules.
- Integration tests for critical flows (e.g., booking, payments).
- Use mocked database and JWTs for automated testing.

---

## 📊 Optimization and Caching

- Use Redis to cache frequently accessed data (e.g., property search).
- Add indexes for faster queries (e.g., by location, price, date).
- Implement pagination (default: 10 results per page).
