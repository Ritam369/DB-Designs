# Comic-Con Parking System - DB Schema Summary

## Entities & Attributes

| Entity | Primary Key | Key Attributes | Description |
|---|---|---|---|
| `VehicleType` | `vehicle_type_id` | `type_name` (enum: Bike, Car, SUV, Cab, EV), `description`, `created_at` | List of vehicle categories allowed in the parking system. Used to match vehicles to compatible parking spots. |
| `ReservationCategory` | `category_id` | `category_name` (enum: General, CosplayerWithProps, Exhibitor, Creator, VIPGuest, Staff, EVCharging), `description`, `is_special_reserved` | Defines parking privileges for different visitor types. Categories can apply to zones, spots, or tickets. |
| `Vehicle` | `vehicle_id` | `license_plate` (unique), `vehicle_type_id` (FK), `which_model`, `color`, `created_at` | Stores unique vehicle records that entered the Parking System identified by license plate. |
| `ParkingZone` | `zone_id` | `zone_name`, `description`, `total_levels`, `reservation_category_id` (FK, nullable) | Zones can be reserved for a specific category or remain general. |
| `ParkingSpot` | `spot_id` | `zone_id` (FK), `level`, `spot_number`, `vehicle_type_id` (FK), `reservation_category_id` (FK) | Individual parking slot with constraints. Spot can enforce both vehicle type and reservation category restrictions. |
| `ParkingTicket` | `ticket_id` | `ticket_number` (unique), `vehicle_id` (FK), `spot_id` (FK), `reservation_category_id` (FK), `entry_timestamp`, `issued_by` | Entry permit issued when a vehicle enters and occupies a spot. Captures which vehicle, spot, and category reservation used. |
| `ParkingSession` | `session_id` | `ticket_id` (FK, unique), `exit_timestamp`, `duration_minutes` | Represents the time span from entry to exit for a parked vehicle. Linked to ticket for billing calculations. |
| `Payment` | `payment_id` | `session_id` (FK), `amount`, `payment_timestamp`, `payment_method`, `status`('Pending','Paid','Failed','Refunded'), `transaction_ref` | Captures payment transaction for a parking session including amount, method, and status. |

## Table Relationships

| Parent Table | Child Table | Relationship | FK Column | Description |
|---|---|---|---|---|
| `VehicleType` | `Vehicle` | 1 : many | `Vehicle.vehicle_type_id` | One vehicle type category can include many registered vehicles. |
| `VehicleType` | `ParkingSpot` | 1 : many | `ParkingSpot.vehicle_type_id` | One vehicle type can have many parking spots designed for it. |
| `ReservationCategory` | `ParkingZone` | 1 : many | `ParkingZone.reservation_category_id` | One reservation category can be assigned to many zones (nullable; some zones are general). |
| `ReservationCategory` | `ParkingSpot` | 1 : many | `ParkingSpot.reservation_category_id` | One reservation category can apply to many individual spots. |
| `ReservationCategory` | `ParkingTicket` | 1 : many | `ParkingTicket.reservation_category_id` | One reservation category can be claimed in many parking tickets. |
| `ParkingZone` | `ParkingSpot` | 1 : many | `ParkingSpot.zone_id` | One parking zone contains many individual parking spots across levels. |
| `Vehicle` | `ParkingTicket` | 1 : many | `ParkingTicket.vehicle_id` | One vehicle can have multiple parking tickets over different visits/days. |
| `ParkingSpot` | `ParkingTicket` | 1 : many | `ParkingTicket.spot_id` | One parking spot can be used in many tickets at different times. |
| `ParkingTicket` | `ParkingSession` | 1 : 1 | `ParkingSession.ticket_id` | Each ticket has exactly one associated session (entry to exit). |
| `ParkingSession` | `Payment` | 1 : 1 | `Payment.session_id` | Each parking session has exactly one payment record. |

## Finding Available Parking Spots

To find the number of currently available parking spots:

```sql
SELECT COUNT(*) AS available_spots
FROM ParkingSpot ps
LEFT JOIN ParkingSession s
    ON ps.spot_id = s.spot_id
    AND s.exit_timestamp IS NULL
WHERE s.session_id IS NULL;
```

**Explanation:**
- Joins all `ParkingSpot` records with active `ParkingSession` records (where `exit_timestamp IS NULL`).
- A spot is available if no active session is associated with it (`s.session_id IS NULL`).
- Returns the count of unoccupied parking spots.
