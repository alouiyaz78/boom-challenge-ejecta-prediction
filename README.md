# Boom Challenge — Physics-Informed Ejecta Prediction

## Overview

This repository contains my solution for the **Boom: Trajectory Unknown Challenge**.

The challenge has two objectives:

1. **Forward Prediction**: predict ejecta outcomes from impact parameters.
2. **Inverse Design**: propose 20 impact scenarios satisfying specific ejecta constraints.

The final solution uses a **physics-informed machine learning pipeline** with out-of-distribution checks and constrained inverse design.

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

## Method Summary

The solution combines:

- physics-informed feature engineering;
- target-specific regression models;
- out-of-distribution diagnostics;
- controlled extrapolation for distance targets;
- robust inverse design under constraints.

### Physics-Informed Features

The raw impact parameters were transformed into physically meaningful features, including:

- effective transferred energy;
- angle decomposition using sine and cosine;
- material resistance and fragmentation proxies;
- gravity-scaled distance features;
- atmosphere and drag proxies;
- regime indicators based on porosity, strength, angle, and atmosphere.

These features were designed to improve generalization on out-of-distribution test scenarios.

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

I used:

```text
ExtraTreesRegressor + advanced fragmentation features
```

### Distance Targets

For the distance targets:

```text
R95
R50_fines
R50_oversize
```

I tested ExtraTrees, MLP, and blended predictions.

The final submission uses a conservative controlled blend:

```text
final_distance = 75% ExtraTrees distance + 25% Hybrid distance
```

This was selected to reduce the risk of over-extrapolation while still correcting the conservative behavior of tree-based models on low-gravity OOD cases.

---

## Inverse Design Strategy

The inverse design task required 20 impact scenarios satisfying:

```text
96 <= P80 <= 101
R95 <= 175
```

The final inverse-design pipeline:

1. generated candidate scenarios within the official input bounds;
2. predicted outcomes using the trained forward model;
3. filtered candidates satisfying the constraints;
4. penalized candidates too close to input bounds;
5. selected 20 diverse and robust scenarios.

The final 20 scenarios were rechecked under multiple forward variants and remained feasible.

---

## Repository Structure

```text
.
├── README.md
├── requirements.txt
├── notebooks/
│   ├── 09_final_forward_prediction_pipeline.ipynb
│   ├── 10_final_inverse_design_pipeline.ipynb
│   ├── 11_forward_model_diagnostics_and_learning_curves.ipynb
│   ├── 12_forward_hybrid_mlp_distance_test.ipynb
│   ├── 13a_ood_controlled_blend_test.ipynb
│   └── 13b_recheck_inverse_design_with_distance_blends.ipynb
├── src/
├── reports/
└── outputs/
    └── submissions/
        ├── prediction_submission.csv
        └── design_submission.csv
```

---

## Main Notebooks

| Notebook | Purpose |
|---|---|
| `09_final_forward_prediction_pipeline.ipynb` | Builds the final forward prediction pipeline |
| `10_final_inverse_design_pipeline.ipynb` | Generates the inverse design scenarios |
| `11_forward_model_diagnostics_and_learning_curves.ipynb` | Checks overfitting, learning curves, and model robustness |
| `12_forward_hybrid_mlp_distance_test.ipynb` | Tests MLP blending for distance targets |
| `13a_ood_controlled_blend_test.ipynb` | Tests OOD-controlled blending |
| `13b_recheck_inverse_design_with_distance_blends.ipynb` | Verifies inverse-design feasibility under multiple models |

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

Then run the main notebooks in this order:

```text
09_final_forward_prediction_pipeline.ipynb
10_final_inverse_design_pipeline.ipynb
13a_ood_controlled_blend_test.ipynb
13b_recheck_inverse_design_with_distance_blends.ipynb
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
- inverse-design feasibility checks.

The final controlled blend was chosen because it provided a safer balance between stability and extrapolation on the OOD test set.

---

## Limitations and Future Work

This solution is physics-informed but not a full physical simulator.

Future improvements could include:

- probabilistic models such as NGBoost;
- symbolic regression for interpretable physical formulas;
- Bayesian optimization or CMA-ES for inverse design;
- physics-informed neural networks;
- uncertainty-aware inverse design.

---

## Key Takeaway

The final solution prioritizes:

```text
physical plausibility
out-of-distribution robustness
target-specific modeling
controlled extrapolation
robust inverse design
```

over relying on a single black-box model.
