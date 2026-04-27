# Database Schema - ShowTime

## Tables

### 1. venues

CREATE TABLE venues (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    city TEXT NOT NULL,
    capacity INT NOT NULL CHECK (capacity > 0),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_venues_city ON venues(city);


---

### 2. events

CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    venue_id INT NOT NULL REFERENCES venues(id) ON DELETE CASCADE,
    start_time TIMESTAMP NOT NULL,
    status TEXT NOT NULL CHECK (status IN ('upcoming','on_sale','sold_out','cancelled')),
    total_seats INT NOT NULL CHECK (total_seats > 0),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_events_status ON events(status);


---

### 3. users

CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    phone TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);


---

### 4. seats

CREATE TABLE seats (
    id SERIAL PRIMARY KEY,
    event_id INT NOT NULL REFERENCES events(id) ON DELETE CASCADE,
    section TEXT NOT NULL,
    row TEXT NOT NULL,
    number INT NOT NULL,
    price NUMERIC NOT NULL CHECK (price > 0),
    category TEXT NOT NULL CHECK (category IN ('VIP','Premium','General')),
    status TEXT NOT NULL CHECK (status IN ('available','held','booked')),
    held_until TIMESTAMP,
    held_by UUID REFERENCES users(id),
    version INT NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_seats_event_status ON seats(event_id, status);


---

### 5. bookings

CREATE TABLE bookings (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    event_id INT NOT NULL REFERENCES events(id) ON DELETE CASCADE,
    status TEXT NOT NULL CHECK (status IN ('pending','confirmed','failed','refunded')),
    total_amount NUMERIC NOT NULL CHECK (total_amount > 0),
    payment_ref TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_bookings_user ON bookings(user_id, created_at DESC);

CREATE INDEX idx_bookings_pending 
ON bookings(status) 
WHERE status IN ('pending','failed');


---

### 6. booking_seats

CREATE TABLE booking_seats (
    id SERIAL PRIMARY KEY,
    booking_id UUID NOT NULL REFERENCES bookings(id) ON DELETE CASCADE,
    seat_id INT NOT NULL REFERENCES seats(id) ON DELETE CASCADE
);

CREATE INDEX idx_booking_seats_booking ON booking_seats(booking_id);


---

## Design Decisions

### Why UUID for bookings and users?
UUIDs are unpredictable and prevent enumeration attacks. They also allow id generation before DB insert.

### Why version column in seats?
Used for optimistic locking. Prevents race conditions when multiple users try to update same seat.

### Why held_until?
Seats are temporarily reserved during payment. This prevents permanent blocking if user abandons.

### Why partial index on bookings?
Only pending/failed bookings are queried frequently by workers. This keeps index small and fast.
