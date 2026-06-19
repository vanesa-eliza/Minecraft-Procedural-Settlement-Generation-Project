# GDMC Settlement Generator

Procedurally generates a coherent medieval settlement in a Minecraft world by 
analysing terrain, planning districts and roads, then constructing biome-aware 
structures—all written back via the GDMC HTTP Interface.

**Built as a team project** where I led systems integration, refactoring, and 
AI decision-making components.

## Requirements

- Python 3.10+
- Minecraft running with the **GDMC HTTP Interface** mod active
- Python deps: `gdpc>=7.0.0`, `numpy>=1.24.0`

## Setup & Run

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

## The Challenge

Building a settlement generator requires solving several interconnected problems:

1. **Placement accuracy** — structures must sit on solid ground, not float or clip 
   into terrain. This required careful heightmap analysis and validation against 
   an occupancy map.
2. **Terrain adaptation** — terraforming logic needed to fill pits and level building 
   sites realistically, as a player would.
3. **Systems integration** — each team member owned a subsystem (terrain analysis, 
   planning, structures, world I/O). My role was ensuring they worked cohesively, 
   refactoring interfaces and eliminating duplication.
4. **Autonomous decisions** — the system needed to choose district types, select 
   appropriate buildings, and score designs without hardcoded rules.

## How It Works

`main.py` → `create_generator()` wires dependencies and runs the full pipeline:

1. **Analyse** terrain, pick best buildable patch
2. **Assign district types** using value iteration MDP
3. **Plan layout** — plaza, ring road, districts (Voronoi), plots
4. **Build structures** — selector picks a building per plot, orchestrators 
   compose grammar rules to construct it
5. **Score designs** — Random Forest + n-gram model evaluate candidate house shapes
6. **Flatten & place** — validate placement, terraform, write to world

See [diagram.mmd](diagram.mmd) for the full dependency map.

## My Contributions

- **Structure architecture** — designed the selector → orchestrator → grammar → 
  primitive builder composition, allowing new building types to be added independently
- **AI integration** — implemented MDP for district-type assignment and integrated 
  trained Random Forest + n-gram models for structure scoring
- **Systems integration** — refactored the codebase to eliminate coupling between 
  terrain analysis, planning, and building subsystems, ensuring each team member's 
  work composed cleanly
- **Placement & terraforming** — solved the ground-level accuracy problem by 
  validating structure footprints against heightmaps and occupancy maps, with 
  realistic pit-filling logic

## What I Learned

- **System design at scale** — coordinating 10+ interdependent modules, each 
  authored by different people, requires clear interfaces and dependency injection
- **Algorithms in practice** — pathfinding (A*, MST), spatial partitioning (Voronoi), 
  and decision-making (MDP, ML scoring) solve real placement and design challenges
- **Team coordination** — refactoring and integration work is invisible but critical; 
  good architecture makes future collaboration seamless
- **Real-world constraints** — theoretical solutions fail when terrain is uneven, 
  structures overlap, or edge cases emerge. Validation and fallback logic matter

## Testing

Standalone debug scripts in `debug/test_*.py` validate individual structures:

```bash
python debug/test_house.py
python debug/test_tavern.py
```

## Project Layout

| Path | Purpose |
|------|---------|
| `generators/` | Composition root + generation pipeline |
| `analysis/` | Terrain fetch, scoring, best-patch selection |
| `ai/` | District-type assignment (MDP via value iteration) |
| `planning/` | District, road, and plot planners |
| `structures/` | Building selector → orchestrators → grammar → primitives |
| `data/` | Config, analysis results, settlement entities |
| `utils/` | A*, MST, Poisson disk, pathfinding, geometry |
| `world_interface/` | Terrain I/O, block buffer, placer, terraforming |
| `training/` | ML training scripts (Random Forest, n-gram) |

For a deeper walkthrough, see [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md).