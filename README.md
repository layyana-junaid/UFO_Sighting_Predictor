<p align="center">
    <img src="https://img.shields.io/badge/status-active-8A2BE2?style=for-the-badge" />
    <img src="https://img.shields.io/badge/python-3.12-4B0082?style=for-the-badge&logo=python&logoColor=white" />
    <img src="https://img.shields.io/badge/model-negative_binomial-C71585?style=for-the-badge" />
    <img src="https://img.shields.io/badge/data-NUFORC-191970?style=for-the-badge" />
  </p>

<h1 align="center">🛸 UFO Sighting Predictor 🌌</h1>
<p align="center"><i>Statistical modeling of unidentified aerial phenomena reports across the United States</i></p>

<p align="center">✨ · 🪐 · 👽 · 🌠 · 🛸 · 🌌 · ✨</p>

---

## 📡 Overview

This project applies exploratory data analysis and statistical modeling to two NUFORC (National UFO Reporting Center) datasets to understand **when, where, and why** UFO sightings get reported — and to build a model that predicts expected sighting rates from location, season, moon phase, and darkness conditions.

This is not a claim about extraterrestrial activity. It's an investigation into **human reporting behavior**: population density, seasonality, and visibility conditions explain the overwhelming majority of the pattern.

---

## 🌍 Datasets

| File | Rows | Scope | Notes |
|---|---|---|---|
| `nuforc_reports.csv` | 141,261 | Global, 1969–2022 | Primary dataset — full narrative reports |
| `ufo_data_nuforc.csv` | 1,317 | US only, 2016–2022 | Curated subset — includes population, images |
| `cities15000.txt` | 23,050 | Global | GeoNames city population lookup (external) |

> ⚠️ Raw data files are excluded from this repository (see `.gitignore`) due to file size. Sources are linked below.

**Data sources:**
- [NUFORC Sighting Reports](https://nuforc.org/)
- [GeoNames Cities Database](https://download.geonames.org/export/dump/)

---

## 🔭 Methodology

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

## 🌠 Hypotheses & Results

| Hypothesis | Prediction | Result | Verdict |
|---|---|---|---|
| Population increases sighting rate | Strong positive effect | p < 0.001 | ✅ Confirmed |
| Summer months (esp. July) peak | July highest | p < 0.001, July strongest coefficient | ✅ Confirmed |
| Moon phase affects sighting rate | No effect | p = 0.448 | ✅ Confirmed (null) |
| Darkness ratio independently affects rate | Strong positive effect | p = 0.270, not significant | ❌ Not confirmed — largely explained by season |

**Key insight:** the well-known "sightings happen at night" pattern is real (81.5% of reports occur after dark), but once *season* is accounted for, darkness ratio adds no further independent predictive power — the effect is absorbed by month.

---

## 🎯 Sample Prediction

```python
predict_sightings(lat=33.4484, lng=-112.0740, month=7)   # Phoenix, AZ — July
>>> 50.5

predict_sightings(lat=33.4484, lng=-112.0740, month=2)   # Phoenix, AZ — February
>>> 28.7
```

---

## 🐛 Notable Engineering Fix

An early model version produced a **13.8 million predicted sightings** anomaly for the NYC metro grid cell. Root cause: raw (non-transformed) population fed into a log-link GLM caused exponential extrapolation blowup on outlier-scale cities. Fixed via `log1p(population)` transformation — a documented example of diagnosing and correcting a real modeling failure mode.

---

## 🛠️ Tech Stack

<p>
  <img src="https://img.shields.io/badge/pandas-150458?style=flat-square&logo=pandas&logoColor=white" />
  <img src="https://img.shields.io/badge/numpy-013243?style=flat-square&logo=numpy&logoColor=white" />
  <img src="https://img.shields.io/badge/statsmodels-8A2BE2?style=flat-square" />
  <img src="https://img.shields.io/badge/matplotlib-3776AB?style=flat-square" />
  <img src="https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikit-learn&logoColor=white" />
  <img src="https://img.shields.io/badge/astral-191970?style=flat-square" />
</p>

---

## 📁 Project Structure

```
EDA_Thingie/
├── eda_1.ipynb          # Primary EDA + modeling (nuforc_reports.csv)
├── eda_2.ipynb          # Secondary EDA (ufo_data_nuforc.csv)
├── eda_3.ipynb          # Cross-dataset validation
├── .gitignore           # Excludes large raw data files
└── README.md
```

---

## 🚀 Future Work

- **Track A:** Explainable-vs-unexplained binary classifier using airport-proximity features (2016+ subset)
- Zero-inflated Negative Binomial to handle sparse rural grid cells
- Full-grid heatmap visualization across all months
- Socioeconomic correlation analysis (unemployment/education), controlled for population and darkness confounds

---

<p align="center">🛸 · 👽 · 🌌 · ✨ · 🪐 · ✨ · 🌌 · 👽 · 🛸</p>
<p align="center"><sub>Built by Layyana Junaid</sub></p>
