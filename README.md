# Quantum Annealing & MILP Models for Integral Cryptanalysis  
### Artifact Repository for  
**"Applying Quantum Annealing to Block Cipher Cryptanalysis: An Experimental Study of Searching for Integral Distinguishers"**

This repository provides the full source code used for all experiments in the paper, including:

- **CQM → BQM → Hybrid Quantum Solver (D-Wave)** model  
- **Classical MILP model (Gurobi / SCIP via OR-Tools)**  
- **Figure generation scripts** (broken axes plots from the paper)  

All code in this repository is self-contained and reproduces the experimental methodology described in the publication.

---

# 📁 Repository Structure

```
.
├── CQM2BQM.ipynb            # Constrained QP → BQM → Hybrid solver model
├── MILP_and_CQM.ipynb           # Mixed-Integer Linear Programming baseline
├── makefigures.py            # Plotting functions for paper figures
└── README.md
```

---

# 🔧 Installation

Install dependencies:

```bash
pip install numpy matplotlib brokenaxes dimod dwave-ocean-sdk ortools
```

If using GUROBI:

- Ensure GUROBI is installed locally  
- Ensure OR-Tools is built with GUROBI support (or use SCIP)

---

# 🚀 CQM2BQM (CQM → BQM → Hybrid Quantum Solver)

`CQM2BQM` implements:

- Multi-round Boolean variable construction  
- S-box constraints via a simple arithmetic-string parser  
- Bit-permutation constraints between rounds  
- Objective (min/max) formulation  
- **Automatic CQM → BQM conversion**  
- **Execution on D-Wave Leap Hybrid Sampler**

### ✔ Basic Example

```python
from GenCQMModel import GenCQMModel

model = GenCQMModel(label="cqm-demo")

S_in, S_out = model.add_round(sbox_size=4, n_sboxes=4)

sbox_constraints = [
    "+ a[0] + a[1] - b[0] == 0",
    "+ a[2] + a[3] - b[1] == 1",
]

model.add_sbox_layer(S_in, S_out, sbox_constraints, sbox_size=4, n_boxes=4)

model.set_objective([1]*len(S_out), S_out, sense="min")

best = model.solve(time_limit=5)
print(best)
```

---

# 🔥 Integral Distinguisher Search (Hybrid Quantum)

The quantum model supports iterative distinguisher search:

```python
model.solve_for_integral_distinguisher(S_out, blocksize=16)
```

This procedure:

1. Minimizes active output bits  
2. Fixes discovered 1-bits to 0  
3. Repeats solving  
4. Stops when distinguisher appears (multiple active bits)  

---

# 🚀 MILP_and_CQM (Classical MILP Baseline)

`MILP_and_CQM` is the classical version of the experiment using:

- OR-Tools interface  
- GUROBI / SCIP backend  
- Same string-based constraint parser  
- Same distinguisher search loop  

### ✔ Basic Example

```python
from GenMILPModel import GenMILPModel

milp = GenMILPModel(solver_name="GUROBI")

S_in, S_out = milp.add_round(sbox_size=4, n_sboxes=4)

milp.add_sbox_layer(S_in, S_out, sbox_constraints, 4, 4)

milp.set_objective([1]*len(S_out), S_out, sense="min")

milp.solve_for_integral_distinguisher(S_out)
```

---

# 📊 Figure Reproduction — Broken-Axes Runtime Plots

The plotting script (`makefigures.py`) produces the paper's broken-axis runtime charts.

### ✔ Example dataset format

```python
runs = [
    {"solver": "HYBRID", "blocksize": 16, "time": 0.045, "found": 1},
    {"solver": "MILP",   "blocksize": 16, "time": 0.512, "found": 1},
    ...
]
```

### ✔ Generate figure

```python
from makefigures import plot_time_boxplot_broken

plot_time_boxplot_broken(
    runs,
    blocks=(16, 32, 64),
    solvers=["MILP", "HYBRID", "QPU"],
    save_path="plots/runtime.png"
)

---

# 📜 License

MIT License.


