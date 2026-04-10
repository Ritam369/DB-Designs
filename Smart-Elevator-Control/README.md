# Smart Elevator Control - DB Schema Summary

## Entities & Attributes

| Entity | Primary Key | Key Attributes | Description |
|---|---|---|---|
| `Building` | `building_id` | `name`, `address`, `city`, `total_floors`, `created_at` | Building schema for which the elevator system is designed. It acts as the parent for zones, floors, and shafts. |
| `Zone` | `zone_id` | `building_id` (FK), `name` (`Low Zone`, `Mid Zone`, `High Zone`), `description` | Groups elevators within a building by zone. Useful for splitting traffic across different parts of the building. |
| `Floor` | `floor_id` | `building_id` (FK), `floor_number`, `floor_name` (`Ground`, `Mezzanine`, `Terrace`) | Represents the floors available in a building. |
| `ElevatorShaft` | `shaft_id` | `building_id` (FK), `shaft_number` (`Shaft A`, `Shaft B`) | Tracks elevator shaft locations inside a building. Each shaft can contain a dedicated elevator. |
| `Elevator` | `elevator_id` | `shaft_id` (FK), `zone_id` (FK), `model`, `capacity_kg`, `max_passengers`, `installation_date`, `updated_at` | Stores elevator details, capacity, and other props. It is the core asset used for transporting passengers between floors. |
| `ElevatorFloorService` | `(elevator_id, floor_id)` | `elevator_id` (FK), `floor_id` (FK) | Junction table that defines which floors are served by which elevators. Supports composite Primary key |
| `FloorRequest` | `request_id` | `origin_floor_id` (FK), `destination_floor_id` (FK), `request_timestamp`, `status`, `request_type` | Captures a passenger’s request to move from one floor to another. It tracks the request lifecycle from pending to completed or cancelled. |
| `RideAssignment` | `assignment_id` | `request_id` (FK), `elevator_id` (FK), `assigned_timestamp`, `assignment_status` | Stores which elevator has been assigned to a floor request. Also supports reassignment and completion tracking. |
| `RideLog` | `log_id` | `assignment_id` (FK), `elevator_id` (FK), `origin_floor_id` (FK), `destination_floor_id` (FK), `start_time`, `end_time`, `passenger_count`, `load_kg` | Keeps complete trip history for analytics and operational review. |
| `ElevatorStatus` | `status_id` | `elevator_id` (FK), `current_status`, `current_floor`, `current_load_kg`, `current_passenger_count`, `last_weight_update_timestamp`, `status_timestamp` | Stores live elevator's state. Used for monitoring movement, load, and maintenance conditions. |
| `MaintenanceRecord` | `maintenance_id` | `elevator_id` (FK), `start_date`, `end_date`, `maintenance_type`, `description`, `status`, `closed_at` | Records scheduled or reactive maintenance work for each elevator. |

## Table Relationships

| Parent Table | Child Table | Relationship | FK Column | Description |
|---|---|---|---|---|
| `Building` | `Zone` | 1 : many | `Zone.building_id` | One building can contain multiple zones. |
| `Building` | `Floor` | 1 : many | `Floor.building_id` | One building can have many floors. |
| `Building` | `ElevatorShaft` | 1 : many | `ElevatorShaft.building_id` | One building can contain multiple elevator shafts. |
| `Zone` | `Elevator` | 1 : many | `Elevator.zone_id` | One zone can group many elevators. |
| `ElevatorShaft` | `Elevator` | 1 : 1 | `Elevator.shaft_id` | Each shaft is linked to one elevator in this design. |
| `Elevator` | `ElevatorFloorService` | 1 : many | `ElevatorFloorService.elevator_id` | One elevator can serve many floors. |
| `Floor` | `ElevatorFloorService` | 1 : many | `ElevatorFloorService.floor_id` | One floor can be served by many elevators. |
| `FloorRequest` | `RideAssignment` | 1 : many | `RideAssignment.request_id` | One floor-request can be assigned more than once if reassignment is needed. |
| `RideAssignment` | `RideLog` | 1 : 1 | `RideLog.assignment_id` | Each completed ride has exactly one ride log. |
| `Elevator` | `RideLog` | 1 : many | `RideLog.elevator_id` | One elevator can appear in many ride logs over time. |
| `Elevator` | `ElevatorStatus` | 1 : many | `ElevatorStatus.elevator_id` | One elevator can have many status snapshots. |
| `Elevator` | `MaintenanceRecord` | 1 : many | `MaintenanceRecord.elevator_id` | One elevator can have many maintenance records. |

