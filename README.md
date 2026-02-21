# Friday Night at the ER - Hospital Flow Optimization (MSE 433)

This repository contains a mixed-integer optimization implementation of the **Friday Night at the ER** hospital flow problem. The project compares a baseline operating policy against an optimized triage-based policy under shared arrivals, bed limits, staffing limits, and transfer rules.

## Problem Understanding

Hospitals are modeled as connected departments with constrained resources:

- `ER` (Emergency)
- `SURG` (Surgery)
- `ICU` (Critical Care)
- `SD` (Step Down)

Key operational challenges:

- Patients arrive each hour (walk-ins and ambulances).
- Departments have finite rooms and permanent staff.
- Patients may need to transfer across departments before discharge.
- Throughput is limited by hourly admissions and exits.
- Congestion creates waiting queues, diversions, and temporary staffing cost.

The core systems question is: **How should hourly admissions, transfers, and staffing be chosen to minimize total cost while preserving feasible patient flow?**

## Project Goals

1. Build a faithful **Original model** of the game-like hospital system.
2. Build an **Optimized model** with explicit triage routing logic.
3. Quantify cost drivers (waiting, diversion, temporary staff, crowding penalties).
4. Run sensitivity sweeps to identify bottlenecks and robust parameter settings.
5. Produce reproducible plots and tables for comparison.

## What the Models Do

### Original Model (baseline)

- Walk-ins can enter department queues directly.
- Ambulances go to ER.
- Ambulances are diverted when ER is already waiting and effectively saturated (based on model constraints/rule).
- Each hour solves a MIP with decisions for:
  - admissions per department,
  - inter-department transfers,
  - temporary staff by department.
- Objective minimizes:
  - waiting cost,
  - temporary staffing cost,
  - ambulance diversion cost.

### Optimized Model (triage-based)

- Walk-ins are routed through triage first.
- Triage classifies/reroutes patient streams (low/high acuity split parameters).
- High-acuity flow can route toward ER and/or directly to ICU/SURG (per scenario rules).
- ER supports crowding policy (double-bunk penalty term when enabled in scenario).
- Triage staffing is explicitly modeled with permanent + temporary coverage logic.
- Objective extends baseline with optimized-policy penalties/costs such as:
  - triage waiting,
  - triage temporary staffing,
  - triage split penalty (if enabled),
  - ER crowding penalty (if enabled).

## Implementation Approach

The implementation is deterministic and reproducible for direct model-to-model comparison.

1. **Shared data layer**
   - Fixed 24-hour arrival profiles are used for both models.
   - Common capacity/staff/cost definitions are centralized.

2. **Hourly optimization loop**
   - For each hour, a Gurobi MIP is built and solved.
   - Decision variables represent admissions, transfers, and temporary staff.
   - Constraints enforce capacity, staffing ratios, transfer rules, and flow balance.

3. **State update**
   - End-of-hour patients and waiting queues are updated from decision outputs.
   - Cost components are accumulated by hour and by department.

4. **Comparison and diagnostics**
   - Both models are run on identical arrivals.
   - Outputs include total cost, queue levels, diversions, and staffing usage.

## Repository Structure

- `1.ipynb` - primary notebook with model runs, debugging, and visuals.
- `sensitivity.py` - scripted sensitivity experiments (scenario sweeps).
- `sensitivity_results.csv` - output table from sensitivity runs.
- `generate_plots.py` - plotting utility for sensitivity outputs.
- `outputs/` - generated figures and summary markdown.
- `hospital_mip.tex` - LaTeX mathematical formulation.
- `requirements.txt` - Python dependencies for scripts/plots.

## How to Run

Install dependencies:

```bash
python3 -m pip install -r requirements.txt
```

Run sensitivity experiments:

```bash
python3 sensitivity.py
```

Generate plots from `sensitivity_results.csv`:

```bash
python3 generate_plots.py
```

## Notes on Cost Interpretation

Large total cost values are expected when high-penalty queues persist over many hours. The dominant term is typically:

- `patient-hours waiting x waiting penalty`

For example, ICU/SURG/SD waiting penalties are much larger than ER waiting penalties, so sustained queueing there drives totals quickly.

## Validation and Reproducibility

- Same input arrivals for both models.
- Same horizon (`24` hours).
- Explicit per-hour state updates.
- Scenario-based sensitivity runs for robustness checks.

To keep comparisons valid, modify one policy lever at a time (e.g., triage split, max admissions, ready-out rates, staffing assumptions) and re-run both models on the same arrivals.
