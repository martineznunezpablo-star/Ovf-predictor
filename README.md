# OVF Predictor — v1.1

Clinical decision-support **research prototype** for thoracolumbar osteoporotic vertebral fractures (OVFs).
Estimates the probability of an unfavourable clinico-radiological outcome at 3 months, and contrasts two
scenarios: conservative management versus vertebral cementation (VA).

**Live application:** https://ovf-predictor.vercel.app/
**Source code:** {{GITHUB_URL}}

---

## ⚠️ Safety notice

> The counterfactual difference between scenarios (Δ) displayed by this calculator is a
> **hypothesis-generating estimate** derived from an observational logistic-regression model in which
> treatment (vertebral cementation) is included as a covariate within the model itself.
> **It is not a causal estimate of treatment effect.**
>
> The model **does not incorporate procedural risk** — cement leakage, adjacent-level fracture, embolic
> events, infection — **nor any contraindication** to cementation.
>
> **It must not be used as a treatment recommendation.** For research use only. It does not replace
> clinical judgment.

This notice is displayed as a blocking modal on every page load, **before any output can be viewed**, and
must be explicitly acknowledged before the calculator will produce a result. A condensed version remains
visible above the form, and a further caveat is attached to the Δ panel itself. The same text is
reproduced in the manuscript.

<details>
<summary>Versión en español</summary>

> La diferencia contrafactual entre escenarios (Δ) que muestra esta calculadora es una **estimación
> generadora de hipótesis**, obtenida de un modelo de regresión logística observacional en el que el
> tratamiento (cementación vertebral) se incluye como covariable dentro del propio modelo. **No es una
> estimación causal del efecto del tratamiento.**
>
> El modelo **no incorpora el riesgo del procedimiento** — fuga de cemento, fractura de nivel adyacente,
> eventos embólicos, infección — **ni las contraindicaciones** a la cementación.
>
> **No debe utilizarse como recomendación de tratamiento.** Solo para uso en investigación. No sustituye
> el juicio clínico.

</details>

---

## Model

| | |
|---|---|
| Algorithm | `LogisticRegression(C=0.1, L2)` wrapped in `CalibratedClassifierCV(cv=5, method='sigmoid')` |
| Calibration | Platt scaling, one calibrator per fold |
| Prediction | Mean of the 5 calibrated probabilities (reproduces the Python reference model) |
| Outcome | P(unfavourable clinico-radiological outcome at 3 months) |
| Derivation cohort | n = 482 (385 train / 97 hold-out) |
| External validation | n = 74, two independent centres |
| Frozen model artifact | v5.1 (the calculator itself is versioned v1.1) |

The model coefficients, the `StandardScaler` parameters and the Platt calibrators are embedded directly in
`index.html`. There is no server, no build step and no runtime dependency: the whole computation runs in the
user's browser.

### Features (9)

| # | Feature | Type | Coding |
|---|---------|------|--------|
| 1 | Age | continuous | years |
| 2 | VA (vertebral cementation) | binary | 0 = no / 1 = yes |
| 3 | Injury mechanism | binary | 0 = low-energy trauma / 1 = no trauma (spontaneous) |
| 4 | Health Status | ordinal | −2 / −1 / 0 |
| 5 | Deformity progression | binary | 0 = no / 1 = yes |
| 6 | BMI | continuous | kg/m² |
| 7 | mFI (modified frailty index) | ordinal | 0 / 1 / 2 |
| 8 | OF classification (fracture morphology) | ordinal | 1–5 (Schnake et al., 2018) |
| 9 | Hounsfield Units | continuous | HU, L1 vertebral body ROI |

### Performance

| Metric | Internal 5-fold CV (n=482) | External validation (n=74) |
|---|---|---|
| AUC-ROC | **0.667** | 0.744 [95 % CI 0.628–0.844] |
| Sensitivity | 63.5 % | 90.7 % |
| Specificity | 62.6 % | 54.8 % |
| PPV | 66.5 % | 73.6 % |
| NPV | 59.4 % | 81.0 % |
| Brier score | 0.227 | 0.211 |

The internal AUC is presented as the primary figure, consistent with the pre-specified, frozen-model
operating point described below. The external AUC is reported alongside it.

---

## Operating point

The Youden index is a property of the dataset it is computed on, not of the model. Three candidate
thresholds exist for this model: 0.556 (internal 5-fold CV), 0.493 (hold-out) and 0.432 (external cohort).

**Primary operating point — threshold 0.556.** Derived by Youden index on stratified 5-fold
cross-validation of the derivation cohort (n=482). It is pre-specified on the derivation data and is
therefore independent of the external cohort used to estimate generalisation performance. This is the
default and the only threshold used unless the user explicitly changes it.

**Alternative screening mode — threshold 0.432.** Derived by Youden index on the external validation
cohort. Because it was selected on the same set used to estimate performance, its apparent metrics are
optimistic. It is available in the application behind a collapsed *Advanced* panel, switched off by
default, and explicitly labelled `post-hoc` · `external validation` · `not the primary operating point`,
with a visible warning. It is provided as a sensitivity analysis only.

Switching modes changes only the binary classification and the reported operating characteristics. The
predicted probabilities and the counterfactual Δ are unaffected.

---

## Input ranges and out-of-range handling

Two independent constraints are applied, because they answer different questions: whether a value is
*physiologically plausible*, and whether the model has *ever seen* values like it.

| Variable | Input range accepted | Derivation cohort range | Behaviour outside |
|---|---|---|---|
| Age | 53–93 years | 53–93 | slider-bounded |
| BMI | 17.6–38.5 kg/m² | 17.6–38.5 | slider-bounded |
| Hounsfield Units | slider 30–300; numeric field unbounded | 16.6–147.6 | see below |

For Hounsfield Units:

- **Plausibility warning** — triggered for any value **below 30 or above 300 HU**. Shown next to the input
  and repeated as a flag on the results page. The slider is bounded to 30–300; the adjacent numeric field
  accepts any value so that clinically implausible entries are surfaced rather than silently clamped.
- **Extrapolation warning (`out-of-training-range`)** — triggered for any value outside **16.6–147.6 HU**,
  the range actually observed in the derivation cohort. Shown next to the input and marked on the results
  page. Predictions outside this interval are extrapolations whose reliability has not been assessed.

The two warnings are independent and can fire together. A value of 20 HU, for instance, is below the
plausibility floor but within the training range; a value of 200 HU is plausible but outside it.

---

## Counterfactual Δ

VA (vertebral cementation) is the only therapeutically modifiable feature among the nine. The calculator
evaluates the model twice for the same patient — once with VA = 0 and once with VA = 1, all other features
held constant — and reports the difference.

This Δ is a within-model contrast, not a causal effect estimate. It inherits every limitation of the
observational design: confounding by indication, absence of procedural risk, and absence of
contraindications. It is presented descriptively, with both scenarios given equal visual weight, and is
accompanied by the safety notice reproduced above.

## Explainability

Per-patient variable contributions are computed as the average model coefficient multiplied by the
standardised feature value (`coef_avg × z-score`), which is equivalent to averaging SHAP values across the
five base classifiers for a linear model. Positive values push towards an unfavourable outcome; negative
values are protective. Contributions are displayed for the conservative scenario (VA = 0), so the bars
represent the patient's intrinsic risk profile; the effect of VA is shown separately in the comparison
panel.

---

## Privacy

No patient data is transmitted or stored. All computation is client-side; the application makes no network
requests other than loading its own static assets and a web font.

## Repository contents

```
index.html      Complete application: markup, styles, i18n (ES/EN), model and inference
package.json    Package metadata
vercel.json     Static deployment configuration (no build step)
CITATION.cff    Machine-readable citation metadata
LICENSE         MIT licence
README.md       This file
```

## Deployment

Static site, no build. Any static host will serve it; the reference deployment is on Vercel with
continuous deployment from the `main` branch.

```bash
# local preview
python3 -m http.server 8000   # then open http://localhost:8000
```

---

## Version history

**v1.1** — 2026-09
- Blocking safety notice shown before any output can be viewed, acknowledged explicitly, reproduced in the manuscript.
- Primary operating point moved to the pre-specified internal threshold **0.556**; the external 0.432 threshold retained as a labelled, off-by-default screening mode.
- Binary classification against the operating point replaces the previous 0.60 / 0.40 traffic-light bands, which corresponded to neither validation set.
- Operating characteristics (Se, Sp, PPV, NPV) displayed for the active threshold.
- Hounsfield Unit input restricted to 30–300 with an unbounded numeric field; plausibility and out-of-training-range warnings added.
- Age and BMI input ranges aligned with the derivation cohort.
- Internal AUC (0.667) presented as the primary performance figure, with the external AUC (0.744, n=74) alongside; corrected the previous presentation, which paired the external AUC with the derivation cohort size.
- Source code link surfaced in the application.

**v1.0**
- Initial public release. Classification at threshold 0.432; passive disclaimer banner.

---

## Citation

> Martínez-Núñez P, Gutiérrez-González R. OVF Predictor: explainable machine-learning model for
> thoracolumbar osteoporotic vertebral fractures. Master's Thesis, Universidad Europea, 2025/2026.

## Authors

Pablo Martínez-Núñez · Raquel Gutiérrez-González
Universidad Europea — TFM 2025/2026

## License

Released under the **MIT License** — see [`LICENSE`](LICENSE). You may use, copy, modify and
redistribute this software, including for commercial purposes, provided the copyright notice and
permission notice are retained.

The licence grants permission to reuse the code. It does not constitute a clinical endorsement: this
is a research prototype, not a medical device, and the safety notice above applies to any derivative.
