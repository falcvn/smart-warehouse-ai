# Smart Warehouse Delivery Robot — AI Decision System

**Author:** Asanali Kuandyk (Qazaq AI Research University)  
**Course:** AI Fundamentals (AIF1303) — Final Project  
**Implementation:** Standalone Interactive Web Simulation (HTML5 Canvas, Vanilla JavaScript, CSS3)

## Overview
An autonomous multi-robot warehouse fulfillment simulator that integrates three core Artificial Intelligence paradigms to navigate a 16×12 warehouse grid, filter noisy sensor readings, and avoid dynamic obstacles in real time.

## Implemented AI Modules
1. **Informed vs. Uninformed Search (`AStarPathfinder` — Week 2):**
   - **A\* Search** using admissible Manhattan distance heuristic `f(n) = g(n) + h(n)` compared directly against **Breadth-First Search (BFS)**.
   - Supports real-time path recalculation when dynamic obstacles (*Spilled Boxes*) are placed on active routes.
2. **Probabilistic Reasoning (`BayesianSensorModel` — Week 6):**
   - Updates posterior obstacle belief `P(Obstacle | Sensor)` via Bayes' Theorem at noisy corridor checkpoints (`S1–S4`), accounting for false alarms `P(BLOCKED | ¬O) = 0.20` and sensitivity `P(BLOCKED | O) = 0.80`.
3. **Sequential Decision Making (`MDPValueIterationSolver` — Week 5):**
   - Solves a Markov Decision Process using **Value Iteration** (`γ = 0.90`, `θ = 0.001`) to weigh collision risk against detour cost and choose the optimal action (`PROCEED` vs. `DETOUR`).

## Benchmark Results (10-Run Average)
| Metric | A* + Bayes + MDP | BFS + Naive Sensor | Improvement |
| :--- | :---: | :---: | :---: |
| **Nodes Explored** | 455.6 | 1444.7 | **-68.5%** |
| **Travel Distance** | 182.2 | 166.2 | +9.6% (safe detour overhead) |
| **Avg Collisions** | 0.1 | 0.1 | Controlled risk |

## How to Run
No installation or server required. Download `index.html` and open it in any modern web browser (Chrome, Safari, Firefox, Edge), or open the live GitHub Pages deployment.

## Code Architecture
- `Grid` — 16×12 warehouse map state, shelf racks, charging pads, drop-off bays, sensor checkpoints, and dynamic obstacle toggling.
- `AStarPathfinder` — A* (Manhattan heuristic) and BFS shortest-path solvers.
- `BayesianSensorModel` — Posterior probability updates for noisy binary sensor signals.
- `MDPValueIterationSolver` — Bellman optimality solver for `PROCEED` vs. `DETOUR` policy selection.
- `WarehouseSim` — Main simulation loop, Canvas renderer, live telemetry, and 10-run automated benchmark suite.
