# Smart Warehouse Delivery Robot — AI Decision System

**Asanali Kuandyk** · Qazaq AI Research University · AI Fundamentals (AIF1303) final project

Three robots collect parcels and deliver them across a 16 × 12 warehouse. Each robot plans a route around shelves, estimates the risk at noisy sensor checkpoints, and decides whether to proceed or take a detour.

## Run locally

Download the project from [falcvn/smart-warehouse-ai](https://github.com/falcvn/smart-warehouse-ai) and unzip it, or use your local checkout. Double-click `index.html` to open the simulation in a modern browser. The page runs offline with all JavaScript, CSS, and Canvas drawing code in one file.

The published version is available on [GitHub Pages](https://falcvn.github.io/smart-warehouse-ai/).

## Controls

| Control | What it does |
| --- | --- |
| Pause / Resume, Simulation Speed | Stop or resume live movement, or change its speed from 0.5× to 4×. |
| Reset Warehouse, Spawn 5 Random Orders | Reset the map, robots, and counters with five starting orders, or add up to five more at available pickups. Spawning also refreshes checkpoint hazards. |
| Pathfinding and policy toggles | Choose A* or BFS for route planning. Choose Bayes + MDP or Naive Sensor for checkpoint decisions. |
| Prior, Sensitivity, False Alarm, Discount Factor | Set the obstacle probability, the chance of a BLOCKED reading with and without an obstacle, and the weight given to future rewards. The calibration panel shows the decision for each possible reading. |
| Warehouse map | Click an available aisle cell to add or remove a spilled box. If a box blocks an active route, the robot replans. |
| Run Benchmark Test (10 Runs) | Compare both preset configurations using the current model parameters and a fresh benchmark map. |
| Clear | Clear the AI decision log. |

## Three AI algorithms

1. **A* search** finds a shortest path on the four-direction grid. Each move costs one. A* chooses which cell to explore using `f = g + h`, where `g` counts moves already taken and `h` is the remaining horizontal plus vertical distance. Breadth-first search (BFS) is the comparison algorithm and explores cells by distance from the start. Both return the moves after the starting cell, so distance equals the number of cells in the returned path. A walkable start equal to the goal costs zero. An unreachable goal has an empty path and infinite cost.
2. **Bayesian inference** uses the prior obstacle probability and the sensor’s error rates to estimate the probability of an obstacle after a BLOCKED or CLEAR reading. For example, with the default settings, a BLOCKED reading gives `(0.30 × 0.80) / (0.30 × 0.80 + 0.70 × 0.20) ≈ 0.6316`. A CLEAR reading gives about 0.0968.
3. **MDP value iteration** uses that probability to choose PROCEED or DETOUR. It repeatedly compares each action’s immediate reward plus its discounted expected future value. Proceeding risks a collision (−15) for a chance at direct delivery (+10). A detour incurs a step cost (−0.2) and offers a safer route to delivery (+8). The solver uses full precision for decisions and refines close calls before rounding values for display. The naive comparison policy detours on BLOCKED and proceeds on CLEAR.

## Reproduce the benchmark

1. Set sensitivity to **0.80**, false alarm to **0.20**, discount factor to **0.90**, and obstacle prior to **0.30**. Click **Run Benchmark Test (10 Runs)**.
2. Change only the prior to **0.80** and click the benchmark button again. The live algorithm toggles can be in either position.

Each run compares **A* + Bayes + MDP** with **BFS + Naive Sensor**. For run numbers 1 through 10, each configuration starts its random generator from the same seed, `7919 × run number`. Within a pair, both configurations receive the same checkpoint hazards, orders, and sensor readings. Eight orders are assigned to three robots in rotation. For each order, the policy evaluates all four checkpoints before the robot plans its two legs, first to pickup and then to drop-off.

| Obstacle prior | Configuration | Nodes explored | Distance (grid moves) | Collisions |
| ---: | --- | ---: | ---: | ---: |
| 0.30 | A* + Bayes + MDP | 455.6 | 182.2 | 0.1 |
| 0.30 | BFS + Naive Sensor | 1444.7 | 182.2 | 0.1 |
| 0.80 | A* + Bayes + MDP | 448.4 | 182.8 | 0.0 |
| 0.80 | BFS + Naive Sensor | 1432.0 | 182.4 | 0.3 |

Each value is the mean **per eight-order batch** over ten seeds. The node count includes each cell examined during a search, including its goal. Distance counts grid moves across all 16 legs. A collision counts each entry into a checkpoint containing a hidden obstacle. [Results for every seed](benchmark-results.txt)

At prior 0.30, both policies made the same checkpoint decisions. Both configurations averaged 182.2 moves and recorded one collision each across all ten batches. A* explored 68.5% fewer nodes. At prior 0.80, even a CLEAR reading left a 0.50 obstacle probability, so the MDP chose DETOUR for either reading. Across the ten batches, the AI configuration traveled four extra cells in total and recorded zero collisions. The baseline recorded three.

### Measurement limits

The benchmark measures search effort, distance, and collisions. It does not time execution. At the higher prior, both the search algorithm and the avoidance policy can affect the comparison. Ten seeds on one layout give a reproducible classroom example and a limited sample of collisions.

Benchmark hazards stay fixed for the whole batch, so repeated visits to an occupied checkpoint can each count as a collision. In the live simulation, robots read sensors as they approach, a collision clears the hazard, and charging, delays, dispatch priorities, and user-placed boxes can change a trip.

## Project architecture

All five classes are in [index.html](index.html).

| Class | Responsibility |
| --- | --- |
| `Grid` | Stores shelves, aisle cells, pickups, charging pads, drop-offs, checkpoint hazards, and spilled boxes. |
| `AStarPathfinder` | Implements A* and BFS and returns the route, explored cells, and cost. |
| `BayesianSensorModel` | Generates noisy sensor readings and calculates the posterior obstacle probability. |
| `MDPValueIterationSolver` | Solves the corridor decision and returns the action values and selected action. |
| `WarehouseSim` | Dispatches orders, moves and charges robots, draws the map, updates telemetry, and runs the paired benchmark. |
