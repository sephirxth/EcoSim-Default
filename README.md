# EcoSim -- Individual-Based Evolving Predator-Prey Ecosystem Simulation

An open-source, individual-based model (IBM) that simulates the co-evolution of predator and prey populations on a 2D grid, using Fuzzy Cognitive Maps for autonomous decision-making and genetic algorithms for trait evolution and speciation.

## What is EcoSim?

EcoSim models a virtual ecosystem where thousands of prey ("rabbits") and predators ("wolves") live, forage, reproduce, and evolve on a toroidal grid world. Each individual carries its own behavioral brain -- a Fuzzy Cognitive Map (FCM) -- and a physical genome encoding traits like maximum energy, speed, vision range, and lifespan. Over generations, mutation and sexual reproduction drive trait divergence, producing emergent speciation events, arms races, and population dynamics without any top-down scripting.

## Features

- **Autonomous agent behavior via Fuzzy Cognitive Maps.** Each individual perceives its local environment (nearby food, friends, enemies, energy level) and uses an FCM to weigh competing drives -- fear, hunger, curiosity, socialization -- to choose an action each time step.
- **Evolvable physical genome.** Eight genes per prey and six per predator encode traits including maximum energy (body size), maximum speed, vision range, maximum age, reproduction age, state-of-birth energy, defence, and cooperative defence. All are subject to mutation and crossover.
- **Emergent speciation.** Species identity is tracked by evolutionary distance in both FCM weights and genome space. When an individual's cumulative distance from its species mean exceeds a threshold, a new species is created, enabling phylogenetic tracking.
- **Sexual reproduction with mate selection.** Three mating modes: random, "good gene" (fitness-proportional), and intermediate. Gender-differentiated reproduction costs and pregnancy energy mechanics are modeled.
- **Rich resource dynamics.** Grass grows stochastically on the grid, with configurable options for fluctuating resources (sinusoidal maxGrass cycles) and circular food patches. Dead prey leave meat that decays over time, creating a secondary food source for predators.
- **Comprehensive statistics and save/restore.** Per-generation statistics (population counts, average traits, speciation events, death causes) are written to results files. Full simulation state can be saved (MaxSave) and restored for long-running experiments. Optional HDF5 output for large-scale data analysis.
- **BMP visualization.** Periodic snapshots of the world are rendered as BMP images showing prey, predator, grass, and meat distributions.

## Quick Start

### Prerequisites

- A C++ compiler (the default makefile uses Intel `icpc`; see below for g++)
- HDF5 C++ library (`libhdf5-dev`) if HDF5 output is desired (optional)

### Build

```bash
git clone https://github.com/sephirxth/EcoSim-Default.git
cd EcoSim-Default
```

The default `makefile` uses `icpc` (Intel C++ Compiler). To use g++ instead, edit the first line of `makefile`:

```makefile
CC=g++
```

If you do not need HDF5 output (the default), also remove `-lhdf5_cpp` from `LINKFLAGS` and `-D H5_USE_16_API` from `CFLAGS`:

```makefile
CC=g++
CFLAGS=-c -Wall -O3
LINKFLAGS=
```

Then build and run:

```bash
make
./EcoSim
```

### Starting a New Run

1. Set `Restore: 0` in `Parameters1.txt`.
2. Run `./EcoSim`. The simulation will create output files in the working directory.

### Continuing a Previous Run

1. Ensure a `MaxSave` file exists in the working directory (created automatically at the interval specified by `MaxSave` in `Parameters1.txt`).
2. Set `Restore: 1` in `Parameters1.txt`.
3. Run `./EcoSim`.

The simulation runs indefinitely until a population goes extinct or you terminate it manually.

## How It Works

### Simulation Loop

Each generation (time step), the ecosystem:

1. **Perceives** -- every individual scans its local neighborhood (within its vision range) for food, friends, and enemies.
2. **Decides** -- the individual's FCM integrates sensory inputs with internal states (fear, hunger, curiosity, satisfaction, etc.) through weighted connections, then selects the highest-activation motor node as the action: escape, eat, search for food, reproduce, socialize, explore, or wait.
3. **Acts** -- the chosen action is executed, consuming energy and potentially changing position, feeding, or spawning offspring.
4. **Dies** -- individuals die of old age, starvation (zero energy), or predation (killed in a fight).
5. **Reproduces** -- mating requires a willing, nearby, opposite-gender partner of sufficient age and energy. Offspring inherit FCM weights and genome via crossover with mutation.
6. **Speciates** -- evolutionary distance is recalculated; individuals exceeding the speciation threshold are assigned to new species.
7. **Updates resources** -- grass regrows stochastically; meat decays.
8. **Records statistics** -- per-generation stats and optional per-species breakdowns are written to disk.

### Decision Architecture (FCM)

The Fuzzy Cognitive Map is a weighted directed graph where:
- **Sensory nodes** (22 for prey, 24 for predators) encode fuzzified perceptions: predator proximity, food availability, partner presence, energy level, strength, age.
- **Concept nodes** (7 each) represent internal motivational states: fear/chase-away, hunger, search-partner, curiosity, sedentary, satisfaction, nuisance.
- **Motor nodes** (10 for prey, 11 for predators) represent candidate actions: escape, search-food, eat, reproduce, socialize, explore, wait, and several strategic movement options.

The connection weights between these nodes are encoded in the individual's FCM chart and are subject to mutation and crossover, allowing behavioral strategies to evolve alongside physical traits.

### Genome

Physical traits are encoded as unsigned-char genes mapped to floating-point ranges:
- Gene 0: Maximum energy (body size), range 100--6475
- Gene 1: Maximum age, range 10--265
- Gene 2: Vision range, 1--25 cells
- Gene 3: Maximum speed, 1--25 cells/step
- Gene 4: Reproduction age
- Gene 5: State-of-birth energy (parental investment)
- Gene 6: Defence (prey only), 0--1
- Gene 7: Cooperative defence (prey only), 0--0.5

## When to Use This

- **Ecology and evolution education.** Demonstrate predator-prey dynamics, speciation, and natural selection in a hands-on computational lab.
- **Artificial life research.** Study emergent behavior, evolutionary arms races, and the conditions under which speciation occurs.
- **Agent-based modeling methodology.** The FCM-based decision architecture is a well-documented alternative to rule-based or neural-network agents.
- **Game AI prototyping.** The individual perception-decision-action loop and evolvable behavioral weights can inspire autonomous NPC designs.
- **Parameter sensitivity studies.** Explore how mutation rate, resource availability, mating strategy, or grid size affect long-term population stability and diversity.

## Configuration

All parameters are set in `Parameters1.txt`. Key parameters:

### Environment

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Width` / `Height` | 1000 | Grid dimensions |
| `MaxGrass` | 4000 | Maximum grass per cell |
| `SpeedGrowGrass` | 200 | Grass energy value when consumed (growth speed) |
| `ProbaGrowGrass` | 0.035 | Probability of grass growing in a cell per step |
| `ValueGrass` | 350 | Energy gained from eating grass |
| `MaxMeat` | 4000 | Maximum meat per cell (from dead prey) |
| `FluctuatingResources` | 0 | Enable sinusoidal resource fluctuation (0/1) |
| `CircularFoodGrowth` | 0 | Enable circular food patches (0/1) |

### Population

| Parameter | Default | Description |
|-----------|---------|-------------|
| `InitNbPrey` | 80000 | Initial prey population |
| `InitNbPredator` | 4000 | Initial predator population |
| `AgeMaxPrey` / `AgeMaxPred` | 46 / 50 | Maximum lifespan (generations) |
| `AgeReprodPrey` / `AgeReprodPred` | 6 / 5 | Minimum reproduction age |
| `VisionPrey` / `VisionPredator` | 8 / 20 | Initial vision range (cells) |
| `MatingMode` | 0 | Mate selection: 0=random, 1=good-gene, 2=intermediate |
| `IsWithoutPredator` | 0 | Run prey-only simulation (0/1) |

### Evolution

| Parameter | Default | Description |
|-----------|---------|-------------|
| `ProbaMut` | 0.001 / 0.002 | Mutation probability (prey / predator) |
| `ProbaMutLow` | 0.0005 / 0.0015 | Mutation probability floor |
| `PercentMut` | 0.15 | Mutation magnitude (fraction of gene range) |
| `DistanceSpeciesPrey` | 16 | Evolutionary distance threshold for prey speciation |
| `DistanceSpeciesPred` | 16 | Evolutionary distance threshold for predator speciation |

### Output

| Parameter | Default | Description |
|-----------|---------|-------------|
| `MaxSave` | 20 | Full save interval (0=disabled) |
| `MinSave_Compressed` | 1 | Compressed individual data save interval |
| `WorldSave` | 1 | Save world state as BMP (0/1) |
| `Visualizations` | 0 | BMP snapshot interval (0=disabled) |

The FCM weight matrices for prey and predators are also defined in `Parameters1.txt`, after the scalar parameters. Refer to `EcoSim ODD.docx` for a complete ODD protocol description.

## License

GNU General Public License v3.0. See [LICENSE](LICENSE) for full text.
