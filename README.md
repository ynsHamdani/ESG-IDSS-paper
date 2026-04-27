# A Knowledge-Based ESG Investment Decision Support System for Heterogeneous Investor Groups

**An Opinion Dynamics Approach Based on Non-Additive Measures**
*Ezgi Türkarslan · Younes Hamdani · Mustafa Engin Türkarslan · Christian Oliver Ewald *

---

## ✨ What this repository contains

A complete, research-grade pipeline for **values-based portfolio allocation under ESG**, built around:

* **Linguistic Dominance → DeGroot opinion dynamics** (time-varying, column-stochastic mixing).
* **Fuzzy cross-entropy consensus** (entry-wise, expert × alternative × criterion).
* **LP-SWARA** for **sub-criteria** prioritization.
* **Interaction-aware main-criteria weighting** (super-additive λ-fuzzy measures).
* **2-additive Möbius + Shapley allocation** to translate group preferences into **simultaneous investment rates**.

It reproduces the paper’s results and provides a **dashboard** for interactive exploration.

---

## 🔑 Key contributions (paper highlights)

* **DSS for ESG allocation:** group-aware, explainable, and data + preference driven.
* **Dominance matrix:** drives a **DeGroot-inspired** (time-varying) opinion update.
* **Consensus via fuzzy cross-entropy:** quantifies alignment of each expert with the group.
* **LP-SWARA:** linear-programming–based SWARA for consistent **sub-criteria** weights.
* **Interaction-aware main-criteria weights:** super-additive λ-fuzzy measures (Choquet).
* **Portfolio rates from Shapley values:** derived from a **2-additive Möbius** representation.

---

## 🧭 System at a glance

```
        Raw ESG & Financial Data                   Expert Preferences
     ┌──────────────────────────────┐           ┌──────────────────────┐
     │ /data/esg_scores.csv         │           │ /config/LT.yml       │
     │ /data/financials.csv         │  +       │ /config/epsilon.yml  │
     └──────────────┬───────────────┘           └─────────┬────────────┘
                    │                                     │
         ┌──────────▼─────────┐                ┌──────────▼─────────┐
         │  Algo 3: IDM (per  │                │  Algo 4: Dominance │
         │  expert decisions) │                │  D, Θ from PLTS     │
         └──────────┬─────────┘                └──────────┬─────────┘
                    │                                     │
                    ├─────────────►  Algo 5: Opinion Dynamics  ◄───────┤
                    │            (DeGroot + consensus freeze)          │
                    │                          │                       │
                    ▼                          ▼                       ▼
           Final preference S·, ω·       Choquet scores Sẗ            (if needed)
                                         ─────────────────►  Algo 6: 2-additive Möbius
                                                            + Shapley allocation ν


## 📊 Datasets

**Input schema (minimum):**

* `esg_scores.csv` (wide or long):

  * `company_id`, `C1`, `C2`, … (main criteria scores) **or**
  * `company_id`, `criterion`, `value`
* Optional sub-criteria file for LP-SWARA:

  * `criterion`, `subcriterion`, `value` (or paired comparisons/ordinal SWARA inputs)
* Optional financials (for reporting only): `company_id`, `sector`, `cap`, …

> All scores are normalized internally (min-max per criterion) before subjective/objective fusion.

---

## ⚙️ Configuration

### Linguistic Trust (PLTS) + consensus threshold

`config/LT.yml`

```yaml
revision: 3
epsilon: 0.075        # consensus threshold; lower = stricter
terms: [-3, -2, -1, 0, 1, 2, 3]

LT:
  - # E1’s views [E1,E2,E3]
    - [[0, 0.5], [1, 0.5]]      # E1→E1 (self)
    - [[1, 0.6], [2, 0.4]]      # E1→E2
    - [[-1,1.0]]                # E1→E3
  - # E2’s views
    - [[2, 0.7], [1, 0.3]]      # E2→E1
    - [[0, 1.0]]                # E2→E2
    - [[-1,1.0]]                # E2→E3
  - # E3’s views
    - [[-2, 1.0]]               # E3→E1
    - [[0, 0.5], [1, 0.5]]      # E3→E2
    - [[1, 0.6], [2, 0.4]]      # E3→E3
```

> **Hot-reload:** editing `epsilon` or `LT` and bumping `revision` allows **Algo 5** to refresh **without restarting** (see *Workflow* below).


## 🔄 Workflow (high-level)

1. **Algo 3 — Initial Decision Matrices (IDM)**
   Fuse **subjective** (expert thresholds & ψ·Ω) and **objective** (κ / σ) via sigmoids → per-expert normalized IDM.

2. **Algo 4 — Dominance & Expert Weights**
   From **PLTS (LT.yml)** compute **dominance matrix D** and expert weights **Θ**.

3. **Algo 5 — Opinion Dynamics & Consensus**

   * DeGroot mixing with **column-stochastic D** to evolve per-expert IDMs.
   * Build collective preferences (\tilde S) and compute **fuzzy cross-entropy** consensus.
   * **Freeze** alternatives that pass policy (exact / majority).
   * If unresolved remain, **refresh** (re-read `LT.yml`: **LT + epsilon**) and re-run **Algo 4**; continue until done.

4. **Algo 6 — Rates via 2-Additive Möbius + Shapley**
   Map **final Choquet scores** to a **2-additive capacity**. The **Shapley values** give **investment rates**; pairwise Möbius terms reveal **synergies/redundancies**.

---

## 🧪 Reproducibility tips

* Fix seeds for any randomized preprocessing.
* Log all hyper-parameters: (\lambda, \psi, \kappa, \epsilon), freeze/refresh policies, majority ratio.
* Export run artifacts: `D`, `Θ`, (\tilde S), freeze rounds, final (S\dot{S}), Shapley & pair interactions.

---

## 🖥️ Dashboard (example)

**Pages:**

* **Opinion Dynamics:** per-round (\tilde S), unresolved set, CE heatmaps, freeze timeline.
* **Capacity & Shapley:** λ-measure, Möbius pairs, Shapley rates, Pareto charts.
* **What-if:** edit `epsilon` & PLTS (via `LT.yml` + revision) and **watch convergence**.

---

## 📈 Typical outputs

* **Final preference matrix** ( \dot S ) per alternative × criterion.
* **Choquet scores** ( S\dot{S}_q ) (ranking).
* **Shapley rates** ( \nu(\tilde A_q) ) (simultaneous allocation).
* **Interaction indices** ( m({i,j}) ) (synergy / redundancy insights).

---

## 🧩 Notes for practitioners

* **ε (consensus threshold)** is the *most impactful* knob: larger → easier consensus; smaller → stricter.
* **Refresh policy**:

  * *on_no_consensus* → aggressive; reacts every round if any unresolved remain.
  * *on_stagnation* → reacts only if unresolved set stops changing.
  * *never* → pure DeGroot; should converge if (D) is primitive/aperiodic.
* **Algo 6**: prefer **light interaction** mode first (bounded pairs, small cut budget), then refine if needed.

---

## 📚 Citation

If you use this code or datasets, please cite:

> **A Knowledge-Based ESG Investment Decision Support System for Heterogeneous Investor Groups: An Opinion Dynamics Approach Based on Non-Additive Measures**
> Ezgi Türkarslan, Younes Hamdani, Mustafa Engin Türkarslan, Christian Oliver Ewald,  2026.

---


## 🙌 Acknowledgements

We thank collaborators and institutions supporting this research and tooling. Contributions and issues are welcome—please open a PR or create a discussion thread.


