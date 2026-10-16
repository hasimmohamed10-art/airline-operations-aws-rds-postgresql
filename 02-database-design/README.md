# Lab 02 — Airline Database Design and Sample Data

## Overview

This lab covers the design and implementation of an Airline Operations database using Amazon RDS for PostgreSQL. It includes database and schema creation, relational table design, sample data insertion, and validation of table relationships.

## Objectives

- Create the `airlinedb` database.
- Create `airline` and `booking` schemas.
- Create six relational tables with primary and foreign keys.
- Insert sample airport, aircraft, flight, passenger, booking, and ticket records.
- Validate table structures, row counts, and relationships using SQL queries.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL server | 18.3 |
| Database | `airlinedb` |
| SQL client | `psql` 18.4 |
| Connection security | SSL/TLS |

## Database Structure

### Schemas

- `airline` — Stores airport, aircraft, and flight information.
- `booking` — Stores passenger, booking, and ticket information.

### Tables

| Table | Purpose |
|---|---|
| `airline.airports` | Airport codes, names, cities, and countries |
| `airline.aircraft` | Aircraft registration, model, and seat capacity |
| `airline.flights` | Flight numbers, routes, aircraft assignments, and schedules |
| `booking.passengers` | Passenger names and email addresses |
| `booking.bookings` | Passenger bookings and booking status |
| `booking.tickets` | Ticket seat numbers and ticket status |

## Implementation Steps

### Step 1 — Create the database and schemas

Created the `airlinedb` database and established the `airline` and `booking` schemas to separate operational flight data from booking data.

### Step 2 — Create relational tables

Created the six tables with appropriate data types, primary keys, uniqueness constraints, and foreign-key relationships.

Key relationships include:

- Flights reference departure and arrival airports.
- Flights can reference an aircraft.
- Bookings reference passengers and flights.
- Tickets reference bookings.

### Step 3 — Insert sample data

Inserted representative records for:

- Three airports: Chennai (MAA), Delhi (DEL), and Dubai (DXB).
- Two aircraft: A320 and B737.
- Two flights: AI101 and AI202.
- Three passengers.
- Three bookings.
- Three ticket records.

The records cover confirmed and pending bookings, along with ticket status values.

### Step 4 — Validate table structure

Queried PostgreSQL's `information_schema.tables` to verify that all six expected tables existed in the `airline` and `booking` schemas.

### Step 5 — Validate row counts

Executed SQL count queries against all six tables.

| Table | Validated rows |
|---|---:|
| `airline.airports` | 3 |
| `airline.aircraft` | 2 |
| `airline.flights` | 2 |
| `booking.passengers` | 3 |
| `booking.bookings` | 3 |
| `booking.tickets` | 3 |

### Step 6 — Validate relationships

Used SQL joins to verify the relationships between flights, airports, aircraft, passengers, bookings, and tickets.

The checks confirmed the following flight routes:

- AI101: Chennai (MAA) to Delhi (DEL), using an A320.
- AI202: Delhi (DEL) to Dubai (DXB), using a B737.

The booking and ticket checks also confirmed that the sample records were linked to the expected passengers and flights.

## Validation Results

- The `airline` and `booking` schemas were created.
- All six expected tables were present.
- The recorded row counts matched the expected sample data.
- Join queries returned the expected flight, booking, and ticket relationships.

## Screenshots

Screenshots are stored in the `screenshots/` directory.

| Screenshot | Evidence |
|---|---|
| [Database and schemas created](screenshots/02-01-database-and-schemas-created.png) | Database and schema creation |
| [Create airports and aircraft tables](screenshots/02-02-create-airports-and-aircraft-tables.png) | Initial table creation |
| [Create flights, passengers, and bookings](screenshots/02-03-create-flights-passengers-and-bookings-tables.png) | Relational tables and constraints |
| [Create tickets table](screenshots/02-04-create-tickets-table.png) | Ticket table creation |
| [Verify tables](screenshots/02-05-verify-airline-and-booking-tables.png) | Table validation |
| [Insert airport, aircraft, and flight data](screenshots/02-06-insert-airports-aircraft-and-flights.png) | Operational sample data |
| [Insert passenger, booking, and ticket data](screenshots/02-07-insert-passengers-bookings-and-tickets.png) | Booking sample data |
| [Row-count validation](screenshots/02-08-table-row-count-validation.png) | Counts across six tables |
| [Flight and booking join validation](screenshots/02-09-flight-and-booking-join-validation.png) | Flight and booking relationships |
| [Ticket relationship validation](screenshots/02-10-ticket-relationship-validation.png) | Ticket, passenger, booking, and flight relationships |

## Outcome

The Airline Operations database structure and sample data were created and validated on Amazon RDS for PostgreSQL. Table counts and SQL join results confirmed the expected relationships between the database entities.

## Related Documentation

- [`commands.md`](commands.md) — Implementation commands and SQL statements.
- [`commands-with-output.md`](commands-with-output.md) — Recorded commands and validation outputs.

## Security Note

Connection details and passwords are not included in this documentation. Review screenshots before publishing to ensure that private AWS account information and database endpoints are not exposed.
