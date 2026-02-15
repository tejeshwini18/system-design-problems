# LLD: Elevator System

## 1. Requirements

- **Building** with N **floors** and M **elevators**.
- **Elevator:** Current floor, direction (UP/DOWN/IDLE), state (IDLE, MOVING, MAINTENANCE), list of **pending requests** (floor + direction).
- **Request:** User presses button (floor, direction: UP/DOWN); or user inside selects target floor.
- **Scheduling:** Assign request to an elevator (SCAN, nearest car, or direction-based); elevator serves requests in order (e.g. same direction first).
- **Movement:** Elevator moves one floor at a time; stops at requested floors; opens door; closes; continues.

---

## 2. Core Classes

```text
Elevator
  - id, currentFloor, direction (UP/DOWN/IDLE), state (IDLE/MOVING/MAINTENANCE)
  - pendingStops: Set or Queue (floors to stop at)
  - addRequest(floor, direction)   // add to pending
  - move()   // one floor per tick; stop if currentFloor in pendingStops
  - openDoor(), closeDoor()

ElevatorController
  - elevators: List<Elevator>
  - requestElevator(floor, direction) → assign to best elevator (e.g. nearest, same direction)
  - submitRequest(elevatorId, targetFloor)   // from inside elevator

Building
  - floors: int; elevators: List<Elevator>
  - getController() → ElevatorController
```

---

## 3. Scheduling (Simple Strategy)

- **Nearest car:** For request (floor, direction), pick elevator with min distance (consider direction: elevator moving toward floor and will pass it is better).
- **Direction-based:** Prefer elevator that is IDLE or moving in same direction and will pass the request floor; else nearest IDLE.
- **SCAN:** Elevator keeps moving in one direction, serving all requests in that direction, then reverses (like disk scheduling).

---

## 4. State Transitions (Elevator)

- **IDLE** → request added → **MOVING** (set direction).
- **MOVING** → reached floor in pendingStops → stop, **openDoor**, wait (e.g. 2s), **closeDoor**, remove from pending; if no more pending → **IDLE**.
- **MAINTENANCE** → manual; no new requests assigned.

---

## 5. Request Types

- **External:** (floor, direction) – user on floor wants to go UP or DOWN. Controller assigns elevator; elevator adds floor to pending (and direction for future).
- **Internal:** (elevatorId, targetFloor) – user inside selects floor. Add targetFloor to that elevator’s pendingStops.

---

## 6. Design Patterns

- **State:** ElevatorState (IdleState, MovingState, MaintenanceState); behavior (move, addRequest) depends on state.
- **Strategy:** SchedulingStrategy (NearestCarStrategy, SCANStrategy).
- **Singleton:** Building or Controller if single building.

---

## 7. Key Methods

```text
ElevatorController.requestElevator(floor, direction) → elevatorId
Elevator.addRequest(floor) or addRequest(targetFloor)
Elevator.move()   // called by scheduler every 1s
Elevator.getCurrentFloor(), getDirection(), getState()
```
