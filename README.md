# NeurIPS – Open Polymer Prediction 2025

## Competition Summary
The **NeurIPS – Open Polymer Prediction 2025** challenged participants to build machine learning models that predict **five key polymer properties** directly from chemical structure (SMILES strings). The goal was to accelerate **sustainable materials discovery** by enabling accurate virtual screening, reducing reliance on computationally expensive molecular dynamics simulations.

---

## Objective
Your mission was to predict a polymer's real-world performance from its SMILES representation by forecasting the following **five properties:**

- **Glass Transition Temperature (Tg)**
- **Fractional Free Volume (FFV)**
- **Thermal Conductivity (Tc)**
- **Density**
- **Radius of Gyration (Rg)**

These properties were derived from **multiple molecular dynamics simulations** and averaged to create robust ground truth labels.

---

## Submission Format
The final submission was a CSV with predictions for each property:

```csv
id,Tg,FFV,Tc,Density,Rg
2112371,0.0,0.0,0.0,0.0,0.0
2021324,0.0,0.0,0.0,0.0,0.0
343242,0.0,0.0,0.0,0.0,0.0
```

---

## Impact
This competition advances **polymer informatics** by:
- Providing a **large-scale open-source dataset** (10× bigger than existing resources)
- Encouraging **multi-task ML models** for material property prediction
- Enabling faster discovery of **eco-friendly, biocompatible materials** for applications in medicine, electronics, and sustainability.

---

## Citation
> Gang Liu, Jiaxin Xu, Eric Inae, Yihan Zhu, Ying Li, Tengfei Luo, Meng Jiang, Yao Yan, Walter Reade, Sohier Dane, Addison Howard, and María Cruz. *NeurIPS - Open Polymer Prediction 2025*. [Kaggle Competition](https://kaggle.com/competitions/neurips-open-polymer-prediction-2025), 2025.
