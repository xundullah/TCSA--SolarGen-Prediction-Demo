# Towards Reliable Solar Plant Generation Forecasting: Temporal Context Self-Attention Across Recurrent and Transformer Models

**Supplementary Material for Review** — code, curated artifacts, and interactive
dashboard supporting the manuscript submitted to *IEEE Transactions on Industrial
Informatics*.

This repository contains the complete research pipeline behind the **Temporal
Context Self-Attention (TCSA)** mechanism proposed in the manuscript, together
with the exported figures, metric tables, trained-model checkpoints, and the
web-based **Interactive Analytical Dashboard (IAD)** described in Section IV-D.
It is provided so that reviewers can trace every reported number and figure back
to the code that produced it.

---

## ▶ Interactive Analytical Dashboard (live)

### **https://xundullah.github.io/TCSA--SolarGen-Prediction-Demo/**

This is the **IAD of Fig. 6** in the manuscript, and the single most direct way to
inspect the results without running any code. The dashboard — *SPFS: Solar-Plant
Forecasting Simulator* — replays the day-ahead predictions of the baseline, MHSA,
and TCSA variants against the measured generation, hour by hour, over the 144-hour
test horizon, for each of the three solar plants and each architecture group.

The dashboard provides:

- **Simulation setup** — selection of the Solar Plant (SP) and the architecture
  family (LSTM, GRU, Transformer), with play, step, next-peak, and speed controls.
- **System log** — operational events, including sunrise infeed onset, sunset
  cessation, and the daily generation peak with the corresponding TCSA deviation.
- **Data log** — the current-hour measured output alongside the baseline, MHSA,
  and TCSA predictions.
- **Prediction-versus-actual chart** — hour-by-hour traces, with not-yet-played
  hours dimmed.
- **Live cumulative error** — MAE, RMSE, and R² of the proposed model updated over
  the hours played so far, so the error can be observed as it evolves through the
  day rather than as a single aggregate value.
- **Single-line diagram** — the multi-plant electrical architecture of Fig. 1, in
  which each component reports its parameters interactively.

No installation is required. The same page is `index.html` in this repository and
runs locally by opening that file directly in a browser — no build step, no server.

---

## 1. Scope and correspondence with the manuscript

The manuscript is organized around three domains; this repository follows the same
organization.

| Manuscript domain | Manuscript modules | Where it lives in this repository |
|---|---|---|
| **DE** — Data Engineering (Sec. III-A, Algorithm 1) | RHD, HDF, DCI | `Code/Site02_DataPreparation.ipynb`, `Library/dataProcessing.py`, `Library/dataAnalysis.py` |
| **FMA** — Forecasting Model Architecture (Sec. III-B, Algorithms 2–3) | RSN family, TEN family | `Code/Site02_ModelDevelopment.ipynb`, `Library/modelDevelopment.py` |
| **FEV** — Forecast Evaluation & Visualization (Sec. IV) | MPV, DHF, IAD | `Code/Site02_ResultAnalysis.ipynb`, `Library/modelEvaluation.py`, `index.html` |

### Naming map

Internal identifiers in the code predate the manuscript notation. The
correspondence is:

| In this repository | In the manuscript |
|---|---|
| `Site-02`, `Site#02` | The site in Gyeongju-si, Gyeongsangbuk-do, South Korea (Sec. II) |
| Plant `...G0001` | Solar Plant I (SP 1) |
| Plant `...G0003` | Solar Plant II (SP 2) |
| Plant `...G0004` | Solar Plant III (SP 3) |
| Station `...G4001` | The co-located weather station (Sec. II) |
| `PkW` | Plant output `P_pv^SP` in kW |
| `Base` / `MHSA` / `TCSA` | Variant `ζ ∈ {Base, MHSA, TCSA}` of (4) and (5) |
| `skill score` | Prediction Skill (PS) of (6) |

---

## 2. The eight benchmarked models

The TCSA mechanism is applied to two model families — Recurrent Sequence Networks
(RSN) and Transformer Encoder Networks (TEN) — and compared against the baseline
and conventional multi-head self-attention (MHSA) counterparts under a common
experimental setting. All models share the Day-Ahead Hourly Forecasting (DHF)
task: a 168-hour (one-week) input window of plant output plus six meteorological
variables, mapped to the next 24 hourly outputs.

| Family | Cell / encoder | Baseline | MHSA (typical) | TCSA (proposed) |
|---|---|---|---|---|
| RSN | LSTM | ✓ | ✓ | ✓ |
| RSN | GRU | ✓ | ✓ | ✓ |
| TEN | Transformer encoder | — | ✓ (family reference) | ✓ |

The Transformer group has no plain baseline, since its encoder is already
attention-based; the MHSA variant therefore serves as its family reference for the
Prediction Skill (PS) computation. Every model is trained and validated
independently for each of the three plants, giving 24 SP–model pairings.

---

## 3. Reproduction pipeline

Run the notebooks in `Code/` in this order.

### 3.1 `Site02_DataPreparation.ipynb` — the DE layer

Implements Algorithm 1. Loads the pre-processed hourly record from
`Export/Data/Site#02_Data.gzip`, removes observations outside the physical bounds,
partitions the record chronologically, reconstructs the remaining gaps, applies the
variable-wise Min–Max scaler fit on the training partition alone, and windows the
result per plant.

Gap reconstruction follows (1): a harmonic-climatology fit with shifted-residual
transfer for the meteorological variables, and diurnal bootstrapping — uniform
sampling from the same-month, same-hour training pool — for the PV power, so that
night-time zeros and the sunrise/sunset ramp are preserved by construction.
Implemented in `Library/dataProcessing.py`.

Also produces the diurnal-pattern figure of the site via
`dataAnalysis.plot_diurnal_patterns`.

### 3.2 `Site02_ModelDevelopment.ipynb` — the FMA layer

Trains all eight models for each of the three plants under the MSE objective, using
the 168 h → 24 h windowing scheme of `modelDevelopment.make_windows`. The final
20% of windowed samples form the validation set, separated from the training
samples by a 192-hour gap (`τ_w + τ_o`) so that no validation window overlaps a
training window. Early stopping uses a patience of 20 epochs with best-validation
checkpointing. Training histories are written to `Export/History/Site-02/` and the
best checkpoints to `Export/Model/Site-02/`.

### 3.3 `Site02_ResultAnalysis.ipynb` — the FEV layer

Evaluates every SP–model pairing on the held-out test partition, inverts the
normalization so that all reported errors refer to actual plant output in kW, and
generates the manuscript figures and metric tables through
`Library/modelEvaluation.py`.

---

## 4. Figure and table correspondence

All figures are exported as vector PDF and 600 dpi PNG under
`Export/Figure/Site-02/`, using a colorblind-safe (Okabe–Ito) IEEE style.

| Manuscript | Artifact | Generated by |
|---|---|---|
| Fig. 2 — diurnal characteristics of the site dataset | `diurnal_patterns.png` / `.pdf` | `dataAnalysis.plot_diurnal_patterns` |
| Fig. 3 — training and validation learning curves, 3 plants × 8 models | `LearningCurves-Grid-AllModels-AllPlants.png` / `.pdf` | `modelEvaluation.plot_learning_curves_grid` |
| Fig. 4 — test-set metric summary with TCSA improvement | `MetricSummary-PkW-AllPlants.png` / `.pdf` | `modelEvaluation.plot_metric_summary` |
| Fig. 5 — hourly day-ahead forecast traces versus measured generation | `PredictionComparison-PkW-AllPlants.png` / `.pdf` | `modelEvaluation.plot_prediction_comparison` |
| Fig. 6 — Interactive Analytical Dashboard | `index.html` (live demo above) | — |
| Table II — test-set performance of all models across the three plants | `Export/Data/Site-02/PredictionComparison-PkW-AllPlants-Metrics.csv` | `modelEvaluation.plot_prediction_comparison` |
| TCSA improvement percentages quoted in Sec. IV-B | `Export/Data/Site-02/MetricSummary-PkW-AllPlants-TCSA-Improvement.csv` | `modelEvaluation.plot_metric_summary` |

Two points a reviewer may wish to verify directly:

1. **All scalar metrics in Fig. 5 are computed over the full test set**, not over
   the five days displayed in the panels. The displayed window (June 9–13) is
   illustrative; the per-panel RMSE, MAE, and PS boxes come from the same CSV that
   backs Table II.
2. **Fig. 3 is the overfitting check.** Each panel annotates the best epoch, the
   early-stop epoch, the patience window, and the train–validation gap at the best
   epoch, supporting the claim in Sec. IV-B that all 24 pairings converge without
   overfitting.

---

## 5. Headline results

Reproduced from Table II of the manuscript for orientation; the authoritative
values are the exported CSVs.

| Plant | Best model | MAE (kW) | RMSE (kW) | rRMSE (%) | R² (%) | PS (%) |
|---|---|---|---|---|---|---|
| Solar Plant I | TCSA-LSTM | 4.74 | 8.43 | 42.58 | 88.63 | 15.95 |
| Solar Plant II | TCSA-LSTM | 4.96 | 8.74 | 42.85 | 87.91 | 20.15 |
| Solar Plant III | TCSA-LSTM | 5.32 | 9.22 | 42.07 | 88.09 | 15.50 |

Across all 24 SP–model pairings the TCSA variant is the best of its group without
exception: MAE is reduced by up to 28.6% and RMSE by up to 20.2%, while R²
increases from 81.03% to 87.91% in the best case. Inference cost is unaffected —
every model produces a 24-hour forecast in at most 1.7 ms per window.

---

## 6. Repository layout

```
Code/                             Research pipeline notebooks
  Site02_DataPreparation.ipynb      DE layer  — Algorithm 1: curation, gap
                                      reconstruction, scaling, windowing
  Site02_ModelDevelopment.ipynb     FMA layer — trains 8 models x 3 plants
  Site02_ResultAnalysis.ipynb       FEV layer — evaluation and manuscript figures

Library/                          Reusable modules shared by the notebooks
  dataProcessing.py                 Harmonic-climatology fill for the meteorological
                                      variables, diurnal-bootstrap fill for PV power
  dataAnalysis.py                   Diurnal-pattern figures, out-of-range removal
  modelDevelopment.py               168h -> 24h windowing, train/validation split,
                                      per-model evaluation, loss-curve plots
  modelEvaluation.py                Full evaluation-figure suite and metrics export

Export/                           Generated artifacts
  Data/Site#02_Data.gzip            Pre-processed hourly site dataset (pipeline input)
  Data/Site-02/                     Exported metrics and prediction CSVs
  Figure/Site-02/                   All figures (vector PDF + 600 dpi PNG)
  History/Site-02/                  Per-model training-history pickles
  Model/Site-02/                    Keras checkpoints (best epoch only)

Sim/Site-02/                      Prediction and metric CSVs consumed by the IAD
index.html                        IAD / SPFS - Solar-Plant Forecasting Simulator
```

---

## 7. Environment

The pipeline is built on a TensorFlow/Keras stack. Two interpreters are used
because the modelling notebooks are pinned to a GPU-capable TensorFlow release.

| Component | Python | Key packages |
|---|---|---|
| `Site02_DataPreparation.ipynb` | 3.11 | pandas, numpy, matplotlib, scikit-learn |
| `Site02_ModelDevelopment.ipynb`, `Site02_ResultAnalysis.ipynb` | 3.9.13 | `tensorflow-gpu<2.10`, `tensorflow-addons==0.19.0`, pandas, numpy, matplotlib, scikit-learn |

The dashboard is dependency-free and runs in any modern browser.

---

## 8. Data availability

The raw operational archives are retrieved from the plant operator's SQL database
through the connection descriptor of Algorithm 1 and are not redistributed here.
The pre-processed hourly record that feeds the published pipeline is included as
`Export/Data/Site#02_Data.gzip`, so the DE, FMA, and FEV layers are reproducible
end to end from this repository alone.
