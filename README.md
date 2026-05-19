# Battle City — AI Adaptive Combat

> A fully playable Python clone of the classic Tank 1990 arcade game, built as a semester project for **AL2002 Artificial Intelligence Lab (Spring 2026)**. Every enemy tank runs a different AI algorithm — from simple BFS pathfinding to a Minimax boss with Alpha-Beta pruning.

---

## How to Run

**Requirements:** Python 3.8+, pygame

```bash
# 1. Clone the repo
git clone https://github.com/YOUR-USERNAME/battle-city-ai.git
cd battle-city-ai

# 2. Install dependency
pip install pygame

# 3. Run the game
python main.py
```

### Controls

| Key | Action |
|-----|--------|
| W / ↑ | Move up |
| S / ↓ | Move down |
| A / ← | Move left |
| D / → | Move right |
| Space | Shoot |
| Enter | Start game (from menu) |
| 1 | Jump to Stage 1 |
| 2 | Jump to Stage 2 |
| B | Jump to Boss Level |
| D | Toggle debug mode |

---

## Project Structure

```
battle-city-ai/
│
├── main.py           # Game loop, state management, rendering
├── tank.py           # Tank base class, PlayerTank, EnemyTank, BossTank
├── bullet.py         # Bullet movement and collision logic
├── game_map.py       # CSP map generator, tile rendering
├── hud.py            # HUD panel, overlays, AI analysis display
├── constants.py      # All game settings, colours, tile types, spawn points
│
└── agents/
    ├── basic_tank_agent.py   # BFS pathfinding — Simple Reflex Agent
    ├── fast_tank_agent.py    # Greedy Best-First Search — Goal-Based Agent
    ├── armor_tank_agent.py   # A* Search — Model-Based Reflex Agent
    └── boss_tank_agent.py    # Minimax + Alpha-Beta Pruning — Adversarial Agent
```

---

## AI Modules

### Module A — CSP Map Generator

Every level is procedurally generated using **Constraint Satisfaction**. The generator applies backtracking search with five hard constraints:

| Constraint | Rule |
|------------|------|
| Base Safety | Eagle must be surrounded by at least one ring of brick/steel |
| Reachability | BFS must confirm a valid path from every spawn to the Eagle |
| Fairness | No spawn tile within 10 Manhattan-distance tiles of the player |
| Density | Wall tiles cannot exceed 40% of the 26×26 grid |
| Water Placement | Water tiles may never block the only path to the Eagle |

Maps are rejected and regenerated if any constraint fails after placement.

---

### Module B — Search Algorithms

Three enemy types, three algorithms — each behaviorally distinct.

#### Basic Tank — BFS (Simple Reflex Agent)
- Finds the **shortest-hop path** to the Eagle using Breadth-First Search
- Replans at spawn, every 5 seconds, or when a wall is destroyed mid-game
- Cost-blind: treats all passable tiles equally
- Will take a long detour around a single brick wall

#### Fast Tank — Greedy Best-First Search (Goal-Based Agent)
- Uses **Manhattan distance to the Eagle** as its sole heuristic
- Makes a single-step greedy decision every tick — no full path computed
- Rushes the base fast but gets stuck in local minima (intentional — demonstrates why greedy alone fails)
- Ignores the player entirely; only goal is destroying the Eagle

#### Armor Tank — A\* Search (Model-Based Reflex Agent)
- **Cost-aware pathfinding**: `f(n) = g(n) + h(n)`
- Tile costs: Empty = 1, Forest = 1, Brick = 3, Steel = ∞, Water = ∞
- Will drill through a thin brick wall (cost 3) rather than walk a 6-tile detour (cost 6)
- Maintains internal `hitCount` state — retreats to the nearest steel wall on the 3rd hit, waits 2 seconds, then resumes attack
- Replans whenever a wall in its current path is destroyed

**Key demo:** Place a 1-tile brick wall across the direct path with a 6-tile open detour beside it. BFS goes around. A\* shoots through the wall.

---

### Module C — Adversarial Search (Boss Level)

The Boss Tank uses full **Minimax search with Alpha-Beta Pruning**, simulating your best responses before every move.

```
MAX node: Boss picks action that maximises its evaluation score
MIN node: Simulated player picks action that minimises Boss's score
Branching factor: ~5 (Up / Down / Left / Right / Shoot)
```

#### Boss Phases

| Phase | HP | Speed | Fire Rate | Depth | Behaviour |
|-------|----|-------|-----------|-------|-----------|
| Phase 1 | 9–12 | Slow | 2s | 2 | Aggressive push toward player |
| Phase 2 | 4–8 | Medium | 1.5s | 3 | Balanced attack + seek steel cover |
| Phase 3 | 1–3 | Fast | 0.8s | 4 | Desperate rush, ignores self-preservation |

#### Evaluation Heuristic

| Factor | Score |
|--------|-------|
| Player within 3 tiles | +60 |
| Player in line-of-sight | +50 |
| Boss adjacent to steel wall | +30 |
| Player HP missing (per hit) | +20 |
| Player in forest tile | −20 |
| Boss HP missing (per hit) | −40 |

#### Alpha-Beta Pruning Results

Without pruning at depth 4: **625 nodes**  
With Alpha-Beta: **~25 nodes**  
Speedup: **~25×**

These are measured and logged live to `boss_stats.txt` during every Boss fight. The HUD also displays raw vs pruned node counts in real time.

---

## Game Mechanics

- **Grid:** 26×26 tiles for Stages 1–2, 12×12 arena for the Boss Level
- **Tile types:** Empty, Brick (destructible), Steel (indestructible), Water (bullets pass, tanks blocked), Forest (hides tanks), Eagle (destroy = game over)
- **Dynamic map:** Destroyed brick walls permanently open new paths — AI agents replan in real time
- **Enemy pool:** 20 enemies per level, maximum 4 active simultaneously
- **Lives:** Player starts with 3 lives
- **Win:** Destroy all enemies in the pool
- **Lose:** Player runs out of lives, or any bullet hits the Eagle

---

## License

This project was built for academic purposes as part of AL2002 Artificial Intelligence Lab, Spring 2026.
