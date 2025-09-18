# BestSolution1.md

* **Goal:** Predict five polymer properties (**Tg, FFV, Tc, Density, Rg**) from a molecule string called **SMILES**.
* **Result:** **Final Leaderboard Result: 125 / 2,240 teams (Top 6%, Bronze Medal)**
* **Idea in one line:** Clean the SMILES → turn each molecule into numbers (descriptors) → train one model per target (XGBoost) → submit.

> **What is SMILES?** A short text that describes a molecule’s structure. Example: `CCO` is ethanol. Computers can read SMILES to compute useful features.

---

## 1) The Task

We are given a list of polymers written as **SMILES**. For each polymer, we want to estimate:

* **Tg** (glass transition temperature)
* **FFV** (fractional free volume)
* **Tc** (thermal conductivity)
* **Density**
* **Rg** (radius of gyration)

The competition checks our answers using **Mean Absolute Error (MAE)** and then combines the five tasks into a **weighted MAE** score.

---

## 2) The Plan

1. **Clean the data**

   * Remove broken/invalid SMILES and standardise (canonicalise) the valid ones so the same molecule always looks the same.

2. **Add more examples**

   * Bring in a few outside datasets for some targets (e.g., Tg, Density). Match them by the same canonical SMILES.
   * If the same SMILES appears many times, average its value.

3. **Turn molecules into numbers**

   * Use **Mordred** to compute a big table of numeric **descriptors** (size, shape, polarity, etc.).
   * Scale the numbers so features are on comparable ranges.

4. **Train models**

   * Train one **XGBoost** regressor **per target** with objective **MAE** and **early stopping**.
   * Use slightly different settings depending on how much data a target has (e.g., smaller trees for low‑data targets like **Rg**).

5. **Predict & submit**

   * Run the trained models on the test SMILES and create the submission CSV.

---

## 3) Why This Works

* **Garbage in, garbage out:** Cleaning SMILES removes bad rows that would confuse the model.
* **More data helps:** Adding trusted external rows (matched by SMILES) gives the model more examples to learn patterns.
* **Descriptors capture chemistry:** Mordred converts a molecule into many meaningful numbers (e.g., size, flexibility), which correlate with Tg, FFV, Rg, etc.
* **Right metric:** Training with **MAE** matches how the leaderboard is scored.
* **Per‑target tuning:** Different targets have different data sizes; adjusting model depth and regularisation reduces overfitting.

---

## 4) What’s Inside the Code

* **SMILES cleaning:** Drop patterns like `[R]`, invalid tokens, and anything RDKit cannot parse. Canonicalise valid SMILES.
* **External data loader:** Safely reads extra CSV/XLSX files, fixes column names/units (e.g., **Tg \[K] → °C**), groups by SMILES, and merges.
* **Featurisation:** Computes **Mordred** numeric descriptors for every SMILES and standardises features.
* **Models:** Per‑target **XGBoost** with `reg:absoluteerror` (MAE) and early stopping on a small validation split.

---

## 5) Limitations (What to Improve Next)

* External sources use different units or scales; more unit checks could help.
* Low‑data targets (e.g., **Rg**) are harder; try fingerprints or simple SMILES augmentation if you have time.
* Ensembling and AutoGluon can squeeze extra points if you can afford longer runs.

---
