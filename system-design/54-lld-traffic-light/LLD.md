# LLD: Traffic Light System

## 1. Requirements

- **Intersection:** Multiple **lights** (e.g. N-S, E-W); each light has **states**: RED, YELLOW, GREEN.
- **Cycle:** GREEN → YELLOW (brief) → RED → (other direction GREEN) → …; fixed duration per state or configurable.
- **Controller:** Switches state at fixed intervals (e.g. GREEN 30s, YELLOW 3s, RED 33s for other); optional **sensor** (pedestrian button, vehicle sensor) to extend or request.
- **Optional:** Multiple intersections coordinated; pedestrian crossing phase.

---

## 2. Core Classes (State Pattern)

```text
TrafficLight – id, direction (e.g. "N-S"); currentState: LightState
LightState (interface) – durationSeconds; nextState() → LightState
  GreenState, YellowState, RedState implement LightState

TrafficController – lights: List<TrafficLight>; run() loop: for each light, tick(); if elapsed >= state.duration, light.setState(state.nextState())
  Ensures: only one direction GREEN at a time (or define pairs: N-S green implies E-W red)
```

---

## 3. State Transitions

- **N-S:** GREEN (30s) → YELLOW (3s) → RED. When N-S goes RED, E-W goes GREEN (30s) → YELLOW (3s) → RED → N-S GREEN.
- **Model:** Two lights; Light1: GreenState → YellowState → RedState → (then Light2 turns Green) → back to GreenState. Or single controller with phase: PHASE_NS_GREEN, PHASE_NS_YELLOW, PHASE_EW_GREEN, PHASE_EW_YELLOW; each phase has duration; cycle through.

---

## 4. Key Methods

```text
TrafficLight.setState(state), getState(), tick() or getElapsed()
TrafficController.run()   // main loop: sleep(1); update all; check transition
LightState.getDuration(), nextState()
```

---

## 5. Design Patterns

- **State:** LightState (Green, Yellow, Red); behavior and next state defined per state.
- **Observer:** Optional: notify (UI or sensors) on state change.
