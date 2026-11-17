# Atomic Toy Model — FOSC Course Project

This is our group project for **Fundamentals of Scientific Computing** (Nov 2026), submitted to Prof. Anuj Goyal and Prof. Saswata Bhattacharyya.

We built a small 2D particle simulation. Particles bounce around inside a square box with reflective walls, and any two particles closer than a cutoff distance are joined by a spring. We used it to look at:
- whether the code is correct
- how well energy is conserved
- how the timestep and the integrator affect accuracy
- what drag does to the system
- float32 vs float64
- how fast Python, NumPy and C are compared to each other

At the end we added a cell-linked neighbour list as our "innovation" part.

The full write-up with equations, tables and plots is in `FinalReport.docx` / `FinalReport.pdf`. This README is mainly here so you can find the files and run the code.

---

## Group members

| Name | Roll number |
|---|---|
| Krishna Chaitanya Namala | ES24BTECH11023 |
| Rohith Perugu | ES24BTECH11026 |
| Purna Nanda Reddy Chinthapally | ES24BTECH11014 |
| Praveen S | MS26MTECH14002 |
| Sowbarnika G | MS26MTECH14001 |

## Sequence of videos

Please watch them in this order. It follows the order of the tasks in the notebook.

1. `ES24TECH11026_FOSC_PART1`: model setup, walls, forces, energies and the Task 1 checks
2. `ES24TECH11014_FOSC_PART2`: animation and the Euler vs Verlet timestep study
3. `ES24TECH11023_FOSC_PART3`: dissipation/relaxation and float32 vs float64
4. `MS26MTECH14002_FOSC_PART4`: Python vs NumPy vs C timings and the memory study
5. `MS26MTECH14001_FOSC_PART5`: cell list, final recommendation and wrap-up

## Roles and responsibilities

- **Rohith Perugu (P1):** Set up the model: reflective walls, pairwise distances, neighbour finding, spring forces, kinetic/potential/total energy. Also wrote the Task 1 verification checks (symmetry, no self-neighbours, force balance, particles staying inside the box).
- **Purna Nanda Reddy Chinthapally (P2):** Made the particle animation (Task 2) and the diagnostics we used to read it. Ran the timestep study comparing semi-implicit Euler and velocity Verlet (Task 3).
- **Krishna Chaitanya Namala (P3):** Added the drag force, defined the relaxation criterion and ran the γ sweep (Task 4). Did the float32 vs float64 comparison (Task 5).
- **Praveen S (P4):** Wrote the Python, NumPy and compiled-C force routines and benchmarked them (Task 6). Did the memory scaling study, including measuring peak memory with `tracemalloc` (Task 7).
- **Sowbarnika G (P5):** Implemented the cell-linked neighbour search (Task 8) and the final engineering decision (Task 9). Also wrote the README, merged the report and did the final full run of the notebook.

## Acknowledgment of AI usage

- **Claude (Anthropic):** used for help with code development and debugging.

We checked all the results in the report ourselves by running the notebook.

---

## Files shared

| File / folder | What it is |
|---|---|
| `README.md` | This file |
| `Video-1 … Video-5 *.mp4` | Presentation videos, one per member (order above) |
| `FOSC_Atomic_Toy_Complete_Project_updated (1) (1).ipynb` | All source code in one notebook: simulation, tests, plots, benchmarks |
| `FinalReport.docx`, `FinalReport.pdf` | Project report with algorithms, implementation details, results, time snapshots and plots |
| `particle_animation.gif` | Animation of the 12 particles, produced by the notebook |
| `force_code.c`, `force_code.so` | C force routine; the notebook writes and compiles this itself during Task 6 |

We kept everything in one notebook instead of separate scripts. That way the testing code sits right after the function it tests, and each plot sits next to the code that makes it.

---

## How to run it

### What you need

- Python 3.9 or newer
- `numpy`, `matplotlib`, `pillow` (Pillow is used to save the GIF) and `jupyter`
- A C compiler called `gcc` on your PATH, for the compiled-C benchmark
  - macOS: `xcode-select --install` (Homebrew `gcc` also works)
  - Linux: usually already installed, otherwise `sudo apt install gcc`
  - Windows: the easiest option is WSL. MinGW should also work, but the notebook builds a `.so` file, so you may need to change the extension to `.dll`.

```bash
pip install numpy matplotlib pillow jupyter
```

### Running

```bash
jupyter notebook "FOSC_Atomic_Toy_Complete_Project_updated (1) (1).ipynb"
```

Then use **Kernel → Restart & Run All**. Run the cells from top to bottom because later sections reuse variables from earlier ones, like `pos`, `vel` and the force functions.

A few things to know:

- The random seed is fixed (`np.random.seed(42)`), so the 12-particle results should match the report.
- The timing numbers (Tasks 5, 6 and 8) will differ on your machine. The speedup ratios should be roughly the same, but don't expect the same seconds.
- The slowest parts are the dissipation runs (Task 4) and the Python all-pairs loop at N = 2000 (Task 8). On a normal laptop the full notebook takes a few minutes.
- If `gcc` isn't found, the Task 6 cell will fail. Every cell before it still works.
- The notebook saves `particle_animation.gif`, `force_code.c` and `force_code.so` to the current folder.

### Where the tests are

The tests are inside the notebook, next to the code they check:

| Notebook section | What is tested |
|---|---|
| "Testing wall reflections" | Particles placed outside the box get reflected back in and their velocity flips |
| "Verification of pairwise distances and neighbours" | Distance matrix is symmetric, diagonal is zero, no particle is its own neighbour |
| "Testing one simulation step" | Sum of internal forces is ~0 (we got about 1e-13) |
| §12 "Final numerical checks for Task 1" | Energy drift, force balance and wall containment after the full run |
| Task 2 diagnostics | Coordinate range, largest jump between frames, closest pair, time spent near walls |
| Tasks 6 & 8 | NumPy, C and cell-list forces are compared with the Python loop using `np.allclose` before timing |

---

## Main results (short version)

| What we looked at | Result |
|---|---|
| Baseline energy drift (Δt = 0.001, 1000 steps) | 0.041% |
| Euler vs Verlet at Δt = 0.01 | 3.19% vs 1.25% drift |
| Largest safe timestep (5% tolerance) | Δt = 0.01 |
| Relaxation time, γ = 0.5 | t = 149 |
| float32 vs float64 energy change | 1.86% vs 0.54% |
| C vs Python loop at N = 1000 | about 165× faster |
| Cell list vs Python loop at N = 2000 | about 15.3× faster, same forces |

What we ended up recommending: **float64**, **velocity Verlet** with **Δt = 0.01**, **C for large N** (NumPy is fine for moderate N), **γ = 0.5** if you want the particles to settle, and the **cell list** for big sparse systems.

## Known limitations

These are covered in more detail in Section 14 of the report:

- The force jumps to zero at the cutoff, which is probably why the timestep results aren't perfectly smooth.
- Our relaxation check only looks at energy. It can call the system "relaxed" while particles are still moving (γ = 0.05 is an example of this).
- There is no hard-core repulsion, so particles can get very close (about 1.3 units in the animation run).
- We didn't record the machine specs for the timings. The 8 GB RAM figure in the memory limit is an assumption, not a measured value.
- Our cell list is written in Python. A C version would be the obvious next step.
