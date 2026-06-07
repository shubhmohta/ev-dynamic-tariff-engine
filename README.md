# ⚡ Algorithmic Tariff Optimization for EV Charging Grids
> A Closed-Loop Machine Learning & Behavioral Economics Pipeline

**Institution:** Indian Institute of Technology Roorkee — Department of Electrical Engineering  
**Scholar:** Shubh Mohta (`24115142`)

---

## 📌 Overview

Unmanaged EV charging creates localized grid congestion and negative profit margins for operators during peak wholesale energy hours. This project addresses the critical intersection of **energy infrastructure** and **quantitative finance** by proposing a closed-loop, algorithmic pricing architecture.

Rather than relying on static Time-of-Use (TOU) pricing, a **3-agent autonomous pipeline** is engineered:

1. A **Multi-Head LightGBM Oracle** predicts physical grid stress 15 minutes into the future.
2. A **Tariff Pricing Agent** dynamically adjusts consumer prices to shape demand via a simulated behavioral elasticity model.
3. A **Monitoring & Learning Agent** evaluates system-wide outcomes against a flat-rate baseline.

A deterministic mathematical safety floor overrides the AI to guarantee positive unit-level profitability — executing a profitable arbitrage strategy between wholesale TOU costs and dynamic consumer pricing.

---

## 📂 Repository Structure

```
├── PPT_DECK.pdf                  # Executive presentation (Problem statement, EDA, architecture)
├── README.md                     # Project documentation and metric scorecard
├── engine.ipynb                  # Core notebook: data pipelines and agent logic
└── master_project_metrics.csv    # Exported logs: session-by-session financial/physical outcomes
```

---

## 🗄️ Data & Feature Engineering

The pipeline is trained and validated on a **dual-environment dataset**, capturing both economic and physical realities of EV networks:

| Dataset | Purpose |
|---|---|
| **ACN Dataset** | Baseline charging behaviors, session durations, unit economics |
| **UrbanEV Dataset** | Massive-scale node telemetry, spatial constraints, physical queue formations |

### Key Preprocessing Implementations

- **Time-Series Memory** — Engineered 1-hour rolling momentums (`load_rolling_sum_1hr`) and autoregressive lags ($t-1$, $t-2$, $t-3$) to allow tabular models to capture the physical velocity of the charging grid.
- **Strict Chronological Validation** — Random shuffling bypassed in favor of a strict **7-Day Chronological Holdout** to eliminate look-ahead bias and simulate a true production environment.
- **Memory Optimization** — Aggressive data downcasting (e.g., `float64 → float32`, `int8` for cyclical features) to prevent memory blowouts on massive 4D dynamic tensors.

---

## ⚙️ System Architecture: The 3-Agent Pipeline

```
┌─────────────────────────────────────────────────────────┐
│                    INPUT: Grid Telemetry                │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│          Agent 1 — Demand Prediction (The Oracle)       │
│  Multi-Head LightGBM · 15-min ahead forecast            │
│  Outputs: Utilization Rate · Energy Volume · Congestion │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│        Agent 2 — Tariff Pricing (The Executioner)       │
│  3-Tier Policy Engine · Elasticity Simulator (E = -0.5) │
│  Safety Floor: Price_consumer ≥ Cost_grid + Margin_min  │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│      Agent 3 — Monitoring & Learning (The Auditor)      │
│  Isolates intervened sessions · Computes deltas vs.     │
│  flat-rate baseline across queue, volume, and finance   │
└─────────────────────────────────────────────────────────┘
```

### Agent 1 — Demand Prediction (The Oracle)

- **Architecture:** Multi-Head LightGBM optimized for massive tabular time-series data
- **Function:** Predicts grid states (Utilization Rate, Energy Volume, Congestion) **15 minutes ahead**
- **Interpretability:** Feature importance reveals the model acts as a **physics engine**, heavily weighting short-term rolling loads (`load_rolling_sum_1hr`) over simple time-of-day features

### Agent 2 — Tariff Pricing (The Executioner)

- **3-Tier Policy Engine:** Maps Oracle predictions to deterministic pricing actions:
  - 🟢 **Tier 1 — Discount:** Empty/underutilized grids → attract demand
  - 🟡 **Tier 2 — Neutral:** Baseline pricing for normal load
  - 🔴 **Tier 3 — Surge:** Saturated grids → suppress demand
- **Demand Elasticity Simulator:** Models human behavioral response via standard Price Elasticity of Demand ($E = -0.5$)
- **The Safety Floor:** Hard mathematical constraint — `Price_consumer ≥ Cost_grid + Margin_min` — prevents loss-leading operations during wholesale spikes

### Agent 3 — Monitoring & Learning (The Auditor)

- Acts as the objective evaluator, isolating intervened sessions
- Calculates exact deltas in queue lengths, volume shifts, and financial efficiency vs. a flat-rate baseline

---

## 📊 Results Scorecard

| Metric | Focus Area | Result | Interpretation |
|---|---|---|---|
| **Revenue Gain** | Financial Health | **+3.33%** | Proves Safety Floor efficacy — off-peak discounts offset by surge revenue, expanding unit-level margins |
| **Peak Charger Utilization** | Grid Optimization | **−31.38%** | Highly successful peak-shaving; baseline was at 98.26% (cascade failure territory), reduced to 66.88% |
| **Off-Peak Demand Uplift** | Demand Shifting | **−3.64%** | Realistic reflection of human elasticity limits — surge aversion is strong; off-peak discounts cannot override circadian behavior |

---

## ⚠️ Limitations & Transparency

In compliance with rigorous data science standards, results are subject to the following assumptions:

**1. Simulated Elasticity (Not Causal Inference)**  
The reported physical volume shift is a *simulated* response. Live A/B testing on a physical grid was not feasible; human response to price surges was modeled deterministically using $E = -0.5$. Actual load-shifting will fluctuate based on local socioeconomic factors and user urgency.

**2. Uniform Rationality Assumption**  
The Pricing Agent assumes a uniform response across the entire customer base. In reality, **commercial fleets are highly inelastic** compared to casual drivers. Future iterations should incorporate a user-segmentation classifier for dynamic, personalized elasticity functions.

**3. Zero-Friction Environment**  
The simulation assumes perfect market information (instant tariff awareness via app) and zero geographic friction. The reported peak-shaving effect represents a **theoretical maximum efficiency bound**, not a guaranteed production baseline.

---

## 🔭 Future Work

- [ ] User-segmentation classifier for personalized elasticity modeling (fleet vs. casual)
- [ ] Live A/B testing integration to replace simulated behavioral response
- [ ] Reinforcement learning layer on top of the Pricing Agent for adaptive policy updates
- [ ] Multi-node spatial optimization accounting for geographic charging station distribution
- [ ] Real-time wholesale price feed integration for dynamic margin computation

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![LightGBM](https://img.shields.io/badge/LightGBM-Gradient%20Boosting-5b9bd5?style=flat)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat&logo=jupyter&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Processing-150458?style=flat&logo=pandas&logoColor=white)

---

*Department of Electrical Engineering · IIT Roorkee*
