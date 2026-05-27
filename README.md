# Energy and Performance-Aware Task Scheduling in Mobile Cloud Computing

**EECE 7205 — Fundamentals of Computer Engineering, Project 2**
Animesh Kashid (002080335)

A C++ implementation of the energy- and performance-aware task scheduling algorithm for a mobile cloud computing (MCC) environment described in:

> X. Lin, Y. Wang, Q. Xie, and M. Pedram, "Energy and Performance-Aware Task Scheduling in a Mobile Cloud Computing Environment," *2014 IEEE 7th International Conference on Cloud Computing*, Anchorage, AK, 2014, pp. 192–199. doi: 10.1109/CLOUD.2014.35.

The program takes a task dependency graph and per-core execution times as input, produces an initial schedule across three heterogeneous local cores and a cloud channel, and then runs a task-migration phase to minimize total energy consumption while keeping completion time under a user-specified bound `Tmax`.

---

## Repository contents

| File | Description |
|------|-------------|
| `Project_2_Animesh_Kashid_002080335.cpp` | Full algorithm implementation (entry point in `main`) |
| `Matrix.h` | Bjarne Stroustrup's multidimensional matrix library (used for the dependency graph `G` and execution-time table `ta`) |
| `Project_2_Animesh_Kashid_002080335.exe` | Pre-built Windows executable |
| `project_report.pdf` | Full write-up with five worked examples, Gantt charts, and energy breakdowns |
| `project_presentation_pptx.pptx` | Slide deck summarizing the project |
| `outputs.xlsx` | Spreadsheet with the visualized initial and final schedules for each test case |

---

## Algorithm overview

The scheduler runs in two stages.

**Stage 1 — Initial scheduling.** Each task is first classified as a local-only or cloud-eligible task (`primary`), then prioritized in reverse topological order using the recursive priority rule from the paper (`prioritize`). Tasks are scheduled greedily in priority order onto whichever channel — one of the three local cores or the cloud — gives the earliest finish time (`initials`, supported by `locals` and `clouds`).

**Stage 2 — Task migration.** The outer loop (`outerloop`) repeatedly invokes the migration kernel (`mcc`) until no further energy reduction is possible. For each task currently on a local core, the kernel (`kernel`) tentatively re-runs it on every other channel and rebuilds the schedule using `ready1` (unscheduled predecessors) and `ready2` (unscheduled same-channel tasks ahead of it) counters. A migration is accepted if it strictly reduces energy without increasing completion time, or if it offers the best energy-per-time-unit trade-off while staying within `Tmax`.

Energy accounting (`find_en`) follows the paper: per-core active power `P1`, `P2`, `P3` for local execution, and `Ps` for the RF send phase. Cloud compute and receive are treated as free from the mobile device's perspective.

---

## Building

The code is a single translation unit with one header dependency.

```bash
g++ -std=c++11 -O2 Project_2_Animesh_Kashid_002080335.cpp -o scheduler
```

Make sure `Matrix.h` sits in the same directory as the `.cpp` file. The pre-built `Project_2_Animesh_Kashid_002080335.exe` is included for Windows users who don't want to compile.

---

## Running

The program reads all inputs interactively from `stdin`. A typical session asks for:

1. Number of tasks `N`
2. Maximum allowed completion time `Tmax`
3. Per-core active powers: `P1`, `P2`, `P3` (typical values: 1, 2, 4)
4. Wireless send power `Ps` (typical: 0.5)
5. Number of edges in the dependency DAG, then each edge as a (from, to) pair (0-indexed)
6. The `N × 3` execution-time table — for each task, its run time on local cores 1, 2, and 3

The cloud timing parameters `Ts=3`, `Tc=1`, `Tr=1` (send, compute, receive) are hard-coded to match the paper.

### Output

For both the initial schedule and the post-migration schedule, the program prints:

- Each task's assigned channel (local core 1/2/3 or cloud), start time, and finish time
- Total energy consumption
- Total completion time
- Wall-clock runtime of that stage in milliseconds

---

## Example results

From the report, on the five test graphs (10–20 tasks, `Tmax` from 27 to 50):

| Example | Tasks | Tmax | Initial time / energy | Final time / energy | Energy reduction |
|---------|-------|------|------------------------|-----------------------|------------------|
| 1 | 10 | 27 | 18 / 100.5 | 27 / 22 | 78% |
| 2 | 10 | 27 | 22 / 107.5 | 27 / 31.5 | 71% |
| 3 | 20 | 50 | 33 / 174 | 39 / 74.5 | 57% |
| 4 | 20 | 40 | 32 / 189 | 40 / 79 | 58% |
| 5 | 20 | 40 | 24 / 158 | 40 / 73 | 54% |

The migration stage consistently trades a modest increase in completion time (still within `Tmax`) for a large energy reduction by offloading the heavy local-core-3 tasks to the cloud.

---

## Data structures

The core task representation (`struct task`) tracks per-channel ready and finish times — `RTl`/`FTl` for local, `RTWS`/`FTWS` for wireless send, `RTC`/`FTC` for cloud compute, `RTWR`/`FTWR` for wireless receive — alongside scheduling metadata: priority, channel assignment, entry/exit flags, and the two ready counters used during migration. The DAG is stored as an `N × N` adjacency matrix `G`, and execution times as an `N × 3` matrix `ta`, both built on `Numeric_lib::Matrix`.

---

## Notes and limitations

- Inputs are typed in by hand, which is fine for the paper's small graphs but tedious for larger ones; redirecting from a file with the same line-by-line format works.
- Channel encoding: `0`, `1`, `2` are local cores 1, 2, 3 and `3` is the cloud.
- Edge input uses 0-indexed node IDs (so for 10 tasks, valid IDs are 0–9), while the schedule output uses 1-indexed task numbers.
- The first example's final schedule differs slightly from the paper, as noted in the conclusion; the remaining four match exactly.

---
