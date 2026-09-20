# SHAP Presentation Report — Soft vs Good Reactor Days

**Notebook 05** · chemistry/field only · temporal train ≤2016, test ≥2017  
**Question:** Which chemistry features are most predictive of softer dioxane-removal days?

---

## 1. The question in one slide

Continuous removal % sits near a high ceiling, so regression was weak. We instead label days:

| | Definition (default) |
|--|--|
| **Soft day** | Blend → effluent removal (3320→3340) **< 95%** |
| **Good day** | Removal **≥ 95%** |

**Counts:** 226 soft · 879 good · 1,105 total days.

**Model:** Logistic classifier on **15 chemistry/field features**.  
**Excluded from the model:** `dioxane_3320`, blend ratio, sed–blend gap (those reuse blend dioxane and are entangled with the label).

**Test performance at 95%:** ROC-AUC **0.758** · AUPRC **0.401** (chance ≈ 0.178).

---

## 2. Simple picture (use this first)

| | Good days | Soft days |
|--|--|--|
| Reactor performance | Strong dioxane removal | Weaker dioxane removal |
| Phosphorus (3320) | Higher (median 2200) | Lower (median 1100) |
| Nitrate/nitrite (3310) | Higher (median 240) | Lower (median 50) |
| Ammonia (3310) | Lower (median 169) | Higher (median 220) |
| Incoming dioxane (3310) | Lower | Higher |
| Temperature (3320) | A bit warmer (20.5°C) | A bit cooler (18.9°C) |

These are **associations**, not proof of cause.

---

## 3. SHAP ranking at 95% (test days)

Mean |SHAP| = how much each feature moves the soft-day score, on average, on **future** days.

| Rank | Feature | Mean \|SHAP\| | Role |
|--:|--|--:|--|
| 1 | Phosphorus (3320) | 0.234 | Strongest, most stable driver |
| 2 | Nitrate/nitrite (3310) | 0.138 | Main nitrogen signal |
| 3 | Conductivity (3310) | 0.127 | Bulk ionic strength |
| 4 | Incoming dioxane (3310) | 0.126 | Influent load |
| 5 | TSS (3310) | 0.115 | Solids / turbidity-related |
| 6 | Temperature (3320) | 0.114 | Cooler on soft days |
| 7 | THF (3320) | 0.101 | Co-contaminant |
| 8 | Ammonia (3310) | 0.071 | Related N story; **not** top SHAP |

**Talking point:** Chris used ammonia as an *example*. Ammonia is higher on soft days, but SHAP says **nitrate/nitrite and phosphorus** are the predictive N/P leads.

---

## 4. Threshold sweep — 90 / 92 / 95 / 97 / 98%

Same features, same time split. Only the soft/good line changes.

| Soft if removal < | Soft rate | Test ROC-AUC | Test AUPRC | Chance AUPRC | #1 SHAP feature |
|--:|--:|--:|--:|--:|--|
| 90% | 3.9% | 0.728 | 0.138 | 0.044 | Phosphorus (3320) |
| 92% | 6.4% | 0.769 | 0.200 | 0.069 | Phosphorus (3320) |
| **95%** | **20.5%** | **0.758** | **0.401** | **0.178** | **Phosphorus (3320)** |
| 97% | 43.0% | 0.612 | 0.534 | 0.420 | Phosphorus (3320) |
| 98% | 58.3% | 0.709 | 0.806 | 0.622 | Phosphorus (3320) |

**How to read this**

- **90–95%:** model still has real ranking skill (AUC ~0.73–0.77). Very strict cutoffs make soft days rare, so AUPRC is low.
- **95%:** best balance of a meaningful “soft” class (~1 in 5 days) and strong AUPRC vs chance.
- **97–98%:** AUPRC rises because *most* days become “soft.” That is a class-balance effect, not automatically a better scientific definition. AUC is weakest at 97%.

**95% is a reasonable default, and we can show we checked it.**

---

## 5. Feature stability (the SHAP robustness result)

How often a feature is in the **top-5 SHAP** list across the five cutoffs:

| Feature | Times in top-5 | Verdict |
|--|--:|--|
| Phosphorus (3320) | **5 / 5** | Stable key driver |
| Nitrate/nitrite (3310) | 4 / 5 | Recurring |
| Incoming dioxane (3310) | 4 / 5 | Recurring |
| Conductivity (3310) | 4 / 5 | Recurring |
| 1,2-DCA (3310), THF | 2 / 5 | Threshold-specific |
| TSS, temperature | 1 / 5 | Strong at 95%, not everywhere |

**Headline:** phosphorus is not an artifact of picking 95%.

---

## 6. Microbial hypothesis (Chris’s ask)

SHAP gives **testable chemistry → microbe** leads when more 16S arrives (Chris mentioned ~35–40 days later, biased newer):

1. **Phosphorus-related** communities / limitation at the blend (3320).
2. **Nitrogen cycling** — especially nitrate/nitrite (not only ammonia).
3. Incoming load / conductivity as process context, not a taxa story by themselves.

Do **not** claim we have proven those microbes. Claim we now know **what to look for**.

---

## 7. Limits (one slide)

- Associations only — not causal.
- Temporal split is intentional: the plant’s operations changed over years.
- Chemistry-only. 16S is still too sparse for a full microbial model.
- Blend topology features were dropped from the **classifier** so SHAP is not recycling the label.

---

## 8. Suggested closer

> Soft-day classification works on future data. SHAP identifies **stable chemistry drivers — especially phosphorus and nitrate/nitrite** — across nearby thresholds. That is a coherent Phase 1 result and a concrete list of microbial hypotheses for when more 16S data arrive.

**Figures to show:**  
`nb05_shap_bar_soft95.png` · `nb05_shap_bars_by_threshold.png` · `nb05_shap_feature_stability.png`  
under `data/processed/analysis_results/`
