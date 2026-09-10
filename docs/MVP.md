# BOM MVP

## Overview

BOM is a short-term vacation rental platform focused exclusively on Bombinhas, Santa Catarina, Brazil.

The goal of the MVP is to provide the smallest useful version of the product that allows property owners to publish accommodations and guests to search for available properties and request bookings.

## MVP Goal

The first version of BOM must support this complete flow:

### Property Owner

1. Create an account.
2. Log in.
3. Create one or more property listings.
4. Add basic property information.
5. Define the price per night.
6. Upload property photos.
7. Block unavailable dates.
8. Receive booking requests.
9. Accept or reject booking requests.

### Guest

1. Create an account.
2. Log in.
3. Select check-in and check-out dates.
4. Specify the number of guests.
5. Search for available properties.
6. View property details and photos.
7. Send a booking request.

## Core Features

### Authentication

- User registration.
- User login.
- JWT-based authentication.
- Two initial roles:
  - `OWNER`
  - `GUEST`

### Property Management

Property owners can:

- Create a property.
- View their properties.
- Edit a property.
- Delete or deactivate a property.
- Add a title and description.
- Set a price per night.
- Define maximum guest capacity.
- Define number of bedrooms and bathrooms.
- Add address and neighborhood.
- Upload multiple photos.

### Availability

- Properties are considered available by default.
- Owners can create unavailable date ranges.
- The system must exclude unavailable properties from search results.

### Property Search

Guests can search using:

- Check-in date.
- Check-out date.
- Number of guests.
- Neighborhood.

The system should only return properties that:

- Are active.
- Support the requested number of guests.
- Are available for the complete requested period.

### Booking Requests

Guests can request a booking for a property.

A booking stores:

- Property.
- Guest.
- Check-in date.
- Check-out date.
- Number of guests.
- Price per night at the time of booking.
- Total price.
- Booking status.

Initial booking statuses:

- `PENDING`
- `CONFIRMED`
- `REJECTED`
- `CANCELLED`

### Booking Rules

- `checkOut` must be later than `checkIn`.
- The requested number of guests cannot exceed the property's maximum capacity.
- Confirmed bookings cannot overlap.
- A check-out date is not considered an occupied night.
- Therefore, one guest can check out on the same date another guest checks in.

## Out of Scope for the MVP

The following features are intentionally excluded from the first version:

- Online payments.
- Pix integration.
- Internal chat.
- Reviews and ratings.
- Favorites.
- Promotional coupons.
- Recommendation algorithms.
- Artificial intelligence features.
- Interactive maps.
- Multi-city support.
- Loyalty or points systems.
- Advanced pricing rules.

These features may be considered in future versions.

## Initial Tech Stack

- React Native
- Expo
- TypeScript
- NestJS
- PostgreSQL
- TypeORM
- Docker
- REST API
- JWT
- Swagger
- Jest
- Git
- GitHub

## Definition of Done for the MVP

The MVP will be considered complete when:

1. An owner can register and log in.
2. An owner can create and manage a property.
3. An owner can upload property photos.
4. An owner can block unavailable dates.
5. A guest can register and log in.
6. A guest can search by dates and number of guests.
7. Search results only show properties available for the requested dates.
8. A guest can request a booking.
9. An owner can accept or reject the booking request.
10. The system prevents conflicting confirmed bookings.

## Future Vision

BOM may later evolve into a broader local platform for short-term rentals in Bombinhas, with payments, reviews, messaging, maps, promotions, and additional tools for property owners and guests.
