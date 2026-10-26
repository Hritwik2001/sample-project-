# MediTrack --- Hospital Management Database

MediTrack is a MySQL database project for managing patients, doctors,
appointments, prescriptions, and billing information. It demonstrates
relational database design, primary and foreign keys, joins, aggregate
queries, indexes, and views.

## Table of Contents

-   [Overview](#overview)
-   [Features](#features)
-   [Database Schema](#database-schema)
-   [ER Diagram](#er-diagram)
-   [Relationships](#relationships)
-   [Requirements](#requirements)
-   [Setup](#setup)
-   [Example Queries](#example-queries)
-   [Database Concepts Demonstrated](#database-concepts-demonstrated)

## Overview

The database stores patient details, doctor specializations and
consultation fees, appointment records, prescriptions issued during
appointments, and billing/payment details.

**Database name:** `MediTrack`\
**Database system:** MySQL

## Features

-   Store and manage patient information.
-   Maintain doctor details, specializations, fees, and experience.
-   Record appointments between patients and doctors.
-   Store prescriptions linked to appointments.
-   Track bills, payment methods, and payment status.
-   Query appointment counts, average doctor fees, and revenue
    summaries.
-   Use indexes to support lookups and views for common billing reports.

## Database Schema

### 1. `Patients`

  Column         Type           Key                           Description
  -------------- -------------- ----------------------------- ---------------------------
  `Patient_ID`   INT            Primary Key                   Unique patient identifier
  `Name`         VARCHAR(100)                                 Patient name
  `Age`          INT                                          Patient age
  `Gender`       VARCHAR(10)                                  Patient gender
  `Contact`      VARCHAR(15)                                  Contact number
  `Email`        VARCHAR(100)   Unique index in the project   Patient email
  `Address`      VARCHAR(100)                                 Patient address

### 2. `Doctors`

  Column               Type            Key           Description
  -------------------- --------------- ------------- --------------------------
  `Doctor_ID`          INT             Primary Key   Unique doctor identifier
  `Name`               VARCHAR(100)                  Doctor name
  `Specialization`     VARCHAR(100)                  Medical specialization
  `Fee`                DECIMAL(10,2)                 Consultation fee
  `Experience_Years`   INT                           Years of experience

### 3. `Appointments`

  --------------------------------------------------------------------------------
  Column               Type              Key                     Description
  -------------------- ----------------- ----------------------- -----------------
  `Appointment_ID`     INT               Primary Key             Unique
                                                                 appointment
                                                                 identifier

  `Patient_ID`         INT               Foreign Key →           Patient for the
                                         `Patients.Patient_ID`   appointment

  `Doctor_ID`          INT               Foreign Key →           Doctor assigned
                                         `Doctors.Doctor_ID`     to the
                                                                 appointment

  `Appointment_Date`   DATE                                      Appointment date

  `Status`             VARCHAR(20)                               Appointment
                                                                 status
  --------------------------------------------------------------------------------

### 4. `Prescriptions`

  ---------------------------------------------------------------------------------------
  Column              Type              Key                             Description
  ------------------- ----------------- ------------------------------- -----------------
  `Prescription_ID`   INT               Primary Key                     Unique
                                                                        prescription
                                                                        identifier

  `Appointment_ID`    INT               Foreign Key →                   Appointment
                                        `Appointments.Appointment_ID`   associated with
                                                                        the prescription

  `Medicine`          VARCHAR(100)                                      Medicine name

  `Dosage`            VARCHAR(20)                                       Prescribed dosage

  `Duration_Days`     INT                                               Duration of
                                                                        medication in
                                                                        days
  ---------------------------------------------------------------------------------------

### 5. `Billing`

  --------------------------------------------------------------------------------------
  Column             Type              Key                             Description
  ------------------ ----------------- ------------------------------- -----------------
  `Bill_ID`          INT               Primary Key                     Unique bill
                                                                       identifier

  `Appointment_ID`   INT               Foreign Key →                   Appointment being
                                       `Appointments.Appointment_ID`   billed

  `Amount`           DECIMAL(10,2)                                     Bill amount

  `Payment_Method`   VARCHAR(50)                                       Cash, UPI, debit
                                                                       card, credit
                                                                       card, etc.

  `Payment_Status`   VARCHAR(20)                                       Payment status
  --------------------------------------------------------------------------------------

## ER Diagram

The following Mermaid ER diagram can be rendered by GitHub and other
Markdown viewers that support Mermaid.

``` mermaid
erDiagram
    PATIENTS ||--o{ APPOINTMENTS : books
    DOCTORS ||--o{ APPOINTMENTS : attends
    APPOINTMENTS ||--o{ PRESCRIPTIONS : has
    APPOINTMENTS ||--o| BILLING : generates

    PATIENTS {
        int Patient_ID PK
        varchar Name
        int Age
        varchar Gender
        varchar Contact
        varchar Email
        varchar Address
    }

    DOCTORS {
        int Doctor_ID PK
        varchar Name
        varchar Specialization
        decimal Fee
        int Experience_Years
    }

    APPOINTMENTS {
        int Appointment_ID PK
        int Patient_ID FK
        int Doctor_ID FK
        date Appointment_Date
        varchar Status
    }

    PRESCRIPTIONS {
        int Prescription_ID PK
        int Appointment_ID FK
        varchar Medicine
        varchar Dosage
        int Duration_Days
    }

    BILLING {
        int Bill_ID PK
        int Appointment_ID FK
        decimal Amount
        varchar Payment_Method
        varchar Payment_Status
    }
```

**Note:** The diagram shows the intended business relationships. The
database schema uses a foreign key from `Billing.Appointment_ID` to
`Appointments.Appointment_ID`. Unless `Appointment_ID` is also declared
`UNIQUE` in `Billing`, MySQL permits multiple billing rows for one
appointment; the `||--o|` notation represents the common intended
one-bill-per-appointment design. Use `||--o{` instead if multiple bills
per appointment are allowed.

## Relationships

1.  **Patients → Appointments:** One patient can have many appointments;
    each appointment references one patient.
2.  **Doctors → Appointments:** One doctor can attend many appointments;
    each appointment references one doctor.
3.  **Appointments → Prescriptions:** One appointment can have multiple
    prescription records.
4.  **Appointments → Billing:** Billing records reference an
    appointment. The current foreign key alone does not enforce a
    maximum of one bill per appointment.

## Requirements

-   MySQL 8.0 or later (or a compatible MySQL version)
-   MySQL Workbench or MySQL command-line client (optional, for running
    and inspecting SQL)

## Setup

1.  Open MySQL Workbench or the MySQL command-line client.

2.  Create and select the database:

    ``` sql
    CREATE DATABASE MediTrack;
    USE MediTrack;
    ```

3.  Create the tables in this order so that referenced tables exist
    first:

    `Patients` → `Doctors` → `Appointments` → `Prescriptions` →
    `Billing`

4.  Run the table-creation SQL from your project script.

5.  Insert sample records after creating the tables.

6.  If your SQL script contains MySQL client prompts or query output
    (for example, `mysql>` or `Query OK`), remove those lines before
    executing it. Run only SQL statements.

## Example Queries

### List appointments with patient and doctor names

``` sql
SELECT
    a.Appointment_ID,
    p.Name AS Patient_Name,
    d.Name AS Doctor_Name,
    a.Appointment_Date,
    a.Status
FROM Appointments AS a
JOIN Patients AS p ON p.Patient_ID = a.Patient_ID
JOIN Doctors AS d ON d.Doctor_ID = a.Doctor_ID;
```

### Count appointments per patient

``` sql
SELECT
    Patient_ID,
    COUNT(*) AS Total_Appointments
FROM Appointments
GROUP BY Patient_ID;
```

### Calculate average consultation fee by specialization

``` sql
SELECT
    Specialization,
    AVG(Fee) AS Avg_Fee
FROM Doctors
GROUP BY Specialization;
```

### Find bills above ₹700

``` sql
SELECT *
FROM Billing
WHERE Amount > 700;
```

### Summarize revenue by payment method

``` sql
SELECT
    Payment_Method,
    SUM(Amount) AS Total_Revenue
FROM Billing
GROUP BY Payment_Method;
```

### Find medicines prescribed to a specific patient

``` sql
SELECT pr.Medicine, pr.Dosage, pr.Duration_Days
FROM Prescriptions AS pr
JOIN Appointments AS a
    ON a.Appointment_ID = pr.Appointment_ID
JOIN Patients AS p
    ON p.Patient_ID = a.Patient_ID
WHERE p.Name = 'Rohit Sharma';
```

## Database Concepts Demonstrated

-   Relational database design
-   Primary keys and foreign keys
-   One-to-many relationships
-   Joins across related tables
-   Aggregate functions: `COUNT()`, `AVG()`, and `SUM()`
-   `GROUP BY` and `HAVING`
-   Subqueries
-   Indexes, including a unique index on patient email
-   Views for reusable reports, such as high-value bills and revenue
    summaries

## Future Improvements

-   Add `NOT NULL`, `CHECK`, and status constraints where appropriate.
-   Make `Billing.Appointment_ID` unique if the system must allow only
    one bill per appointment.
-   Add doctor availability and appointment-conflict checks.
-   Add users/roles and audit logging for administrative operations.
-   Store prescription instructions such as frequency separately if
    needed.
-   Add automated backup and data privacy controls.

## Author

**Hritwik Kumar Roy**
