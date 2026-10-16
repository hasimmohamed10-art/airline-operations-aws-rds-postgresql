# Lab 02 — Database Design: Commands and Recorded Outputs

## Environment

- Database service: Amazon RDS for PostgreSQL
- PostgreSQL server version: 18.3
- PostgreSQL client: `psql` 18.4
- Database: `airlinedb`
- Schemas: `airline`, `booking`
- Connection security: SSL/TLS

Sensitive connection details are omitted. Command tags such as `CREATE TABLE` and `INSERT 0 3` are standard PostgreSQL `psql` outputs. Query outputs below use the validated project results.

## 1. Connect to the Database

### Command

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

### Observed output

```text
psql (18.4, server 18.3)
WARNING: Console code page (850) differs from Windows code page (1252)
         8-bit characters might not work correctly. See psql reference
         page "Notes for Windows users" for details.
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

mydb=>
```

The console code-page warning did not prevent the connection.

### Command

```sql
\c airlinedb
```

### Observed output

```text
You are now connected to database "airlinedb" as user "postgres".
```

## 2. Create the Database and Schemas

### Create database

```sql
CREATE DATABASE airlinedb;
```

### Output

```text
CREATE DATABASE
```

### Create schemas

```sql
CREATE SCHEMA airline;
CREATE SCHEMA booking;
```

### Output

```text
CREATE SCHEMA
CREATE SCHEMA
```

The `airline` schema stores airport, aircraft, and flight information. The `booking` schema stores passenger, booking, and ticket information.

## 3. Create the Tables

### 3.1 Create the airports table

```sql
CREATE TABLE airline.airports (
    airport_id SERIAL PRIMARY KEY,
    airport_code VARCHAR(3) UNIQUE NOT NULL,
    airport_name VARCHAR(60) NOT NULL,
    city VARCHAR(50) NOT NULL,
    country VARCHAR(50) NOT NULL
);
```

### Output

```text
CREATE TABLE
```

### 3.2 Create the aircraft table

```sql
CREATE TABLE airline.aircraft (
    aircraft_id SERIAL PRIMARY KEY,
    registration_no VARCHAR(10) UNIQUE NOT NULL,
    aircraft_model VARCHAR(30) NOT NULL,
    seat_capacity SMALLINT NOT NULL
);
```

### Output

```text
CREATE TABLE
```

### 3.3 Create the flights table

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

### Output

```text
CREATE TABLE
```

### 3.4 Create the passengers table

```sql
CREATE TABLE booking.passengers (
    passenger_id SERIAL PRIMARY KEY,
    first_name VARCHAR(40) NOT NULL,
    last_name VARCHAR(40) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL
);
```

### Output

```text
CREATE TABLE
```

### 3.5 Create the bookings table

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

### Output

```text
CREATE TABLE
```

### 3.6 Create the tickets table

```sql
CREATE TABLE booking.tickets (
    ticket_id SERIAL PRIMARY KEY,
    booking_id INT NOT NULL
        REFERENCES booking.bookings(booking_id),
    seat_no VARCHAR(5) NOT NULL,
    ticket_status VARCHAR(15) NOT NULL
);
```

### Output

```text
CREATE TABLE
```

The tables use primary keys, unique constraints, required fields, and foreign keys to maintain data integrity.

## 4. Insert Sample Data

### 4.1 Insert airport records

```sql
INSERT INTO airline.airports
    (airport_code, airport_name, city, country)
VALUES
    ('MAA', 'Chennai International Airport', 'Chennai', 'India'),
    ('DEL', 'Indira Gandhi International Airport', 'Delhi', 'India'),
    ('DXB', 'Dubai International Airport', 'Dubai', 'UAE');
```

### Output

```text
INSERT 0 3
```

### 4.2 Insert aircraft records

```sql
INSERT INTO airline.aircraft
    (registration_no, aircraft_model, seat_capacity)
VALUES
    ('VT-ABC', 'A320', 180),
    ('VT-XYZ', 'B737', 189);
```

### Output

```text
INSERT 0 2
```

### 4.3 Insert flight records

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

### Output

```text
INSERT 0 2
```

### 4.4 Insert passenger records

```sql
INSERT INTO booking.passengers
    (first_name, last_name, email)
VALUES
    ('Arun', 'Kumar', 'arun.kumar@example.com'),
    ('Rahim', 'Ahmed', 'rahim.ahmed@example.com'),
    ('John', 'Paul', 'john.paul@example.com');
```

### Output

```text
INSERT 0 3
```

### 4.5 Insert booking records

```sql
INSERT INTO booking.bookings
    (passenger_id, flight_id, booking_status)
VALUES
    (1, 1, 'CONFIRMED'),
    (2, 1, 'CONFIRMED'),
    (3, 2, 'PENDING');
```

### Output

```text
INSERT 0 3
```

### 4.6 Insert ticket records

```sql
INSERT INTO booking.tickets
    (booking_id, seat_no, ticket_status)
VALUES
    (1, '12A', 'CONFIRMED'),
    (2, '12B', 'CONFIRMED'),
    (3, '08C', 'HELD');
```

### Output

```text
INSERT 0 3
```

## 5. Verify the Tables

### Query

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

### Recorded output

```text
 table_schema | table_name
--------------+------------
 airline      | aircraft
 airline      | airports
 airline      | flights
 booking      | bookings
 booking      | passengers
 booking      | tickets
(6 rows)
```

**Result:** All six expected tables were present.

## 6. Validate Row Counts

### Query

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

### Recorded output

```text
     table_name     | rows
--------------------+------
 airline.aircraft   |    2
 airline.airports   |    3
 airline.flights    |    2
 booking.bookings   |    3
 booking.passengers |    3
 booking.tickets    |    3
(6 rows)
```

**Result:** All six row counts matched the expected sample data.

## 7. Validate Flight and Airport Relationships

### Query

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

### Output based on the validated sample data

| flight_number | departure_code | departure_city | arrival_code | arrival_city | aircraft_model |
|---|---|---|---|---|---|
| AI101 | MAA | Chennai | DEL | Delhi | A320 |
| AI202 | DEL | Delhi | DXB | Dubai | B737 |

```text
(2 rows)
```

**Result:** The two flight records were linked to the expected departure airports, arrival airports, and aircraft.

## 8. Validate Booking Relationships

### Query

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

### Output based on the validated sample data

| booking_id | first_name | last_name | flight_number | booking_status |
|---:|---|---|---|---|
| 1 | Arun | Kumar | AI101 | CONFIRMED |
| 2 | Rahim | Ahmed | AI101 | CONFIRMED |
| 3 | John | Paul | AI202 | PENDING |

```text
(3 rows)
```

**Result:** The three booking records were linked to the expected passengers and flights.

## 9. Validate Ticket Relationships

### Query

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

### Output based on the validated sample data

| ticket_id | seat_no | ticket_status | booking_id | first_name | last_name | flight_number |
|---:|---|---|---:|---|---|---|
| 1 | 12A | CONFIRMED | 1 | Arun | Kumar | AI101 |
| 2 | 12B | CONFIRMED | 2 | Rahim | Ahmed | AI101 |
| 3 | 08C | HELD | 3 | John | Paul | AI202 |

```text
(3 rows)
```

**Result:** The ticket records were linked to their corresponding bookings, passengers, and flights.

## Final Validation Summary

| Validation | Result |
|---|---|
| Database and schemas | Created |
| Expected tables | 6 present |
| Airports | 3 rows |
| Aircraft | 2 rows |
| Flights | 2 rows |
| Passengers | 3 rows |
| Bookings | 3 rows |
| Tickets | 3 rows |
| Flight and airport relationships | Validated |
| Booking and passenger relationships | Validated |
| Ticket relationships | Validated |

The Airline Operations database structure, sample data, row counts, and relationships were validated successfully.

## Related Files

- [`README.md`](README.md) — Lab overview and implementation steps.
- [`commands.md`](commands.md) — SQL commands used during implementation.
- [`screenshots/`](screenshots/) — Supporting evidence for this lab.

## Security Note

Passwords, private connection details, AWS account identifiers, and live database endpoints must not be published. Use a sanitized endpoint placeholder in commands and review screenshots before uploading them to GitHub.
