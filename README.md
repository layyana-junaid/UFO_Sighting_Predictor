![banner](ufobanner.png)

<h1 align="center">🛸 UFO Sighting Predictor 🌌</h1>
<p align="center"><i>Statistical modeling of unidentified aerial phenomena reports across the United States</i></p>

<p align="center">✨ · 🪐 · 👽 · 🌠 · 🛸 · 🌌 · ✨</p>

---

## Overview

This project applies exploratory data analysis and statistical modeling to two NUFORC (National UFO Reporting Center) datasets to understand **when, where, and why** UFO sightings get reported — and to build a model that predicts expected sighting rates from location, season, moon phase, and darkness conditions.

This is not a claim about extraterrestrial activity. It's an investigation into **human reporting behavior**: population density, seasonality, and visibility conditions explain the overwhelming majority of the pattern.

---

## Datasets

| File | Rows | Scope | Notes |
|---|---|---|---|
| `nuforc_reports.csv` | 141,261 | Global, 1969–2022 | Primary dataset — full narrative reports |
| `ufo_data_nuforc.csv` | 1,317 | US only, 2016–2022 | Curated subset — includes population, images |
| `cities15000.txt` | 23,050 | Global | GeoNames city population lookup (external) |

> Raw data files are excluded from this repository (see `.gitignore`) due to file size. Sources are linked below.

**Data sources:**
- [NUFORC Sighting Reports](https://nuforc.org/)
- [GeoNames Cities Database](https://download.geonames.org/export/dump/)

---

## Methodology

### 1. Exploratory Data Analysis
- Structure, missingness, and duplicate profiling across both datasets
- Temporal patterns (hour / day-of-week / month)
- Geographic distribution and mapping (country → state → city → lat/lng)
- Free-text `duration` field parsed into standardized seconds via regex classification
- Shape distribution and text-frequency analysis on report summaries
- Missingness-correlation diagnostics
- Cross-dataset validation (overlap + distribution agreement check)

### 2. Astronomical Enrichment
Using the `astral` library — pure orbital-mechanics calculation, no external API:
- **Moon phase** (New / Waxing / Full / Waning) computed per sighting date
- **Daylight status** (Day / Twilight / Night) computed per sighting location + time

### 3. Predictive Modeling — Sighting Rate Estimation

| Step | Approach |
|---|---|
| Target | Sighting count, aggregated per 0.5° grid cell, per month |
| Baseline model | Poisson Regression |
| Overdispersion check | Deviance / df = **9.76** → Poisson assumption violated |
| Final model | **Negative Binomial Regression** (`statsmodels`) |
| Features | `log(population)`, `month`, `moon_phase`, `pct_night` |

---

## Models Used & Why

The target variable — sighting count per grid cell — is a **non-negative integer count**, not a continuous value. This single fact drives every modeling decision below.

| Model | Considered? | Verdict |
|---|---|---|
| **Linear Regression** | ❌ Rejected | Assumes a continuous, unbounded target — can predict nonsensical negative sighting counts. Wrong tool for count data. |
| **Poisson Regression** | ✅ Fit as baseline | The standard model for count data, but assumes **mean = variance**. Used specifically to *test* that assumption. |
| **Negative Binomial Regression** | ✅ **Final model** | Poisson's mean=variance assumption failed (dispersion ratio 9.76 — real variance ~10× higher than Poisson expects, driven by extreme hotspot cities like Phoenix/NYC vs. mostly-sparse rural cells). Negative Binomial adds a dispersion parameter to correctly model this, without changing the interpretable, coefficient/p-value-based output needed for hypothesis testing. |
| **Logistic Regression** | 🔜 Reserved for Track A | Correct tool for a *binary* question ("is this sighting explainable, yes/no") — not used here since this track predicts a count, not a category. |
| **XGBoost** | ⏸️ Considered, not used | Could offer better raw predictive accuracy and captures non-linear interactions automatically, but sacrifices the interpretable p-values that this project's hypothesis-testing goal depends on. Statsmodels' GLM output directly answers "does population/season/moon/darkness significantly matter?" — XGBoost only gives feature importance, a weaker, non-statistical signal. Kept as a possible future accuracy benchmark, not the core deliverable. |

**Library choice:** `statsmodels` (not `scikit-learn`) — deliberately chosen because this project's goal is *explanation* (which factors significantly drive sighting rates, with p-values) rather than pure prediction accuracy. `statsmodels` surfaces coefficients, standard errors, and p-values directly; `scikit-learn` optimizes for prediction and doesn't expose this by default.

---

## Hypotheses & Results

| Hypothesis | Prediction | Result | Verdict |
|---|---|---|---|
| Population increases sighting rate | Strong positive effect | p < 0.001 | ✅ Confirmed |
| Summer months (esp. July) peak | July highest | p < 0.001, July strongest coefficient | ✅ Confirmed |
| Moon phase affects sighting rate | No effect | p = 0.448 | ✅ Confirmed (null) |
| Darkness ratio independently affects rate | Strong positive effect | p = 0.270, not significant | ❌ Not confirmed — largely explained by season |

**Key insight:** the well-known "sightings happen at night" pattern is real (81.5% of reports occur after dark), but once *season* is accounted for, darkness ratio adds no further independent predictive power — the effect is absorbed by month.

---

## Sample Prediction

```python
predict_sightings(lat=33.4484, lng=-112.0740, month=7)   # Phoenix, AZ — July
>>> 50.5

predict_sightings(lat=33.4484, lng=-112.0740, month=2)   # Phoenix, AZ — February
>>> 28.7
```

---

## Tech Stack

<p>
  <img src="https://img.shields.io/badge/pandas-150458?style=flat-square&logo=pandas&logoColor=white" />
  <img src="https://img.shields.io/badge/numpy-013243?style=flat-square&logo=numpy&logoColor=white" />
  <img src="https://img.shields.io/badge/statsmodels-8A2BE2?style=flat-square" />
  <img src="https://img.shields.io/badge/matplotlib-3776AB?style=flat-square" />
  <img src="https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikit-learn&logoColor=white" />
  <img src="https://img.shields.io/badge/astral-191970?style=flat-square" />
</p>

---


<p align="center">🛸 · 👽 · 🌌 · ✨ · 🪐 · ✨ · 🌌 · 👽 · 🛸</p>
<p align="center"><sub>Built by Layyana Junaid</sub></p>
