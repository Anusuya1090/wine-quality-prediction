# United Airlines Gate Utilization Optimization at Denver International Airport

**How many gates does United really need at its largest hub?** This project uses linear optimization to find the minimum number of gates United Airlines needs each day to run its July 2025 departure schedule at Denver International Airport (DEN) without gate-time conflicts.

**Tools:** Python, pandas, Gurobi Optimizer, matplotlib, seaborn
**Course:** MSBC 5680, University of Colorado Boulder, Fall 2025
**Team:** Anthony Navarro, Lauren Yoder, Anusuya Roysarkar

---

## Business Problem
Gates at DEN are limited and expensive to expand. United operates roughly 90 gates there, mostly in Concourse B. Understanding true peak gate demand helps the airline plan schedules, spot congestion, and make better long-term capacity and capital decisions.

## Data
- **Source:** U.S. Bureau of Transportation Statistics (BTS), [Detailed Statistics – Departures](https://www.transtats.bts.gov/ONTIME/Departures.aspx)
- **Scope:** All United Airlines departures from DEN, July 2025
- **Fields used:** carrier, date, flight number, tail number, destination, scheduled departure time

> The raw CSV is not included in this repo. Download it from BTS (Carrier = UA, Airport = DEN, July 2025) and save it as `Detailed_Statistics_Departures.csv` next to the notebook.

## Approach
1. Combined date and scheduled time into a departure timestamp for every flight.
2. Defined a **gate-occupancy window** for each departure: 45 minutes before to 15 minutes after scheduled departure (boarding, loading, fueling, pushback).
3. Mapped every window onto a **minute-by-minute timeline** for each day to count overlapping flights.
4. Solved a small **integer optimization model** in Gurobi for each of the 31 days:
   - **Decision variable:** K = number of gates needed that day (integer)
   - **Objective:** minimize K
   - **Constraint:** K ≥ number of flights occupying gates at every minute t
5. Ran a **sensitivity analysis** on the before/after buffer assumptions.

> A full flight-to-gate assignment model was the original goal, but it exceeded the academic Gurobi license limits. The daily capacity model answers the same capacity question with a fraction of the variables.

## Key Results
- United needs only **48–52 gates per day** for departures, about **55% of its ~90 gates**.
- Gate demand is driven by **how departures cluster in time, not by total flight volume**. July 5 had fewer flights than many days but needed the most gates (52) because departures were bunched together.
- Results are **robust to moderate assumption changes**: longer buffers such as (45, 30) or (60, 15) raise the peak by just 1 gate (to 53). Only very conservative buffers push demand to 69–90 gates.

| Daily minimum gates | Flights vs. gates needed |
|---|---|
| ![Daily minimum gates](images/daily_min_gates.png) | ![Flights vs gates](images/flights_vs_gates.png) |

| Weekday patterns | Sensitivity to buffer assumptions |
|---|---|
| ![Weekday trends](images/weekday_trends.png) | ![Sensitivity heatmap](images/sensitivity_heatmap.png) |

## Recommendations
1. **Use peak-overlap patterns** to pinpoint the hours where the schedule is most strained.
2. **Smooth peak departure banks** where feasible to lower peak gate demand and free capacity.
3. **Use the baseline in long-term gate planning** and capital decisions before investing in expansion.

## Limitations
- Models departures only; arrivals, delays, and irregular operations would raise real gate usage.
- Occupancy buffers are assumptions, not an industry standard (tested via sensitivity analysis).
- Treats all gates as interchangeable (no aircraft-size or international-gate restrictions).

## Files
| File | Description |
|---|---|
| `united_gate_optimization.ipynb` | Data prep, Gurobi model, sensitivity analysis, and charts |
| `Project_Report.pdf` | Full written report with methodology, results, and appendix |
| `images/` | Charts used in this README |

## How to Run
```bash
pip install gurobipy pandas matplotlib seaborn
jupyter notebook united_gate_optimization.ipynb
```
A free Gurobi academic or restricted license is enough for this model.
