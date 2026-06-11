# SSA Yield Uncertainty — Risk-Adjusted Returns to Fertilizer Across Sub-Saharan Africa

**A spatially explicit modeling framework that estimates site-specific maize yield response to fertilizer and the risk-adjusted profitability of fertilizer investment across Sub-Saharan Africa, under joint climate and price uncertainty.**

> **🚧 Code release pending.** The analysis behind this repository is still being finalised. This repository currently documents *what* we model and *how* the pipeline is structured. The full R/Python pipeline, configuration, and final result maps will be published here once the working paper is complete. Watch/star the repo to be notified.

---

## 1. What is this project, in one paragraph?

Fertilizer recommendations in Sub-Saharan Africa are usually built on *expected* yield response at *average* prices. But smallholders farm rainfed land with erratic rainfall and sell into volatile markets — what matters to them is not the average return but the **distribution** of returns, including the chance of losing money. This project builds, for every maize-growing pixel in Sub-Saharan Africa, that full distribution: it combines agronomic trial data, soil and climate layers, and predicted local input and output prices, then jointly simulates rainfall and maize-price realisations with a spatial copula Monte Carlo engine to map **where, and at what application rate, fertilizer is a good bet — not just a good average**. By separating climate risk from price risk and mapping high- and low-response zones, the framework supports precision targeting of subsidies, extension, insurance, and complementary interventions.

## 2. Why this matters

- **Fertilizer use in SSA is strikingly low** — around **17 kg of nutrients per hectare** against a global average of ~135 kg/ha, and well short of the 2006 Abuja Declaration target of 50 kg/ha.
- **Profitability, not awareness, is the binding constraint in much of the region.** Fertilizer is expensive at the farm gate while crop prices are low; in Ghana the value–cost ratio for maize fertilizer fell from ~6.8 in the 1980s to ~2.2 in the early 2000s, and recent farm surveys found fertilizer fully profitable (VCR ≥ 2) for only ~28 % of farmers, with nearly half not breaking even.
- **Risk changes the calculus drastically.** Farmers buy fertilizer months before harvest in rainfed systems — essentially betting on the season. A recent continental analysis (McCullough et al. 2022, *Nature Food*) estimated that in about a quarter of rainfed maize areas in SSA, farmers cannot expect even a 30 % return on fertilizer in at least 70 % of seasons.
- **Blanket recommendations make this worse.** One-size-fits-all rates ignore the enormous spatial heterogeneity in soils, rainfall, and prices — oversupplying nutrients where response is weak and undersupplying where it is strong.

The right question for a farmer, an extension service, or a subsidy designer is therefore site-specific and probabilistic: **on this field, at this fertilizer price and this likely maize price, what application rate maximises risk-adjusted profit — and what is the chance of a loss?** That is the question this framework answers, pixel by pixel.

## 3. What we model

The framework couples three estimated surfaces — yield response, input/output prices, and production risk — inside a stochastic profit simulation:

```
profit(pixel, N rate)  =  price_maize(pixel, draw) × yield(pixel, N, rainfall draw)
                        −  price_N(pixel) × N rate
                        −  application & financing costs
```

evaluated over many joint draws of seasonal rainfall and maize price, preserving their spatial structure and their dependence.

### Pipeline stages

| Stage | What it does |
|---|---|
| **0 — Climate data** | Google Earth Engine export of CHIRPS daily rainfall (1981–2024), aggregated to maize growing-season indicators per pixel. |
| **1 — Base layers & rainfall indicators** | SSA country mask (GADM), iSDA Africa soil property rasters, and derived season-rainfall indicators including drought (< 50 % of normal) and excess-rain (> 150 % of normal) flags. |
| **2 — Price models** | Cleaning and spatial prediction of fertilizer (nitrogen) prices from the AfricaFertilizer 2010–2024 price database, and of maize producer prices from World Bank RTFP / WFP monthly market data (2007–2024) — including a maize price-volatility surface. Farmgate prices decay with travel time to the nearest market. |
| **3 — Yield response** | Random-forest yield-response models for nitrogen and phosphorus, trained on the CAROB-compiled multi-location agronomic-trial database enriched with soil and seasonal-rainfall covariates, with genetic-algorithm rate optimisation and model-explainability diagnostics (DALEX/iml). Trial-station yields are calibrated downward (~0.65×) to reflect the documented 30–50 % trial-versus-smallholder yield gap. |
| **4 — Crop-loss probability** | Per-pixel probability of severe crop loss estimated from two independent sources: household-survey reports and an NDVI-anomaly-based classifier. |
| **5 — Monte Carlo risk engine** | The core contribution: a **spatial copula simulation** (latent Gaussian random fields with empirical-CDF margins; Kendall τ mapped to Gaussian correlation) jointly simulating seasonal rainfall and maize price over the whole SSA domain, fed through the yield-response model to produce per-pixel **profit distributions** under three scenarios: (i) joint climate–price risk, (ii) climate-only risk, and (iii) price-only risk. |
| **6 — Summaries** | Final maps, figures, and summary tables. |

### Key modelling choices

| Choice | Setting |
|---|---|
| Crop | Maize (the dominant SSA staple) |
| Nitrogen rates evaluated | 0–200 kg N/ha in 10 kg steps |
| Rainfall record | CHIRPS 1981–2024 |
| Price record | 2007–2024, fertilizer and maize |
| Copula spatial ranges | ~200 km (rainfall), ~250 km (price) |
| Smallholder yield calibration | 0.65 × trial-fitted yields |
| Farmgate price factor | 0.55 near markets → 0.15 beyond 6 h travel time |

## 4. Data sources

All inputs are public or published datasets:

| Dataset | Role |
|---|---|
| CHIRPS daily rainfall (via Google Earth Engine) | growing-season rainfall indicators, 1981–2024 |
| GADM boundaries | SSA country mask |
| iSDA Africa soil properties | soil covariates for the yield-response model |
| CAROB agronomic-trial compilation | multi-location fertilizer-response trials for model training |
| AfricaFertilizer price database (2010–2024) | nitrogen price surfaces |
| World Bank RTFP / WFP monthly market prices (2007–2024) | maize price prediction and volatility |
| NDVI time series | remote-sensing crop-loss probability |
| Household survey data | survey-based crop-loss probability |
| Travel-time-to-cities surface | farmgate price adjustment |

## 5. Outputs the final release will include

| Output | Description |
|---|---|
| **Risk-adjusted profitability maps** | Per-pixel expected profit, probability of loss, and downside-risk metrics for fertilizer use, under joint climate–price, climate-only, and price-only uncertainty. |
| **Optimal-rate maps** | Economically optimal (risk-adjusted) nitrogen application rates, contrasted with blanket recommendations. |
| **Yield-response surfaces** | Site-specific predicted maize response to N and P. |
| **Price surfaces** | Predicted local nitrogen prices, maize prices, and maize price volatility. |
| **Crop-loss probability maps** | Survey-based and NDVI-based severe-loss probabilities. |
| **Full pipeline** | The reproducible R/Python code, configuration, and documentation. |

Results are being finalised; headline maps and numbers will be added to this README with the working-paper release. The model outputs also power a farmer-facing digital advisory service that translates the per-pixel distributions into plain-language recommendations.

## 6. Repository structure

```
ssa-yield-uncertainty/
├── README.md          ← you are here
├── LICENSE            ← MIT
├── CITATION.cff
├── R/                 ← shared helpers (placeholder — released with the working paper)
├── config/            ← centralized pipeline configuration (placeholder)
├── data/              ← input & intermediate data (placeholder — fetch instructions to follow)
└── docs/              ← figures and documentation (to be added)
```

The numbered pipeline scripts (stages 0–6 above) will live at the repository root and run in numeric order.

## 7. Citation

If you reference this work before the working paper is out, please cite the repository (see [`CITATION.cff`](CITATION.cff)).

## 8. Contact

Bisrat Haile — bismignot@gmail.com

---

*This repository is a public placeholder for an analysis in progress. The framework description above is current; results and code will follow.*
