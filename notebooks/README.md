# Boom Challenge — Physics-Informed Ejecta Prediction

## Overview

This repository contains my solution for the **Boom: Trajectory Unknown Challenge**.

The challenge has two objectives:

1. **Forward Prediction**: predict ejecta outcomes from impact parameters.
2. **Inverse Design**: propose 20 impact scenarios satisfying specific ejecta constraints.

The final solution uses a **physics-informed machine learning pipeline** with out-of-distribution validation and constrained inverse design.

The goal was not only to optimize validation metrics, but also to respect the expected physical behavior of ejecta travel distances under varying gravity, energy transfer, material resistance, and atmospheric drag.

---

## Final Submission Files

The two official submission files are:

```text
outputs/submissions/prediction_submission.csv
outputs/submissions/design_submission.csv
```

### Forward Prediction Output

`prediction_submission.csv` contains predictions for 492 test scenarios:

```text
scenario_id, P80, fines_frac, oversize_frac, R95, R50_fines, R50_oversize
```

### Inverse Design Output

`design_submission.csv` contains 20 proposed impact scenarios:

```text
submission_id, energy, angle_rad, coupling, strength, porosity, gravity, atmosphere, shape_factor
```

---

## Final Decision

The final forward submission uses:

```text
Hybrid MLP-distance model with weight 0.65
```

This choice was made after Leave-One-Gravity-Out validation showed that the hybrid model was the strongest candidate for distance targets under gravity-based out-of-distribution validation.

A final Ridge meta-blend was also tested after the hybrid model. It produced only a marginal mean improvement and degraded P95 tail robustness, so the hybrid model was retained as the final forward submission.

The final inverse-design file was kept because all 20 proposed designs remained feasible under the final hybrid forward model and under additional robustness checks.

---

## Method Summary

The solution combines:

- physics-informed feature engineering;
- target-specific regression models;
- hybrid modeling for distance targets;
- out-of-distribution validation using Leave-One-Gravity-Out CV;
- robust inverse design under output constraints.

---

## Physics-Informed Approach

The raw impact parameters were transformed into physically meaningful features.

Main feature groups include:

- **energy transfer features**: effective energy and log effective energy;
- **angle decomposition features**: sine, cosine, horizontal energy, and vertical energy;
- **material features**: strength, porosity, material resistance, and fragmentation proxies;
- **gravity-scaled distance features**: energy per gravity and range proxies;
- **atmosphere and drag features**: drag proxy and atmosphere-shape-energy interactions;
- **regime indicators**: porosity, strength, angle, and atmosphere regimes.

These features were designed to help the model learn relationships that are physically meaningful, especially for distance-related outputs affected by gravity.

---

## Forward Prediction Strategy

The final model uses a target-specific approach.

### Fragmentation Targets

For the fragmentation targets:

```text
P80
fines_frac
oversize_frac
```

the final solution uses:

```text
ExtraTreesRegressor + advanced fragmentation features
```

These targets were more stable under tree-based models and benefited from material-resistance and fragmentation features.

### Distance Targets

For the distance targets:

```text
R95
R50_fines
R50_oversize
```

several strategies were tested:

- ExtraTrees baseline;
- MLP distance model;
- controlled distance blends;
- physics-informed residual model;
- Ridge meta-blend;
- OOD validation with Leave-One-Gravity-Out CV.

The final forward submission uses the hybrid MLP-distance model with weight 0.65.

The main OOD failure mode was conservative under-extrapolation of travel distances when gravity regimes were not seen during training. The hybrid distance model helped reduce this under-extrapolation.

---

## Out-of-Distribution Validation

The training set contains only three gravity levels:

```text
1.62, 3.71, 9.81
```

The test set contains unseen gravity levels:

```text
1.02, 1.38, 4.91, 7.03, 10.47
```

Because of this, random KFold validation was not sufficient. Random KFold mainly tests interpolation between examples that share the same gravity regimes. A **Leave-One-Gravity-Out** validation was used to better simulate the OOD setting.

The global Leave-One-Gravity-Out ranking was:

```text
1. hybrid_w_0.65
2. physics_residual
3. alpha25
4. base ExtraTrees
```

This supported the final choice of the hybrid forward submission.

---

## Physics Residual and Meta-Blend Experiments

A physics-informed residual model was tested with the following idea:

```text
prediction = physics_baseline + ML_residual
```

This approach was useful as a scientific benchmark, but it was not selected because it produced more extreme test predictions and was less stable than the hybrid model.

A final Ridge meta-blend was also tested on top of the hybrid model:

```text
final_prediction = (1 - alpha) * hybrid_w_0.65 + alpha * meta_prediction
```

The best meta-blend improvement was marginal and degraded P95 tail robustness. Therefore, the final decision was to keep the hybrid_w_0.65 submission.

---

## Inverse Design Strategy

The inverse design task required 20 impact scenarios satisfying:

```text
96 <= P80 <= 101
R95 <= 175
```

The final inverse-design pipeline:

1. generated candidate scenarios within the official input bounds;
2. predicted outcomes using trained forward models;
3. filtered candidates satisfying the constraints;
4. selected 20 feasible and robust scenarios;
5. rechecked the selected scenarios under multiple forward variants.

The final 20 scenarios remained feasible under:

```text
base ExtraTrees
hybrid_w_0.65
alpha25
alpha40
```

Therefore, the final `design_submission.csv` was kept after selecting the hybrid forward model.

---

## Repository Structure

```text
.
├── README.md
├── requirements.txt
├── notebooks/
│   ├── 01_eda_forward_prediction.ipynb
│   ├── 02_feature_engineering_analysis.ipynb
│   ├── 03_modeling_extratrees_pipeline.ipynb
│   ├── 04_forward_v2_advanced_features_test.ipynb
│   ├── 05_forward_physics_ood_features_test.ipynb
│   ├── 06_forward_model_diagnostics_and_learning_curves_clean.ipynb
│   ├── 07_forward_hybrid_mlp_distance_test.ipynb
│   ├── 08_ood_controlled_blend_test.ipynb
│   ├── 09_comparison_v1_vs_v2_methodology.ipynb
│   ├── 10_leave_one_gravity_out_uncertainty.ipynb
│   ├── 11_final_logo_meta_blend_test.ipynb
│   ├── 12_final_forward_decision_and_submission.ipynb
│   └── 13_final_inverse_design_validation.ipynb
├── outputs/
│   └── submissions/
│       ├── prediction_submission.csv
│       └── design_submission.csv
└── reports/
```

---

## Main Notebooks

| Notebook | Purpose |
|---|---|
| `01_eda_forward_prediction.ipynb` | Exploratory analysis of inputs, targets, regimes, skewness, outliers, and correlations |
| `02_feature_engineering_analysis.ipynb` | Physics-informed feature engineering and feature-target analysis |
| `03_modeling_extratrees_pipeline.ipynb` | First clean ExtraTrees baseline using sklearn pipelines |
| `04_forward_v2_advanced_features_test.ipynb` | Tests additional advanced features |
| `05_forward_physics_ood_features_test.ipynb` | Tests physics/OOD-inspired feature extensions |
| `06_forward_model_diagnostics_and_learning_curves_clean.ipynb` | Checks overfitting, learning curves, and target-level diagnostics |
| `07_forward_hybrid_mlp_distance_test.ipynb` | Tests MLP hybrid modeling for distance targets |
| `08_ood_controlled_blend_test.ipynb` | Tests controlled blends between baseline and hybrid candidates |
| `09_comparison_v1_vs_v2_methodology.ipynb` | Compares baseline, new features, log-transform, and residual strategies |
| `10_leave_one_gravity_out_uncertainty.ipynb` | OOD validation using Leave-One-Gravity-Out CV and bootstrap uncertainty |
| `11_final_logo_meta_blend_test.ipynb` | Final optional meta-blend test; hybrid_w_0.65 was retained |
| `12_final_forward_decision_and_submission.ipynb` | Selects the final forward file and validates submission format |
| `13_final_inverse_design_validation.ipynb` | Validates the final inverse-design file under the selected forward strategy |

---

## How to Reproduce

Install dependencies:

```bash
pip install -r requirements.txt
```

Place the challenge data under:

```text
data/raw/forward_prediction/
data/raw/inverse_design/
```

Run the notebooks in the documented order.

The final files should be located at:

```text
outputs/submissions/prediction_submission.csv
outputs/submissions/design_submission.csv
```

Final validation can be run with:

```bash
python - <<'PY'
import pandas as pd
import numpy as np

pred = pd.read_csv("outputs/submissions/prediction_submission.csv")
design = pd.read_csv("outputs/submissions/design_submission.csv")

assert pred.shape == (492, 7)
assert design.shape == (20, 9)
assert pred["scenario_id"].tolist() == list(range(492))
assert design["submission_id"].tolist() == list(range(20))
assert np.isfinite(pred.drop(columns=["scenario_id"]).values).all()
assert np.isfinite(design.drop(columns=["submission_id"]).values).all()

print("Final submission files OK.")
PY
```

---

## Validation Checks

The final solution was selected after comparing several strategies using:

- MAE;
- RMSE;
- R²;
- P95 absolute error;
- train vs validation gap;
- learning curves;
- OOD distance diagnostics;
- Leave-One-Gravity-Out CV;
- bootstrap confidence intervals;
- inverse-design feasibility checks.

The final forward file was selected only after confirming that the hybrid distance model was stronger under gravity-based OOD validation.

---

## Limitations

This solution is physics-informed, but it is not a full physical simulator.

The main limitation is that the test set contains unseen gravity regimes, so true generalization cannot be fully verified without labels. Leave-One-Gravity-Out validation was used as the strongest available proxy for OOD performance.

---

## Future Work

Possible improvements include:

- probabilistic models for uncertainty estimation;
- Bayesian optimization for inverse design;
- symbolic regression for interpretable physical formulas;
- monotonicity-constrained models for distance targets;
- uncertainty-aware inverse-design selection.

---

## Key Takeaway

The final solution prioritizes:

```text
physical plausibility
out-of-distribution robustness
target-specific modeling
hybrid distance prediction
robust inverse design
```

over relying on a single black-box model.
