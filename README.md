# Flying Block Puzzle — Class-Based Heuristic A\* (CBHA\*)

This repository contains the complete source code and benchmark test cases for **CBHA\*** (Class-Based Heuristic A\*), an algorithm for optimally solving the two-column Flying Block Puzzle. The puzzle is a rigorously NP-complete spatial planning microworld whose bottleneck geometry mirrors constraints found in multi-agent path finding, autonomous vehicle navigation, and block relocation systems.

> **Paper:** *Class-Based Heuristic Selection for Solving the Flying Block Puzzle*
> Sanyar Ahmadi · Pedram Asadzadeh · Amanj Khorramian
> University of Tehran · K. N. Toosi University of Technology · University of Kurdistan

---

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Input Format](#input-format)
- [Kinematic Taxonomy (Seven Classes)](#kinematic-taxonomy-seven-classes)
- [Heuristic Formulas](#heuristic-formulas)
- [Benchmark Results](#benchmark-results)
- [Test Cases](#test-cases)
- [License](#license)
- [Authors](#authors)

---

## Overview

Pieces (polyominoes) are placed in a 2 × h frame. The objective is to find a minimum-length move sequence that brings a designated target piece (ID `1`) to its goal position. Two move types are defined:

- **Jumping move** — the piece's new and old positions are disjoint (C′ ∩ C = ∅).
- **Sliding move** — the piece partially overlaps its previous position (C′ ∩ C ≠ ∅, |C′ ∩ C| < |C|).

Three fundamental constraints govern feasibility:

| Constraint | Statement |
|---|---|
| General Move Constraint | A piece of size *n* with *k* vacant cells satisfies \|C ∩ C′\| ≥ n − k; at most *k* cells can be vacated per move. |
| Jumping Move Constraint | If k < n, jumping moves are impossible. |
| Crossing Constraint | A rectangular piece of size n > 2 with k < n cannot cross columns. |
| Comb-Piece Flip Constraint | A comb with *s* free spaces cannot flip across its horizontal axis when k < s. |

---

## Repository Structure

```
.
├── All codes/
│   ├── Final_7class_priority.py   # CBHA* — seven-class algorithm with class-conditional heuristics and tie-breaking
│   ├── H1.py                      # Standard A* (SA*) with general heuristic
│   ├── H2.py                      # Depth-Prioritized A* (DPA*)
│   ├── BFS.py                     # Breadth-First Search baseline
│   └── branch factor.py           # Effective branching factor computation utility
└── Test case/
    ├── H_A(Test case)/            # 32 Class A instances  (frame heights 5–500, k − n = 0, 1)
    ├── H_B(Test case)/            # 24 Class B instances  (frame heights 10–45, vertical distances 1–23)
    ├── H_C(Test case)/            # 27 Class C instances  (frame heights 5–100, vertical distances 1–47)
    ├── H_D(Test case)/            # 26 Class D instances  (frame heights 9–40, vertical distances 4–29)
    ├── H_E(Test case)/            # 13 Class E instances  (frame heights 5–14, vertical distances 2–24)
    ├── H_F(Test case)/            # 12 Class F instances  (frame heights 9–22, comb free-space configs)
    └── H_G(test case)/            # 12 Class G instances  (frame heights 10–19, residual shapes)
```

Each test case is stored in a subdirectory and consists of two text files:

- `*_Frame.txt` — initial board state (two columns of piece IDs; `0` = vacant).
- `*_Goal.txt` — target board state in the same format.

Subdirectory names encode the instance parameters directly, e.g.:

```
H_B(Test case)/n = 10/# goal piece - # vacant unit = 1/Goal_piece(I) = 3/Vertical distance = 4/
```

---

## Requirements

- Python 3.x
- [`psutil`](https://pypi.org/project/psutil/) — memory monitoring

---

## Installation

```bash
# Clone the repository
git clone https://github.com/Pedyi/Flying-Block-Puzzle.git
cd Flying-Block-Puzzle

# Install the only external dependency
pip install psutil
```

No other packages are required. All algorithms use Python's standard library (`heapq`, `math`, `time`).

---

## Usage

All four algorithms share the same interactive input interface. Run any script and enter the puzzle data when prompted:

```bash
# Run CBHA* (recommended)
python "All codes/Final_7class_priority.py"

# Run Depth-Prioritized A*
python "All codes/H2.py"

# Run Standard A*
python "All codes/H1.py"

# Run BFS baseline
python "All codes/BFS.py"
```

Each script prompts for:
1. Frame height `n`
2. Initial frame contents (n rows, space-separated column values)
3. Goal configuration (n rows, same format)

### Running from a file

You can pipe a test case file directly into any script:

```bash
# Concatenate height, frame, and goal into a single stream
(echo 10; cat "Test case/H_B(Test case)/n = 10/# goal piece - # vacant unit = 1/Goal_piece(I) = 3/Vertical distance = 4/1_Frame.txt"; cat "Test case/H_B(Test case)/n = 10/# goal piece - # vacant unit = 1/Goal_piece(I) = 3/Vertical distance = 4/1_Goal.txt") | python "All codes/Final_7class_priority.py"
```

### Computing the Effective Branching Factor

After a run that reports the number of expanded nodes `N` and solution depth `d`:

```bash
python "All codes/branch factor.py"
# Enter N when prompted for n, then enter d
```

---

## Input Format

Each file contains one row per cell, top to bottom, with two space-separated integers representing piece IDs for column 0 and column 1. `0` denotes a vacant unit. The target piece always has ID `1`.

**Example — `1_Frame.txt` (frame height 10):**

```
1 1
2 1
2 3
2 3
0 4
5 5
6 0
7 8
9 9
10 11
```

**Example — `1_Goal.txt` (same frame, goal for piece 1):**

```
0 0
0 0
0 0
0 0
0 0
2 0
2 0
2 0
0 0
0 0
```

Non-target pieces (`2`, `3`, …) are set to `0` in the goal file; only the target piece (`1`) specifies its required position. The goal file encodes the piece ID, not just the cell index — shown above, piece `2` (size 3) must occupy rows 5–7 of column 0.

---

## Kinematic Taxonomy (Seven Classes)

CBHA\* detects the class in O(1) at initialisation from three parameters: number of vacant units `k`, goal-piece size `n`, and goal-piece geometry.

| Class | Conditions | Goal-piece type | Key kinematic feature |
|---|---|---|---|
| **A** | k ≥ n | Any | Jumping moves available; general heuristic is informative. |
| **B** | k < n, n ≥ 3 | I-shaped | Column-locked sliding only; no column crossing. |
| **C** | k = 1, n = 2 | I-shaped | Rotation and column-crossing enabled. |
| **D** | k = 1 | L-shaped | All path blockers must be fully cleared. |
| **E** | k = 2 | L-shaped | Size-1 blockers can be bypassed via a two-vacancy slide. |
| **F** | s > k | Comb-shaped | Body side-pinned; vertical flip impossible. |
| **G** | s ≤ k, or other symmetric shape | Comb / Other | Residual case; general distance heuristic applied. |

---

## Heuristic Formulas

Each class uses a provably admissible and consistent heuristic. Here `k(s)` is the number of vacant units, `d(s)` or `d_v(s)` is the relevant vertical distance, `P_X(s)` is the set of blocking pieces on the goal path for class X, and `g_p(s)` is the number of path-cells occupied by piece `p`.

| Class | Heuristic formula |
|---|---|
| A | h_A(s) = Σ ⌈g_i / k⌉ + δ(s) |
| B | h_B(s) = ⌈d(s)/k(s)⌉ + Σ_{p ∈ P_B} ⌈g_p(s)/k(s)⌉ |
| C | h_C(s) = 2(d_v(s) + R(s) + F(s)) − M(s) |
| D | h_D(s) = d_v(s) + Σ_{p ∈ P_D} g_p(s) |
| E | h_E(s) = d_v(s) + Σ_{p ∈ P_E} ⌈g_p(s)/2⌉ − δ(s) |
| F | h_F(s) = ⌈d/k⌉ + Σ ⌈g_i/k⌉ |
| G | h_G(s) = ⌈d/k⌉ |

**Class C indicators:** R(s) = 1 if a terminal rotation is needed after vertical alignment; F(s) = 1 if goal piece and target are on opposite frame sides; M(s) = 1 if the goal piece can advance without an extra clearing step.

**Class E indicator:** δ(s) = 1 if the bypass precondition (goal piece pointing toward target, size-1 blocker present) is active.

**Tie-breaking:** depth priority (larger `g` preferred) for Classes A–E; vertical-distance priority (smaller `d` preferred) for Classes F and G.

---

## Benchmark Results

Evaluated over **146 instances** across all seven classes, under a resource budget of 12 GB RAM and 30 minutes wall-clock time per run. Averages below exclude Class A (shallow solutions) and two Class A outlier instances (frame heights 200 and 500).

| Algorithm | Success rate | Avg. expanded nodes (B–G) | Avg. EBF | Avg. max depth |
|---|---|---|---|---|
| BFS | 17% | ~17 M | ~74 | ~4 |
| SA\* | 39% | ~13 M | ~20 | ~5 |
| DPA\* | 64% | ~8 M | ~7 | ~8 |
| **CBHA\*** | **93.4%** | **~1.6 M** | **~3** | **~14** |

CBHA\* reduces node expansions by **87.98%** relative to SA\* and **91.31%** relative to BFS, while sustaining an average effective branching factor of approximately 3.

**Per-class breakdown:**

| Class | N | BFS success | SA\* success | DPA\* success | CBHA\* success | CBHA\* avg. nodes | CBHA\* EBF | CBHA\* avg. depth |
|---|---|---|---|---|---|---|---|---|
| A | 32 | 28% | 87% | 100% | 100% | 8,941 | 4 | 4 |
| B | 24 | 25% | 33% | 58% | 95% | 572,268 | 1.5 | 14 |
| C | 27 | 15% | 18% | 37% | 96% | 657,010 | 1.5 | 22 |
| D | 26 | 7% | 11% | 46% | 96% | 564,743 | 1.8 | 19 |
| E | 13 | 8% | 8% | 46% | 92% | 1,565,849 | 2 | 12 |
| F | 12 | 8% | 33% | 66% | 92% | 2,663,743 | 4 | 9 |
| G | 12 | 23% | 62% | 96% | 82% | 3,575,216 | 9 | 5 |

> Class G is the only class where DPA\* outperforms CBHA\* at small vertical distances, where h_G ≈ 0.

---

## Test Cases

The 146 benchmark instances are organised by class inside the `Test case/` directory. Subdirectory names encode the key instance parameters:

| Class | # instances | Parameter axes encoded in path |
|---|---|---|
| A | 32 (+ 2 outliers h=200,500) | Frame size `n`, `k − n` (k minus goal-piece size), goal piece type and size |
| B | 24 | Frame size `n`, `n − k` (goal-piece minus vacant), goal piece size, vertical distance |
| C | 27 | Frame size `n`, vertical distance |
| D | 26 | Frame size `n`, vertical distance |
| E | 13 | Frame size `n`, vertical distance |
| F | 12 | Frame size `n`, solution depth, goal unit, vacant unit, vertical distance, free spaces of comb |
| G | 12 | Frame size `n`, solution depth, vacant unit, goal unit |

Instances within each class are ordered by increasing difficulty (controlled by vertical distance in Classes B–G, or by `k − n` in Class A).

---

## Resource Budget

All experiments were run under a hard resource limit of **12 GB RAM** and **30 minutes** wall-clock time per run. The memory check is implemented via `psutil` in each algorithm script. An instance is recorded as a failure if either limit is reached before a solution is found.

---

## License

MIT — see [LICENSE](LICENSE).

---

## Authors

- **Sanyar Ahmadi** — Faculty of Computer Engineering, University of Tehran
  `sanyar.ahmadi@ut.ac.ir`
- **Pedram Asadzadeh** — Faculty of Computer Engineering, K. N. Toosi University of Technology
  `p.asadzadeh@email.kntu.ac.ir`
- **Amanj Khorramian** *(corresponding)* — Faculty of Computer Engineering, University of Kurdistan
  `a.khorramian@uok.ac.ir`
