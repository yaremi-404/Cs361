## Technical Design Document  
_Yaremi Dominguez_  
_November 2, 2026_  

**Part 1**  
_Overview_  
The bike shop software system is meant to be used by customers and employees for making reservations by viewing available inventory, and by management to view reports, manage reservations and payments, and manage inventory. The customers can also make reservations through the kiosk.

_Actors_
* Customer: creates reservation, and pays
* Bank Authotization Service: to verify correct account credentials, and approve the amount/cancel amount
* Employee: create/manage reservations
* Kiosk: used to make reservations, view inventory andavailability
* Manager: Pull reports, manage inventory, manage reservations
* Bike maintenance:keep real-time info on bikes available to bike shop  

_Goals_
* Allow customersto view inventory
* Allow reservations
* Have secure payment processing
* Allow kiosk reservations
* Allow employee access to reservations
* Real-time bike inventory and reservations  

_Non-goals_
* We are not focusing on a charge process for unreturned bikes
* We are not including outreach ads
* We are not focusing on how to connect the bike maintenance to the bike shop
* We are not focusing on employee scheduling
* we are not including rewards program    
  
_constraints_  
* Kiosk interface is the same backend api as the website  
* inventory is real-time
* No overlapping time slots
* User data handling must comply with CCPA and GDPR, including the ability to export and delete a customer's data on request.
* Employees and manager must authenticate using the shop’s existing login system.  

_Assumptions_  
* There will be no connectivity issues when creating reservations
* peak trafficis during the weekends
* bike inventory changes infrequently
* bike shop prices change infrequently
* Customers return the bikes on time, unless otherwise communicated
* most reservations are made online

_Non-Functional Recs_
* inventory updates need to reflect online within 1 minute of changes
* The reservation process is no more than 4 steps
* Must be able to handle 100 requests at the same time
* Employees and manager need authenticated access
* Bike innventory additions/removals shouldbe possible without redesigning to database
* Kiosk meets accessability guidelines  

_Component Diagram and Description_
```mermaid
flowchart TD
    A[Customer] --> |Goes to|C(Website)
    A[Customer] --> |Goes to|B(Kiosk)
    B --> D[Bike Inventory]
    C --> D[Bike Inventory]
    D --> |Add to cart |E( Cart)
    E --> F[Review]
    F[Review] --> |Accept| G(payment)
    F[Review] --> |Go back| D[Bike Inventory]
    G(payment) --> |finalize| H(reciept page)  
```
|Component| Description |
|---|---|
|**Kiosk**|Shows all available bikes, allows user to create reservation in store.
|**Website**|Shows all available bikes, allows user to create reservation anywhere anytime.
|**Bike Inventory**|real-time bike availability
|**Cart**|users specific cart with bikes they want to rent.
|**Review**|Before paying,users see this screen to confirm their reservation.
|**Payment**|This processes user payment.
|**Receipt Page**|This is the last page a user sees and it has confirmation the payment went through and the successfully created a reservation.  


**Part 2**   
_Entity Relationship Diagram_  
```mermaid
erDiagram
    CUSTOMER||--|{Reservation : Uses
    Reservation||--|{ Bike: has
    Employee ||--|{ Reservation : manages

    CUSTOMER {
        string customerId
        string name
        string PhoneNum
    }
    Reservation {
        float payment 
        date Date
        string reservationId
    }
    Employee {
        string EmpId
        string name
    }
    Bike {
        string id
        string status
        string Biketype
    }
```


_Class Diagram_
```mermaid
classDiagram
    Bike <|-- Reservation
    Reservation <|-- Customer
    Reservation <|-- Employee
    Bike : +float price
    Bike : +String status
    Bike : +String BikeId
    Bike : +String availability

    class Reservation{
      +String ResId
      +String status

    }
    class Customer{
      +String ResId
      +String name
    }
    class Employee{
    +String EmpId
    +String Name

    }
```


_Interfaces_  
1.  
Purpose:Customer searches for a type of bike
Requires: bikelist/biketype
Outcomes: matching bike search shows up
Guarantees: matching bikes
function: something like below
List SearchBike{
    String bikeType,
}

2.  
Purpose:create reservation
Requires: resid, customerid, bike, paymentid
Outcomes: a complete reservation
Guarantees: a specififc reservation for specific customer, andavailable bike
Code: something like below
reservation createreservation{
    Bike bike
    string customerid
    string resid
    string paymentid
}

3.  
Purpose: calculate cost of reservation
Requires: bikes, reslength
Outcomes: price shown to customer at checkout
Guarantees: a accurate price per bike and for length of reservation
Code:
reservation calculatecost{
    string bikes
    int hours
    int price
}

4.  
Purpose:Manager can generate a report
Requires: Date of report 
Outcomes: Report of specific day
Guarantees: accurate bike, and reservation count
Notes: The report prints out the selected day info
Code:
Report DailyReport(day){
    reservation count
    payments
    bike count
}

5.  
Purpose:Allows editing of reservation after its been placed
Requires: resid, bike, price
Outcomes: employee can add/remove bikes
Guarantees: up to date reservation
Code:
reservation update{
    string reservationid
    bike bike
    int price
}

6.  
Purpose:viewers can view info on bikes
Requires: biketype, 
Outcomes: accurate information on certain bikes
Guarantees: information on specific bike
Code:
List searchbike{
    bike bike
    string Biketype
}

_Invariants_   
1. Bike availability is real-time  
2. reservations have their own unique id per customer
3. Employees have authorized access  
4. Every  bike has a bike unique bike id 

_Failure Modes_  
1. reservation didnt save, no reseravtionid  
2. Two customers trying to reserve the same bike at the same time
3. payments dont process  at checkout
4. unavailable bikes shouldnt be able to be reserved



