# Multi-Agent Elevator Dispatch & Optimization Controller (C++)

[![Language: C++](https://img.shields.io/badge/Language-C%2B%2B-blue.svg)](https://isocpp.org/)
[![Focus: Multi-Agent Systems & Optimization](https://img.shields.io/badge/Focus-Heuristic%20Cost%20Scoring-darkgreen.svg)]()
[![Domain: Discrete-Event Simulation](https://img.shields.io/badge/Domain-Discrete--Event%20Simulation-orange.svg)]()

A discrete-event simulation and autonomous dispatch controller developed in C++ to model vertical transit and real-time passenger routing across a multi-floor, multi-car facility. The engine models dynamic request queues, elevator kinematic states, passenger wait penalties, and capacity constraints, utilizing a multi-factor heuristic cost function to orchestrate multi-agent dispatch decisions in real time.

---

> ### 🔒 Academic Integrity Notice
> This project was developed as part of **EECS 183 (Elementary Programming Concepts) at the University of Michigan**. In strict compliance with the University of Michigan College of Engineering Honor Code and course collaboration policies, all implementation source files (`Person.cpp`, `Elevator.cpp`, `Floor.cpp`, `Move.cpp`, `Building.cpp`, `Game.cpp`, `AI.cpp`) are maintained in a strictly private repository.
>
> System architecture, state machine progression, mathematical dispatch formulations, and object-oriented abstractions are documented below. Technical discussions and code reviews are available upon request for interview evaluations.

---

## ⚡ Engineering Highlights & Controller Architecture

* **Discrete-Event Simulation (DES) Loop:** Engineered a synchronous tick-based event processor updating passenger wait penalties, elevator travel states, and dynamic floor arrival queues per discrete time step.
* **Autonomous Decision Controller (`AI.cpp`):** Engineered a multi-factor heuristic scoring algorithm evaluating competing requests to maximize throughput and prevent passenger wait-time threshold penalties.
* **Multi-Constraint Queue Pruning:** Dynamically filters pickup candidate pools based on directional passenger travel alignment (unidirectional boarding constraints) and car capacity boundaries.
* **Persistent State Serialization:** Implemented file I/O routines enabling full system snapshots (elevator coordinates, remaining queue states, global wait penalties, and future event queues) to save and resume execution seamlessly.

---

## 🛠️ System Architecture & Class Hierarchy

The simulation separates physical infrastructure modeling from autonomous scheduling:

### 1. Object Model & Separation of Concerns
* **`Building`:** Manages spatial bounds (10 floors, 3 elevator shafts) and coordinates cascading `tick()` updates across all subordinate sub-components.
* **`Elevator`:** Encapsulates state transitions (`IDLE`, `MOVING`, `SERVICING`), passenger loading/unloading mechanics, and destination targeting.
* **`Floor` & `Person`:** Tracks individual passenger waiting times, target destination floors, and escalation metrics (wait penalties that increase monotonically with time).
* **`AI`:** Inspects global building telemetry on each unserviced car cycle to generate deterministic `Move` decisions without human intervention.

---

## 📐 Heuristic Decision Formulation

The core dispatch engine formulates car assignment as a real-time utility optimization problem. For every idle elevator $e$ and floor request candidate $f$, the controller computes a composite score $S(e, f)$ balancing urgency against travel overhead:

$$S(e, f) = \text{Anger}(f) - \lambda_{\text{dist}} \cdot \text{Distance}(e, f) + \lambda_{\text{crowd}} \cdot \text{Demand}(f)$$

Where:
* $\text{Anger}(f)$: The maximum accumulated waiting penalty of any passenger on floor $f$, ensuring starved or high-priority requests are prioritized before deadline expiration.
* $\text{Distance}(e, f) = | \text{Floor}_e - \text{Floor}_f |$: The spatial separation between elevator $e$ and destination floor $f$.
* $\text{Demand}(f)$: The cluster density (number of waiting passengers traveling in the dominant directional heading).
* $\lambda_{\text{dist}}, \lambda_{\text{crowd}}$: Tuned hyperparameter coefficients balancing transit delay against batch passenger throughput.

The controller executes an argmax assignment, selects the optimal subset of passengers conforming to directional alignment constraints, and dispatches the car.

---

## ⏱️ Algorithmic Complexity

| Subsystem / Operation | Mechanism | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| **Global Simulation Step** | Cascading `tick()` propagation | $O(F \cdot P + E)$ | $O(1)$ auxiliary |
| **Floor Queue Filtering** | Directional partitioning & boarding | $O(P)$ | $O(P)$ |
| **Heuristic Dispatch Search** | State evaluation across idle units | $O(E_{\text{idle}} \cdot F)$ | $O(1)$ |
| **State Snapshot / Restore** | Serialized file stream extraction | $O(F \cdot P + E)$ | $O(1)$ auxiliary |

*Where $F$ is number of floors (10), $E$ is number of elevators (3), and $P$ is total waiting passengers across floors.*

---

## 💻 Tech Stack & Tooling

* **Language Standard:** C++ (C++11 / C++14)
* **Standard Library:** Streams (`<iostream>`, `<fstream>`, `<sstream>`), Algorithms (`<algorithm>`, `<cmath>`)
* **Development & Build:** Visual Studio / GNU `g++`, modular compilation
* **Validation:** Custom unit-test modules (`test.cpp`) asserting invariant bounds on passenger time steps, directional boarding logic, and multi-turn regression cases.

---

## 📈 Engineering Learnings & Systems Takeaways

* **Starvation vs. Throughput Trade-offs:** Pure greedy dispatch strategies based solely on distance lead to request starvation on distant upper floors; incorporating wait penalties as a weighted factor guarantees long-waiting requests are addressed.
* **Encapsulated State Propagation:** Designing a clean decoupled hierarchical architecture ensures individual entities (`Person`, `Floor`, `Elevator`) can be unit-tested in complete isolation prior to integration testing.
* **Deterministic Discrete Simulation:** Enforcing reproducible event pipelines ensures edge-case routing bottlenecks can be consistently reproduced, debugged, and optimized.
