# Volve Field Production Performance & Well Optimization

> A Production Engineering analysis of Equinor's open **Volve field** dataset — built to demonstrate production-engineering thinking (not just data science or reservoir simulation) for a Production Engineering portfolio.

This project answers one fundamental question: **Which wells deserve engineering attention, and why?** The goal is not just to describe *what* happened to production, but to diagnose *why* it's happening and outline what a Production Engineer would investigate next.

---

## Project Overview

Volve was a North Sea oil field produced by Equinor (then Statoil) from 2008–2016. The full daily production dataset was released publicly under an open license. This project uses that dataset to run the kind of systematic review a Production Engineer conducts on a real field:

1. **Clean and validate** daily well data, catching real physical data-quality issues, not just handling missing values.
2. **Establish baseline performance** to identify which wells carry the field's production.
3. **Track water cut and decline trends** on a well-by-well basis.
4. **Sanity-check rate drivers**, applying an explicit correlation-vs-causation check on operating conditions.
5. **Build a defensible screening score** to prioritize wells for engineering review.
6. **Write cautious, data-supported recommendations** rather than prescriptive fixes the data can't justify.

*Every analytical step is paired with a **Production Engineering Interpretation** in the notebook — explaining what was done, why it matters, and how an engineer uses the result.*

---

## Dataset Description

* **Source:** Equinor Volve field open dataset, "Daily Production Data" sheet (`data/Volve_production_data.xlsx`).
* **License:** Released by Equinor under CC BY-NC-SA 4.0 for education/research use.
* **Total Scope:** 15,634 raw daily records across 7 wellbores (producers and injectors), Sep 2007 – Dec 2016.
* **Project Scope:** 6 producing wells only (`WELL_TYPE == 'OP'` and `FLOW_KIND == 'production'`) — **F-1 C, F-5 AH, F-11 H, F-12 H, F-14 H, F-15 D**. This totals 9,143 producer rows from Feb 2008 – Sep 2016.
* **Key Fields Used:** `ON_STREAM_HRS`, `AVG_CHOKE_SIZE_P`, `AVG_WHP_P`, `AVG_DOWNHOLE_PRESSURE`, `AVG_DP_TUBING`, `BORE_OIL_VOL`, `BORE_GAS_VOL`, `BORE_WAT_VOL`.

---

## Methodology

| Phase | Actions Taken |
| --- | --- |
| **Cleaning** | Filtered to producer/production records. Clipped 4 immaterial negative water-volume rows to 0. Dropped 1 internally inconsistent row (0 on-stream hours, oil > 0). Verified 13 apparent ">24 hour" days as genuine EU daylight-saving 25-hour calendar days. |
| **Data Quality** | Identified and nulled 1,924 physically impossible zero downhole-pressure readings during flowing days — traced to a **permanent downhole gauge failure on F-12 H** from Oct 2010 onward (~67% of its flowing days). |
| **Field Overview** | Calculated cumulative oil/gas/water and field uptime by well. |
| **Water Cut** | Calculated lifetime volume-weighted water cut and plotted early-vs-late monthly trends per well. |
| **Decline Analysis** | Performed a log-linear fit of monthly oil volume vs. time to establish a simple monthly decline % per well. |
| **Operating Conditions** | Calculated per-well Pearson correlation of choke/WHP/downhole pressure/tubing ΔP vs. on-stream-normalized oil rate. Deliberately **not pooled across wells** to avoid confounding variables. |
| **Screening** | Built a transparent weighted score: Decline (35%) / Water Cut (30%) / Downtime (15%) / Pressure Trend (20%). Normalized 0–1 within the well set and classified as High/Medium/Low priority. |

---

## Key Findings

* **Production Concentration:** **F-12 H and F-14 H together produce ~85% of field oil** (45.6% and 39.3% respectively) — they are the field's plateau wells. F-11 H is a strong secondary contributor (11.4%) with the field's best uptime (92.8%).
* **Water Breakthrough:** Water cut rose sharply on every well with sufficient history — moving from near-0% early in life to 59–97% in recent producing months. This is the dominant story in the dataset, consistent with aggressive water breakthrough from the field's injection scheme.
* **Reservoir Pressure:** Downhole pressure is flat to slightly increasing on wells with valid gauge data. This indicates the oil decline is driven mainly by rising water cut / fractional flow rather than reservoir pressure depletion.
* **Choke vs. Rate Confounding:** Choke opening is negatively correlated with oil rate on 4 of 6 wells. This is a time-confounded relationship (chokes were progressively opened as the field aged and rates fell), not evidence that opening the choke actively reduces rate.
* **Well Prioritization:** The screening model flags **F-1 C, F-12 H, and F-15 D as High Priority** for three distinct reasons (steep decline/low uptime, massive water-handling scale on the top producer, and a moderate signal on a small well needing confirmation).

---

## Engineering Insights

* **Look for physical reality, not just missing data:** A standard data-quality check (is a value *physically possible*) surfaced a real instrumentation issue — the F-12 H downhole gauge failure — independent of the production analysis itself.
* **Holistic diagnostics:** Reading rate, pressure, and water cut together changes the diagnosis. Pressure alone suggests a healthy reservoir; oil rate alone suggests across-the-board decline. Together, they clearly point to water breakthrough as the primary driver.
* **Beware of causation traps:** Correlation results were checked well-by-well and against time specifically to avoid presenting a confounded relationship (choke size vs. declining field rate) as causal.

---

## Technologies

* Python, pandas, NumPy, Matplotlib, Jupyter Notebook

## Future Work

* **Accumulate History:** Re-screen F-15 D and F-5 AH once more producing history accumulates (both currently have limited data behind their scores).
* **Injection Allocation:** Incorporate injection-allocation data (if available) to confirm which injector(s) are driving water breakthrough into which producers.
* **Advanced Decline Curve Analysis (DCA):** Extend the log-linear decline analysis to a proper Arps hyperbolic/harmonic fit per well for remaining-life estimates.
* **Maintenance Workflows:** Flag the F-12 H downhole gauge failure to a surveillance/instrumentation workflow for repair or artificial lift evaluation.
