# BOM Initial Database Design

## Overview

This document describes the initial relational data model for the BOM MVP.

The first version uses five main entities:

- `users`
- `properties`
- `property_photos`
- `availability_blocks`
- `bookings`

The model is intentionally simple so the MVP can be developed, tested, and evolved without unnecessary complexity.

## Entity Relationship Overview

```text
users
  |
  | 1
  |------< properties
  |            |
  |            |------< property_photos
  |            |
  |            |------< availability_blocks
  |            |
  |            |------< bookings
  |
  |------< bookings
           guest_id
```

## 1. Users

Table: `users`

Stores both property owners and guests.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | User identifier |
| `first_name` | VARCHAR | NOT NULL | User first name |
| `last_name` | VARCHAR | NOT NULL | User last name |
| `email` | VARCHAR | NOT NULL, UNIQUE | Login email |
| `password_hash` | VARCHAR | NOT NULL | Hashed password |
| `role` | ENUM | NOT NULL | `OWNER` or `GUEST` |
| `phone` | VARCHAR | NULL | Contact phone |
| `created_at` | TIMESTAMP | NOT NULL | Creation date |
| `updated_at` | TIMESTAMP | NOT NULL | Last update date |

### Relationships

- One owner can have many properties.
- One guest can have many bookings.

## 2. Properties

Table: `properties`

Stores accommodations published by owners.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Property identifier |
| `owner_id` | UUID | FK, NOT NULL | References `users.id` |
| `title` | VARCHAR | NOT NULL | Listing title |
| `description` | TEXT | NOT NULL | Property description |
| `price_per_night` | NUMERIC(10,2) | NOT NULL | Nightly price |
| `max_guests` | INTEGER | NOT NULL | Maximum guest capacity |
| `bedrooms` | INTEGER | NOT NULL | Number of bedrooms |
| `bathrooms` | INTEGER | NOT NULL | Number of bathrooms |
| `address` | VARCHAR | NOT NULL | Property address |
| `neighborhood` | VARCHAR | NOT NULL | Area within Bombinhas |
| `is_active` | BOOLEAN | NOT NULL, DEFAULT TRUE | Listing availability in the platform |
| `created_at` | TIMESTAMP | NOT NULL | Creation date |
| `updated_at` | TIMESTAMP | NOT NULL | Last update date |

### Relationship

```text
users.id 1 ------ N properties.owner_id
```

A property belongs to exactly one owner.

## 3. Property Photos

Table: `property_photos`

Stores property image URLs.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Photo identifier |
| `property_id` | UUID | FK, NOT NULL | References `properties.id` |
| `url` | TEXT | NOT NULL | Image URL |
| `position` | INTEGER | NOT NULL | Display order |
| `created_at` | TIMESTAMP | NOT NULL | Creation date |

### Relationship

```text
properties.id 1 ------ N property_photos.property_id
```

A property can have multiple photos.

## 4. Availability Blocks

Table: `availability_blocks`

Stores periods during which a property is unavailable.

The MVP assumes that a property is available by default unless a blocking period exists.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Block identifier |
| `property_id` | UUID | FK, NOT NULL | References `properties.id` |
| `start_date` | DATE | NOT NULL | First unavailable date |
| `end_date` | DATE | NOT NULL | End of unavailable period |
| `reason` | ENUM | NOT NULL | Reason for blocking |
| `created_at` | TIMESTAMP | NOT NULL | Creation date |

Suggested reasons:

- `OWNER_BLOCK`
- `MAINTENANCE`
- `OTHER`

### Relationship

```text
properties.id 1 ------ N availability_blocks.property_id
```

### Date Rule

`end_date` must be later than `start_date`.

The application should use half-open date intervals:

```text
[start_date, end_date)
```

This means `end_date` itself is not considered occupied.

## 5. Bookings

Table: `bookings`

Stores booking requests and confirmed reservations.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Booking identifier |
| `property_id` | UUID | FK, NOT NULL | References `properties.id` |
| `guest_id` | UUID | FK, NOT NULL | References `users.id` |
| `check_in` | DATE | NOT NULL | Arrival date |
| `check_out` | DATE | NOT NULL | Departure date |
| `number_of_guests` | INTEGER | NOT NULL | Number of guests |
| `price_per_night` | NUMERIC(10,2) | NOT NULL | Price captured when booking is created |
| `total_price` | NUMERIC(10,2) | NOT NULL | Total booking price |
| `status` | ENUM | NOT NULL | Booking status |
| `created_at` | TIMESTAMP | NOT NULL | Creation date |
| `updated_at` | TIMESTAMP | NOT NULL | Last update date |

Suggested statuses:

- `PENDING`
- `CONFIRMED`
- `REJECTED`
- `CANCELLED`

### Relationships

```text
properties.id 1 ------ N bookings.property_id

users.id      1 ------ N bookings.guest_id
```

## Booking Overlap Rule

Two confirmed bookings for the same property must not overlap.

For an existing booking:

```text
existingCheckIn
existingCheckOut
```

and a new booking:

```text
newCheckIn
newCheckOut
```

there is an overlap when:

```text
newCheckIn < existingCheckOut
AND
newCheckOut > existingCheckIn
```

Example:

```text
Existing booking:
2027-01-10 -> 2027-01-17

New booking:
2027-01-15 -> 2027-01-20

Result:
CONFLICT
```

But this is valid:

```text
Booking A:
2027-01-10 -> 2027-01-17

Booking B:
2027-01-17 -> 2027-01-22

Result:
VALID
```

The first guest checks out on the same day the second guest checks in.

## Main Constraints

The application and database should enforce the following rules:

- User email must be unique.
- `price_per_night` must be greater than zero.
- `max_guests` must be greater than zero.
- `bedrooms` cannot be negative.
- `bathrooms` cannot be negative.
- `check_out` must be later than `check_in`.
- `number_of_guests` must be greater than zero.
- `number_of_guests` cannot exceed `properties.max_guests`.
- A confirmed booking cannot overlap another confirmed booking for the same property.
- Booking date ranges cannot overlap an availability block.

## Initial Index Recommendations

Indexes should be considered for:

- `users.email`
- `properties.owner_id`
- `properties.neighborhood`
- `property_photos.property_id`
- `availability_blocks.property_id`
- `availability_blocks.start_date`
- `availability_blocks.end_date`
- `bookings.property_id`
- `bookings.guest_id`
- `bookings.status`
- `bookings.check_in`
- `bookings.check_out`

These indexes will support common search and relationship queries.

## Notes for Future Evolution

The MVP intentionally stores `neighborhood` as a text field.

If BOM expands, the model may introduce additional tables such as:

- `neighborhoods`
- `cities`
- `payments`
- `reviews`
- `favorites`
- `messages`
- `amenities`
- `property_amenities`

These tables are not required for the initial MVP.
