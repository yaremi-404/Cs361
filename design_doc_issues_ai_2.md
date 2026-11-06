# Design Document Review

## Critical Issues

### 1. Entity Relationship Diagram Uses Actors Instead of Entities
The ER diagram should model data entities, not system actors.

Current entities:
- CUSTOMER
- Kiosk
- Employee
- Manager

Problems:
- Manager appears to be a specialized type of Employee, not a separate entity with only an ID.
- Kiosk is a system interface/device, not business data.
- The actual core business object (Bike) is missing.

Recommended core entities:
- Customer
- Reservation
- Bike
- Employee
- Payment

### 2. ER Diagram Is Missing Reservation Entity
Reservations appear throughout the interfaces and invariants but do not exist in the ER diagram.

This is a major inconsistency because:
- Reservations have IDs.
- Reservations are created and updated.
- Reservations are used in reports.

A Reservation entity should exist and connect:
- Customer → Reservation
- Reservation → Bike
- Reservation → Payment

### 3. Class Diagram Uses Inheritance Incorrectly
Current diagram:

```text
Bike <|-- Reservation
Reservation <|-- Customer
Reservation <|-- Employee
```

This means:
- Reservation IS A Bike
- Customer IS A Reservation
- Employee IS A Reservation

These relationships are not logically correct.

Inheritance should only be used when the child object is a specialized form of the parent.

Examples of valid inheritance:
- Employee <|-- Manager
- Bike <|-- MountainBike
- Bike <|-- RoadBike

Your relationships should likely be associations instead:

```text
Customer --> Reservation
Reservation --> Bike
Employee --> Reservation
```

### 4. Bike Entity Missing from ER Diagram
Bike exists in:
- Class Diagram
- Interfaces
- Invariants
- Failure Modes

But Bike does not exist in the ER Diagram.

This creates a major mismatch between system design sections.

### 5. Payment Entity Referenced but Never Defined
Payment IDs appear in:

- Create Reservation
- Daily Report

However there is no:
- Payment entity
- Payment class
- Payment attributes

The design references data that does not exist in the model.

---

## Major Issues

### 6. Customer Contains Reservation ID
Current:

```text
Customer
- reservationId
- name
- phoneNum
```

Problem:

A customer may have many reservations.

Storing a reservation ID in Customer implies:
- One customer = one reservation

Recommended:

```text
Customer
- customerId
- name
- phoneNum
```

Reservations should reference customers instead.

### 7. Kiosk Attributes Do Not Match a Kiosk
Current:

```text
Kiosk
- price
- orderDate
- catalog
```

These look like reservation/order attributes, not kiosk attributes.

A kiosk might contain:

```text
kioskId
location
status
```

### 8. Manager Has Only One Attribute
Current:

```text
Manager
- id
```

This entity contributes almost no information.

Either:
- Make Manager inherit from Employee, or
- Remove the Manager entity entirely and add a role field.

Example:

```text
Employee
- empId
- name
- role
```

### 9. Employee-Manager Relationship Is Unclear
Current:

```text
Employee ||--|{ Manager : Works
```

This suggests:
- One Employee works with many Managers

The meaning is unclear.

More likely:

```text
Manager supervises Employee
```

or

```text
Employee isa Manager
```

depending on system requirements.

### 10. No Relationship Between Customers and Reservations
The interfaces clearly indicate:

```text
Customer creates Reservation
```

but the diagrams do not show this relationship.

---

## Class Diagram Issues

### 11. Reservation Missing Core Attributes

Current:

```text
Reservation
- ResId
- status
```

Likely missing:

```text
Reservation
- reservationId
- reservationDate
- startTime
- endTime
- status
- totalCost
```

### 12. Customer Missing Customer ID

Current:

```text
Customer
- ResId
- name
```

Should be:

```text
Customer
- customerId
- name
- phoneNumber
```

### 13. Duplicate Availability Information

Bike currently has:

```text
status
availability
```

These may represent the same concept.

Example:

```text
Available
Reserved
OutOfService
```

One field is likely sufficient.

### 14. Bike Type Is Missing

Your interfaces repeatedly search by bike type:

```text
searchBike
bikeType
```

But Bike does not contain a bikeType field.

Need:

```text
Bike
- bikeId
- bikeType
- price
- status
```

---

## Interface Issues

### 15. Duplicate Search Bike Interface

You have:

#### Interface 1

```text
SearchBike
```

and

#### Interface 6

```text
searchBike
```

Both perform nearly the same function.

These should be combined.

### 16. Calculate Cost Returns a Reservation

Current:

```text
reservation calculatecost
```

A cost calculator should return:

```text
float
decimal
money
```

not an entire Reservation.

### 17. Update Reservation Missing Who Can Update

Current statement:

```text
employee and customer can edit
```

Need explicit rules:

- What can customers edit?
- What can employees edit?
- Is there a deadline?
- Can bikes already checked out be changed?

### 18. SearchBike Parameters Are Incorrect

Current:

```text
List SearchBike{
    Bike bike
}
```

Searching generally requires criteria:

```text
SearchBike(
    String bikeType
)
```

not a full Bike object.

### 19. Daily Report Structure Is Incomplete

Current:

```text
Report DailyReport(day)
```

The report object returns:

```text
resid
paymentid
bike
```

But the description promises:

```text
reservation count
bike count
```

Returned data does not match documented behavior.

---

## Invariant Issues

### 20. Invariant #1 Is Not Really an Invariant

Current:

```text
Bike availability is real-time
```

This is more of a system requirement.

A better invariant:

```text
A bike may not be assigned to more than one active reservation at a time.
```

### 21. Missing Reservation-Bike Constraint

A key business rule is absent:

```text
Reserved bikes must be available before being assigned to a reservation.
```

### 22. Missing Customer Constraint

Potential invariant:

```text
Every reservation must be associated with exactly one customer.
```

---

## Failure Mode Issues

### 23. Failure Modes Do Not Describe Handling

Current examples only identify failures:

```text
payments dont process
reservation didnt save
```

Need expected behavior:

Example:

```text
Failure:
Payment processing fails

System Response:
Display error message and prevent reservation creation.
```

### 24. Missing Concurrency Failure

Important bike rental issue:

```text
Two customers attempt to reserve the same bike simultaneously.
```

Your design does not address this.

### 25. Missing Authentication Failure

Employees and managers have different privileges, but there is no failure mode for:

```text
Unauthorized user attempts restricted action.
```

---

## Structural Issues

### 26. Naming Conventions Are Inconsistent

Examples:

```text
reservationId
ResId
reservationid
resid
BikeId
bike
Biketype
bikeType
```

Use a consistent naming style.

Example:

```text
customerId
reservationId
bikeId
employeeId
paymentId
```

### 27. Case Usage Is Inconsistent

Examples:

```text
PhoneNum
Name
bike
Bike
Biketype
```

Choose one convention:

```text
camelCase
```

or

```text
PascalCase
```

and apply it everywhere.

### 28. Missing System Overview

The document jumps directly into diagrams.

A design document should usually include:

```text
Purpose
Scope
Actors
System Description
Assumptions
```

before technical diagrams.

---

# Summary

The biggest problems are:

1. Reservation is missing from the ER diagram.
2. Bike is missing from the ER diagram.
3. Payment is referenced but never modeled.
4. Class diagram uses inheritance incorrectly.
5. Customer stores reservation IDs.
6. Kiosk is modeled as a data entity when it behaves like a system interface.
7. Duplicate SearchBike interfaces exist.
8. Relationships between Customer, Reservation, Bike, and Employee are not represented correctly.

Overall, the project concept is solid for a bike rental/reservation system, but the ER diagram and class diagram need significant restructuring before implementation.