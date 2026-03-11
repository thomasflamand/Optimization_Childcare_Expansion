# 🧒 Eliminating Child Care Deserts in New York State — Optimization Models

> **IEOR E4004: Optimization Models and Methods — Project I**
> Columbia University | Group 7 | October 2025

---

## 📋 Overview

This project develops **Mixed-Integer Linear Programming (MILP)** models to determine the minimum public investment required for New York State to eliminate child care deserts — regions where licensed child care capacity falls far short of demand.

A child care desert is defined as:
- **High-demand ZIP** (employment ≥ 60% or avg. income ≤ $60k): available slots ≤ 50% of children aged 0–12
- **Normal-demand ZIP**: available slots ≤ 1/3 of children aged 0–12
- Additionally, slots for children aged 0–5 must cover at least 2/3 of that population in every ZIP

---

## 🗂️ Project Structure

```
ChildCareDeserts_Data/
├── avg_individual_income.csv
├── employment_rate.csv
├── population.csv
├── child_care_regulated.csv
└── potential_locations.csv

Part1_Ideal.ipynb          # Idealistic budgeting model (up to 120% expansion)
Part1_Realistic.ipynb      # Realistic expansion model (20% cap, tiered costs)
Part2_Realistic.ipynb      # Spatial feasibility model (distance constraints)
Fairness_Problem.ipynb     # Optional fairness extension
```

---

## 🧮 Models

### Part 1 — Ideal Budgeting

Determines the **minimum investment** to eliminate all deserts assuming no geographic constraints on new construction.

**Key features:**
- Existing facilities may expand up to `min(120% × capacity, +500 slots)`
- Per-slot expansion cost depends on facility size (\\$160–\\$200/slot below 100%; \\$200/slot + \\$20k fixed fee above 100%)
- New facilities in three sizes: **S** (100 slots, \\$65k), **M** (200 slots, \\$95k), **L** (400 slots, \\$115k)
- Additional \\$100/slot equipment cost for any 0–5 slot created

**Result: \\$348,568,508.81**

---

### Part 1 — Realistic (20% cap)

Same model but expansion is capped at **20% of current capacity**, with piecewise increasing costs:

| Expansion tier | Marginal cost |
|---|---|
| 0–10% of capacity | \\$200/slot + \\$20k base |
| 10–15% | \\$400/slot + \\$20k base |
| 15–20% | \\$1,000/slot + \\$20k base |

**Result: \\$519,250,157.09** (+49% vs. ideal)

---

### Part 2 — Realistic + Spatial Feasibility

Adds a **minimum distance constraint** of 0.06 miles between any two facilities (new or existing). Candidate sites are filtered using:

1. Restriction to desert ZIP codes only
2. BallTree nearest-neighbor search (haversine metric) to remove sites too close to existing facilities
3. Greedy spatial thinning to ensure mutual spacing among remaining candidates

This reduces hundreds of thousands of raw candidate sites to a small, tractable set per ZIP, avoiding O(N²) pairwise distance constraints in the MILP.

**Result: \\$391,626,999.00** — *lower than the 20%-cap model*, because the spacing rule implicitly consolidates capacity into fewer, larger, better-placed facilities.

---

### Part 3 — Fairness Extension (Optional)

Introduces an equity objective via the **weighted coverage ratio**:

$$\text{cov}_z = \frac{2}{3} \cdot \frac{\text{Slots}_{0\text{–}5,z}}{\text{Pop}_{0\text{–}5,z}} + \frac{1}{3} \cdot \frac{\text{Slots}_{0\text{–}12,z}}{\text{Pop}_{0\text{–}12,z}}$$

**Fairness gap** = max coverage − min coverage across all ZIPs. Target: gap ≤ 0.10.

| Phase | Objective | Result |
|---|---|---|
| Phase A | Minimize gap under \\$100M budget | Infeasible — gap target unachievable |
| Phase B | Minimum cost to achieve gap ≤ 0.10 | ≈ \\$243.5 billion |

> **Key insight:** Eliminating deserts ≠ achieving equal access. True geographic equity requires transformative, large-scale investment.

---

## 📊 Summary of Results

| Scenario | Optimal Cost |
|---|---|
| Part 1 — Ideal | \\$348,568,508 |
| Part 1 — Realistic (20% cap) | \\$519,250,157 |
| Part 2 — Realistic + Spatial | \\$391,626,999 |
| Fairness target (gap ≤ 0.10) | ≈ \\$243.5 billion |

---

## ⚙️ Setup & Requirements

```bash
pip install gurobipy pandas numpy scikit-learn
```

A valid **Gurobi license** is required. Academic licenses are available free at [gurobi.com](https://www.gurobi.com/academia/academic-program-and-licenses/).

> Models were solved using Gurobi 12.0.3 on Apple M3 (8 cores). Solve times range from under 1 second to ~1.3 seconds depending on the model.

---

## 🚀 Usage

Run each notebook in order:

```bash
jupyter notebook Part1_Ideal.ipynb
jupyter notebook Part1_Realistic.ipynb
jupyter notebook Part2_Realistic.ipynb
jupyter notebook Fairness_Problem.ipynb
```

Each notebook is self-contained and loads the data from `./ChildCareDeserts_Data/`.

---

## 🔬 Technical Highlights

- **MILP formulation** via `gurobipy` with indicator constraints for piecewise cost regimes
- **BallTree spatial indexing** (haversine metric) for scalable distance filtering
- **Greedy spatial thinning** to pre-process candidate locations before optimization
- **Lexicographic multi-objective** optimization for the fairness phase

---

## 📄 License

This project was developed for academic purposes at Columbia University. Data is provided as part of the course.
