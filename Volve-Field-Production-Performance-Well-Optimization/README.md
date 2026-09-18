# Volve Field Production Performance & Well Optimization

A Production Engineering analysis of Equinor's open **Volve field** dataset — built to demonstrate
production-engineering thinking (not data science or reservoir simulation) for a Production
Engineering interview.

The project answers one question: **which wells deserve engineering attention, and why?** — not
just "what happened to production," but why it's happening and what a Production Engineer would
investigate next.

---

## Project Overview

Volve was a North Sea oil field produced by Equinor (then Statoil) from 2008–2016, with the full
daily production dataset released publicly under an open license. This project uses that dataset to
run the kind of review a Production Engineer would run on a real field:

1. Clean and validate the daily well data (catching real data-quality issues, not just missing values)
2. Establish which wells actually carry the field's production
3. Track water cut and decline trends well-by-well
4. Sanity-check what's driving oil rate — with an explicit correlation-vs-causation check
5. Build a transparent, defensible screening score to prioritize wells for engineering review
6. Write cautious, data-supported recommendations — not prescriptive fixes the data can't justify

Every analytical step is followed by a **Production Engineering Interpretation** — what was done,
why it matters, and how an engineer would use the result.

## Dataset Description

- **Source:** Equinor Volve field open dataset, "Daily Production Data" sheet
  (`data/Volve_production_data.xlsx`)
- **License:** Equinor released the Volve dataset under CC BY-NC-SA 4.0 for education/research use
- **Scope:** 15,634 raw daily records across 7 wellbores (producers and injectors), Sep 2007 – Dec 2016
- **In scope for this project:** 6 producing wells only (`WELL_TYPE == 'OP'` and
  `FLOW_KIND == 'production'`) — **F-1 C, F-5 AH, F-11 H, F-12 H, F-14 H, F-15 D** — 9,143 producer
  rows, Feb 2008 – Sep 2016
- **Key fields:** `ON_STREAM_HRS`, `AVG_CHOKE_SIZE_P`, `AVG_WHP_P`, `AVG_DOWNHOLE_PRESSURE`,
  `AVG_DP_TUBING`, `BORE_OIL_VOL`, `BORE_GAS_VOL`, `BORE_WAT_VOL`

## Methodology

| Step | What was done |
|---|---|
| **Cleaning** | Filtered to producer/production records; clipped 4 immaterial negative water-volume rows to 0; dropped 1 internally-inconsistent row (0 on-stream hours, oil > 0); verified 13 apparent ">24 hour" days as genuine EU daylight-saving 25-hour calendar days (kept); identified and nulled 1,924 physically-impossible zero downhole-pressure readings during flowing days — traced to a **permanent downhole gauge failure on F-12 H from Oct 2010 onward** (~67% of its flowing days) plus isolated blips on other wells |
| **Field overview** | Cumulative oil/gas/water and uptime by well |
| **Water cut** | Lifetime volume-weighted water cut, plus early-vs-late monthly trend per well |
| **Decline analysis** | Log-linear fit of monthly oil volume vs. time → simple monthly decline % per well |
| **Operating conditions** | Per-well Pearson correlation of choke/WHP/downhole pressure/tubing ΔP vs. on-stream-normalized oil rate, deliberately **not pooled across wells** to avoid confounding; explicit discussion of a time-confounded choke-vs-rate correlation |
| **Screening** | Transparent weighted score (decline 35% / water cut 30% / downtime 15% / pressure trend 20%), normalized 0–1 within the well set, classified High/Medium/Low priority |

## Key Findings

- **F-12 H and F-14 H together produce ~85% of field oil** (45.6% and 39.3% respectively) — the
  field's plateau wells. F-11 H is a strong secondary contributor (11.4%) with the field's best
  uptime (92.8%).
- **Water cut rose sharply on every well with sufficient history** — from near-0% early in each
  well's life to 59–97% in recent producing months — the dominant story in the dataset, consistent
  with water breakthrough from the field's injection scheme.
- **Downhole pressure is flat to slightly increasing**, not declining, on wells with valid gauge
  data — indicating the oil decline is driven mainly by rising water cut / fractional flow rather
  than reservoir pressure depletion.
- **Choke opening is negatively correlated with oil rate on 4 of 6 wells** — read as a time-confounded
  relationship (chokes progressively opened as the field aged and rates fell), not evidence that
  opening the choke reduces rate. A deliberate correlation-vs-causation check, not a naive finding.
- **Screening flags F-1 C, F-12 H, and F-15 D as High Priority** — for three different reasons
  (steep decline/low uptime, water-handling scale on the top producer, and a moderate signal on a
  small well that needs confirmation), each explained rather than just ranked.

## Engineering Insights

- A data-quality check (is a value *physically possible*, not just non-null) surfaced a real
  instrumentation issue — the F-12 H downhole gauge failure — independent of the production
  analysis itself.
- Reading rate-vs-pressure-vs-water-cut together, rather than any one metric alone, changes the
  diagnosis: pressure alone would suggest a healthy reservoir; oil rate alone would suggest
  across-the-board decline. Together, they point to water breakthrough as the primary driver.
- Correlation results were checked well-by-well and against time, specifically to avoid presenting
  a confounded relationship as causal — a common analysis trap.

## Technologies Used

Python · pandas · NumPy · Matplotlib · Jupyter Notebook

## Results

All screening outputs, summary tables, and figures are reproducible from `notebooks/Volve_Field_Production_Performance_Well_Optimization.ipynb`, and saved separately in `data/*.csv` and `figures/*.png`.

## Future Work

- Re-screen F-15 D and F-5 AH once more producing history accumulates (both currently have limited
  data behind their scores)
- Incorporate injection-allocation data, if available, to confirm which injector(s) are driving
  water breakthrough into which producers
- Extend the decline analysis to a proper Arps hyperbolic/harmonic fit per well for remaining-life
  estimates
- Flag the F-12 H downhole gauge failure to a surveillance/instrumentation workflow for repair

---

## Repository Structure

```
Volve-Field-Production-Performance-Well-Optimization/
│
├── data/
│   ├── Volve_production_data.xlsx        # Raw source data
│   ├── producers_clean.csv                # Cleaned producer-only daily data
│   ├── well_overview_summary.csv          # Section 4 output
│   ├── water_cut_trend.csv                # Section 5 output
│   ├── monthly_by_well.csv                # Section 6 monthly aggregates
│   ├── decline_analysis.csv               # Section 6 output
│   ├── operating_condition_correlations.csv  # Section 7 output
│   ├── pressure_trend.csv                 # Section 7 output
│   └── well_screening.csv                 # Section 8 output
├── notebooks/
│   └── Volve_Field_Production_Performance_Well_Optimization.ipynb
├── figures/
│   ├── 01_field_overview.png
│   ├── 02_water_cut_trend.png
│   ├── 03_monthly_production_trends.png
│   ├── 04_operating_conditions_scatter.png
│   ├── 05_downhole_pressure_trend.png
│   └── 06_well_screening_scores.png
├── README.md
├── requirements.txt
└── presentation/
    └── (summary slide content — see RESUME_BULLETS.md / INTERVIEW_PREP.md for talking points)
```

## Setup

```bash
pip install -r requirements.txt
jupyter notebook notebooks/Volve_Field_Production_Performance_Well_Optimization.ipynb
```

## Data Source & License

Volve field data released by Equinor under CC BY-NC-SA 4.0. This project uses it for educational,
non-commercial portfolio purposes only.
