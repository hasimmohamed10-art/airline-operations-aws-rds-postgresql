# Lab 02 — Database Design and Sample Data Commands

## Overview

This document records the PostgreSQL commands used to create the Airline Operations database, define its schemas and tables, insert sample records, and validate the table structure and relationships.

**Environment:** Amazon RDS for PostgreSQL 18.3, accessed through `psql` 18.4 over SSL/TLS.

> Security note: Replace `<RDS_ENDPOINT>` with the actual endpoint locally. Do not commit a real endpoint, password, or AWS account information to GitHub.

## Step 1 — Connect to the RDS PostgreSQL instance

Open Windows PowerShell and connect using the SSL certificate bundle saved in Downloads.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

Enter the PostgreSQL password when prompted. Do not include the password in a command or documentation file.

## Step 2 — Create the Airline database

Execute the following command while connected to `mydb`:

```sql
CREATE DATABASE airlinedb;
```

Connect to the new database:

```sql
\c airlinedb
```

`\c` is a `psql` command used to change the current database connection.

## Step 3 — Create the schemas

```sql
CREATE SCHEMA airline;
CREATE SCHEMA booking;
```

- `airline` stores airport, aircraft, and flight information.
- `booking` stores passenger, booking, and ticket information.

## Step 4 — Create the airport table

```sql
CREATE TABLE airline.airports (
    airport_id SERIAL PRIMARY KEY,
    airport_code VARCHAR(3) UNIQUE NOT NULL,
    airport_name VARCHAR(60) NOT NULL,
    city VARCHAR(50) NOT NULL,
    country VARCHAR(50) NOT NULL
);
```

The primary key uniquely identifies each airport. The unique constraint prevents duplicate airport codes.

## Step 5 — Create the aircraft table

```sql
CREATE TABLE airline.aircraft (
    aircraft_id SERIAL PRIMARY KEY,
    registration_no VARCHAR(10) UNIQUE NOT NULL,
    aircraft_model VARCHAR(30) NOT NULL,
    seat_capacity SMALLINT NOT NULL
);
```

The table stores aircraft registration numbers, models, and seating capacity.

## Step 6 — Create the flights table

```sql
CREATE TABLE airline.flights (
    flight_id SERIAL PRIMARY KEY,
    flight_number VARCHAR(10) UNIQUE NOT NULL,
    departure_airport INT NOT NULL
        REFERENCES airline.airports(airport_id),
    arrival_airport INT NOT NULL
        REFERENCES airline.airports(airport_id),
    aircraft_id INT
        REFERENCES airline.aircraft(aircraft_id),
    departure_time TIMESTAMPTZ NOT NULL,
    arrival_time TIMESTAMPTZ NOT NULL
);
```

Foreign keys connect flights to their departure airports, arrival airports, and assigned aircraft.

## Step 7 — Create the passengers table

```sql
CREATE TABLE booking.passengers (
    passenger_id SERIAL PRIMARY KEY,
    first_name VARCHAR(40) NOT NULL,
    last_name VARCHAR(40) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL
);
```

The email column is unique and cannot be null.

## Step 8 — Create the bookings table

```sql
CREATE TABLE booking.bookings (
    booking_id SERIAL PRIMARY KEY,
    passenger_id INT NOT NULL
        REFERENCES booking.passengers(passenger_id),
    flight_id INT NOT NULL
        REFERENCES airline.flights(flight_id),
    booking_date TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    booking_status VARCHAR(15) NOT NULL
);
```

Each booking references an existing passenger and flight. The booking date defaults to the current timestamp.

## Step 9 — Create the tickets table

```sql
CREATE TABLE booking.tickets (
    ticket_id SERIAL PRIMARY KEY,
    booking_id INT NOT NULL
        REFERENCES booking.bookings(booking_id),
    seat_no VARCHAR(5) NOT NULL,
    ticket_status VARCHAR(15) NOT NULL
);
```

Each ticket references an existing booking.

## Step 10 — Insert airport records

```sql
INSERT INTO airline.airports
    (airport_code, airport_name, city, country)
VALUES
    ('MAA', 'Chennai International Airport', 'Chennai', 'India'),
    ('DEL', 'Indira Gandhi International Airport', 'Delhi', 'India'),
    ('DXB', 'Dubai International Airport', 'Dubai', 'UAE');
```

## Step 11 — Insert aircraft records

```sql
INSERT INTO airline.aircraft
    (registration_no, aircraft_model, seat_capacity)
VALUES
    ('VT-ABC', 'A320', 180),
    ('VT-XYZ', 'B737', 189);
```

## Step 12 — Insert flight records

```sql
INSERT INTO airline.flights
    (flight_number, departure_airport, arrival_airport,
     aircraft_id, departure_time, arrival_time)
VALUES
    ('AI101', 1, 2, 1,
     '2026-10-05 08:00+05:30',
     '2026-10-05 10:45+05:30'),
    ('AI202', 2, 3, 2,
     '2026-10-05 14:00+05:30',
     '2026-10-05 16:30+04:00');
```

## Step 13 — Insert passenger records

```sql
INSERT INTO booking.passengers
    (first_name, last_name, email)
VALUES
    ('Arun', 'Kumar', 'arun.kumar@example.com'),
    ('Rahim', 'Ahmed', 'rahim.ahmed@example.com'),
    ('John', 'Paul', 'john.paul@example.com');
```

## Step 14 — Insert booking records

```sql
INSERT INTO booking.bookings
    (passenger_id, flight_id, booking_status)
VALUES
    (1, 1, 'CONFIRMED'),
    (2, 1, 'CONFIRMED'),
    (3, 2, 'PENDING');
```

## Step 15 — Insert ticket records

```sql
INSERT INTO booking.tickets
    (booking_id, seat_no, ticket_status)
VALUES
    (1, '12A', 'CONFIRMED'),
    (2, '12B', 'CONFIRMED'),
    (3, '08C', 'HELD');
```

## Step 16 — Verify the tables

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
  AND table_type = 'BASE TABLE'
ORDER BY table_schema, table_name;
```

The expected result contains six tables across the `airline` and `booking` schemas.

## Step 17 — Validate row counts

```sql
SELECT 'airline.airports' AS table_name, COUNT(*) AS rows
FROM airline.airports

UNION ALL

SELECT 'airline.aircraft', COUNT(*)
FROM airline.aircraft

UNION ALL

SELECT 'airline.flights', COUNT(*)
FROM airline.flights

UNION ALL

SELECT 'booking.passengers', COUNT(*)
FROM booking.passengers

UNION ALL

SELECT 'booking.bookings', COUNT(*)
FROM booking.bookings

UNION ALL

SELECT 'booking.tickets', COUNT(*)
FROM booking.tickets

ORDER BY table_name;
```

Expected row counts:

| Table | Rows |
|---|---:|
| `airline.airports` | 3 |
| `airline.aircraft` | 2 |
| `airline.flights` | 2 |
| `booking.passengers` | 3 |
| `booking.bookings` | 3 |
| `booking.tickets` | 3 |

## Step 18 — Validate flight and airport relationships

```sql
SELECT
    f.flight_number,
    dep.airport_code AS departure_code,
    dep.city AS departure_city,
    arr.airport_code AS arrival_code,
    arr.city AS arrival_city,
    a.aircraft_model
FROM airline.flights f
JOIN airline.airports dep
    ON f.departure_airport = dep.airport_id
JOIN airline.airports arr
    ON f.arrival_airport = arr.airport_id
JOIN airline.aircraft a
    ON f.aircraft_id = a.aircraft_id
ORDER BY f.flight_number;
```

This query checks flight routes and their aircraft assignments.

## Step 19 — Validate bookings and passengers

```sql
SELECT
    b.booking_id,
    p.first_name,
    p.last_name,
    f.flight_number,
    b.booking_status
FROM booking.bookings b
JOIN booking.passengers p
    ON b.passenger_id = p.passenger_id
JOIN airline.flights f
    ON b.flight_id = f.flight_id
ORDER BY b.booking_id;
```

This query verifies that each booking is linked to a passenger and flight.

## Step 20 — Validate tickets and bookings

```sql
SELECT
    t.ticket_id,
    t.seat_no,
    t.ticket_status,
    b.booking_id,
    p.first_name,
    p.last_name,
    f.flight_number
FROM booking.tickets t
JOIN booking.bookings b
    ON t.booking_id = b.booking_id
JOIN booking.passengers p
    ON b.passenger_id = p.passenger_id
JOIN airline.flights f
    ON b.flight_id = f.flight_id
ORDER BY t.ticket_id;
```

This query verifies ticket, booking, passenger, and flight relationships.

## Completion Summary

The database structure and sample data were created and validated. The recorded checks confirmed six expected tables, the expected row counts, and the relationships between flights, airports, aircraft, passengers, bookings, and tickets.

See [`commands-with-output.md`](commands-with-output.md) for the recorded validation outputs and [`README.md`](README.md) for the lab overview.
