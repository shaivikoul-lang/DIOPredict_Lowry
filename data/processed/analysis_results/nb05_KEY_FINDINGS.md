# NB05 Key Findings — Soft vs Good Reactor Days

## Simple picture — good days vs soft days

|                     | Good days              | Soft days              |
|:--------------------|:-----------------------|:-----------------------|
| Reactor performance | Strong dioxane removal | Weaker dioxane removal |
| Phosphorus          | Higher                 | Lower                  |
| Nitrate/nitrite     | Higher                 | Lower                  |
| Ammonia             | Lower                  | Higher                 |
| Incoming dioxane    | Lower                  | Higher                 |
| Temperature         | A bit warmer           | A bit cooler           |

## Setup & classifier (default 95%)

| Item                          | Value                                    |
|:------------------------------|:-----------------------------------------|
| Soft-day definition (default) | removal_3320_3340 < 95.0%                |
| Soft / good / total           | 226 / 879 / 1105                         |
| Train / test                  | train ≤ 2016-12-31; test after           |
| Classifier features           | 15 chemistry/field features              |
| Excluded from classifier      | dioxane_3320, blend_ratio, sed_blend_gap |
| Test ROC-AUC @ 95%            | 0.758                                    |
| Test AUPRC @ 95%              | 0.401 (chance ≈ 0.178)                   |

## Soft vs good — key median differences (@ 95%)

| Feature                 |   Soft median |   Good median | Direction      |      p-value |
|:------------------------|--------------:|--------------:|:---------------|-------------:|
| Temperature (3320)      |       18.9    |       20.5    | Lower on soft  | 2.0943e-21   |
| pH (3320)               |        6.78   |        6.78   | Lower on soft  | 0.370635     |
| Ammonia (3310)          |      220      |      169.25   | Higher on soft | 0.000128049  |
| Nitrate/nitrite (3310)  |       50      |      240      | Lower on soft  | 6.23242e-09  |
| Phosphorus (3320)       |     1100      |     2200      | Lower on soft  | 9.17669e-23  |
| THF (3320)              |     2100      |     3200      | Lower on soft  | 2.88758e-29  |
| Incoming dioxane (3310) |    28000      |    17000      | Higher on soft | 1.10837e-15  |
| Dioxane (3320)          |     1800      |     2400      | Lower on soft  | 4.23645e-28  |
| Blend ratio             |        0.06   |        0.15   | Lower on soft  | 1.53203e-27  |
| Sed–blend gap           |    26600      |    13950      | Higher on soft | 3.45013e-17  |
| TSS (3310)              |       68      |       56      | Higher on soft | 3.19066e-07  |
| Dioxane (3340)          |      130      |       43      | Higher on soft | 1.40714e-101 |
| Removal 3310→3340       |       99.5455 |       99.7923 | Lower on soft  | 1.48601e-43  |

## SHAP top drivers (@ 95%, test)

|   Rank | Feature                 |   mean_|SHAP| |
|-------:|:------------------------|--------------:|
|      1 | Phosphorus (3320)       |        0.2336 |
|      2 | Nitrate/nitrite (3310)  |        0.1378 |
|      3 | Conductivity (3310)     |        0.1274 |
|      4 | Incoming dioxane (3310) |        0.1262 |
|      5 | TSS (3310)              |        0.1153 |
|      6 | Temperature (3320)      |        0.1143 |
|      7 | THF (3320)              |        0.1008 |
|      8 | Ammonia (3310)          |        0.0708 |

## Soft-threshold sensitivity

|   Soft if removal < | Soft rate   |   Test ROC-AUC |   Test AUPRC |
|--------------------:|:------------|---------------:|-------------:|
|                  90 | 3.9%        |          0.728 |        0.138 |
|                  92 | 6.4%        |          0.769 |        0.2   |
|                  95 | 20.5%       |          0.758 |        0.401 |
|                  97 | 43.0%       |          0.612 |        0.534 |
|                  98 | 58.3%       |          0.709 |        0.806 |

## SHAP feature stability across thresholds

| Feature                 | Times in top-5   |
|:------------------------|:-----------------|
| Phosphorus (3320)       | 5 / 5            |
| Incoming dioxane (3310) | 4 / 5            |
| Conductivity (3310)     | 4 / 5            |
| Nitrate/nitrite (3310)  | 4 / 5            |
| 1,2-DCA (3310)          | 2 / 5            |
| THF (3310)              | 2 / 5            |
| THF (3320)              | 2 / 5            |
| TSS (3310)              | 1 / 5            |
| Temperature (3320)      | 1 / 5            |

## Bottom line

SHAP identifies stable soft-day chemistry drivers — especially **phosphorus (3320)** and **nitrate+nitrite (3310)** — giving concrete microbial hypotheses to test when more 16S data arrive.

Limits: associations only; not causal. Classifier excludes dioxane_3320, blend_ratio, sed_blend_gap.