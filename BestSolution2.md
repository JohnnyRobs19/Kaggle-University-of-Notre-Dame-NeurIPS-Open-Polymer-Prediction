# BestSolution2: Hybrid RDKit/Mordred + XGB/Tree Models + GNN Ensemble (NeurIPS – Open Polymer Prediction 2025)

* A **two-stage pipeline** that (1) cleans/extends training data, builds **descriptor-heavy tabular features** from SMILES, and predicts each target with **label‑tuned gradient boosting forests**; then (2) **ensembles** those tabular predictions with a **graph neural network (GNN)** inference head. The final submission is the **mean** of all produced prediction frames.
* **Result:** **Final Leaderboard Result: 125 / 2,240 teams (Top 6%, Bronze Medal)**

---

## 1) Approach Overview

* **Config‑driven pipeline** with toggles for feature sets, selection, PCA, CV, and external data usage.
* **SMILES cleaning + canonicalization** followed by integration of **external property datasets** per target.
* **Feature factory** combining Mordred/RDKit descriptors, optional fingerprints, and simple graph topology features.
* **Models**: Label‑specific **XGBoost** baselines with alternatives (LightGBM, CatBoost, ExtraTrees, RF, HGBM, simple MLP). Optional **stacking** with ElasticNet meta‑learner.
* **GNN hybrid head** for inference‑time ensembling.
* **Outputs**: Per‑target predictions merged into `submission.csv`.

---

## 2) Data & Preprocessing

1. **Data sources**

   * Competition: `train.csv`, `test.csv`.
   * External enrichments (merged by SMILES):

     * **Tc**: `Tc_SMILES.csv` (renamed column `TC_mean → Tc`).
     * **Tg**: `TgSS_enriched_cleaned.csv`, `JCIM_sup_bigsmiles.csv` (`Tg (C) → Tg`), `data_tg3.xlsx` (K→°C conversion).
     * **Density**: `data_dnst1.xlsx` with cleaning, unit alignment, and outlier string filtering.
     * **FFV**: `train_supplement/dataset4.csv`.
2. **SMILES hygiene**

   * Drop polymer placeholder tokens (e.g., `[R]`, `R1`…), reject unparsable strings, and **canonicalize** valid molecules.
3. **Target tables**

   * Build one clean sub‑table per label: **Tg, FFV, Tc, Density, Rg** (drop NaNs per label).

---

## 3) Feature Engineering

* **Descriptors**

  * **Mordred** descriptor matrix (numeric subset, inf/NaN → NaN then handled downstream).
  * Selected **RDKit** descriptors: aromaticity, fraction sp3, LabuteASA, MolMR, H‑donors/acceptors, halogen counts, BalabanJ, Kappa2, rotatable bonds.
  * **Graph features** via molecular adjacency: diameter, average shortest path, cycle count.
* **Fingerprints & embeddings (optional)**

  * Morgan, MACCS, AtomPair, Torsion; optional **ChemBERTa** pooled embeddings.
* **Selection & cleanup**

  * **Standard scaling**; optional **variance filter** and **correlation pruning** (ρ > 0.96).
  * **Importance‑based trimming** (permutation importance / SHAP) to drop non‑informative columns.
  * Optional **LassoCV** selector for sparse keeps.
* **PCA (optional)** to reach a specified retained variance.

---

## 4) Modeling (per label)

Primary baseline uses **XGBoost** with **label‑specific** hyperparameters, early stopping on **MAE**. Defaults:

* **Tg**: moderate depth, stronger L2, subsample.
* **FFV**: higher depth/leaves to reflect more data.
* **Tc / Rg**: shallower depth, stronger regularization and smaller subsample for low‑data regimes.
* **Density**: moderate depth, larger subsample.

Alternative single‑models: **LightGBM, CatBoost, ExtraTrees, RandomForest, HistGradientBoosting, TabNet, MLP**.

**Stacking option**: XGB + LGBM + CatBoost + RF base learners with **ElasticNet** meta‑regressor using out‑of‑fold features.

---

## 5) Training Regime & Diagnostics

* **K‑Fold CV** with early stopping; MAE tracked per fold and aggregated.
* **Holdout** metrics optionally recorded.
* **Outlier summaries** available (Z‑score, IQR, Isolation Forest, LOF) to understand heavy‑tail targets.
* **GMM augmentation** (optional) to synthesize samples from feature–target space when helpful.

Artifacts typically persisted to a `NeurIPS/` and `models/` directory (per‑label kept columns, scalers, model binaries, and CV summaries) to enable consistent inference runs.

---

## 6) Inference & Ensembling

1. **Tabular path**

   * Load saved feature metadata (kept columns, medians, scaler), align test descriptors, scale, and predict with per‑label models. Average **fold** predictions.
2. **GNN path**

   * Build molecular graphs from SMILES, apply a light GNN with robust scaling for atom/bond features, produce per‑label predictions. If scalers/models are missing, use a **filtered‑median fallback** for safety.
3. **Final blend**

   * Concatenate produced prediction frames and compute the **row‑wise mean** to form the final `submission.csv`.

---
