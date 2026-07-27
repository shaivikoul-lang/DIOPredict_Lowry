# DIOPredict — Lowry Landfill

Analysis of 1,4-dioxane biodegradation at the full-scale groundwater treatment plant on the
Lowry Landfill Superfund Site (Colorado, USA), combining two decades of process chemistry with
16S rRNA microbial community sequencing.

## Background

1,4-dioxane is a probable human carcinogen and a persistent groundwater contaminant. It resists
conventional treatment, so biological degradation is of significant interest — but almost all
published work is bench-scale, single-isolate, or short pilot studies. The Lowry facility is a
working, full-scale pump-and-treat plant that has been removing dioxane by biodegradation for
over twenty years, which makes it an unusually good place to study how the process behaves at
scale and over time.

The guiding question for this project is whether **microbial community composition explains
plant performance** — specifically whether the abundance of known degraders (*Pseudonocardia*)
or of nitrifiers capable of co-metabolic oxidation (*Nitrospira*, *Nitrosocosmicus*) tracks how
much dioxane the plant removes.

### Plant layout

Water moves through four sampling points referenced throughout the code:

| Port | Stage | Role in the analysis |
|------|-------|----------------------|
| TP-3310 | Sedimentation tank | Influent — predictor variables |
| TP-3320 | Blend tank | Reactor feed, after roughly 10–20× dilution |
| R1 / R2 / R3 | Bioreactors | Where the 16S samples come from |
| TP-3340 | Combined effluent | Outcome side — excluded from predictors |

The headline outcome variable is `removal_3310_3340`, the percentage of dioxane removed between
influent and effluent.

## Contents

```
analysis/notebooks/
  01_chemistry_3310_3320_3340.ipynb    process chemistry pipeline
  02_16S_Fall2025.ipynb                16S microbial community pipeline
  03_integration_Fall2025.ipynb        join, explore, and model
data/processed/
  integrated_plant_by_date.csv         the merged analysis table
```

### The pipeline

**01 — Chemistry.** Reads three Excel workbooks (one per sampling port) and turns them into a
clean table with one row per date. The main work is quality control: the lab's `FINAL Q`
qualifier is applied so that rejected (`R`) results are dropped, non-detects (`U`) are
substituted at half the reporting limit, and estimated (`J`) values are kept as reported. Dates
are normalized, same-day duplicates averaged, and the result pivoted wide. Produces roughly
1,245 dates across 335 columns of dioxane, co-contaminant, and field measurements.

**02 — Microbiology.** Parses a Silva-annotated 16S ASV table down to genus level, drops samples
below 5,000 reads, converts counts to relative abundance, averages technical replicates, and
computes Shannon diversity and richness per date and reactor.

**03 — Integration.** Joins the two on sampling date, writes `integrated_plant_by_date.csv`, and
runs exploratory correlation and random-forest analyses against removal.

### The data file

`integrated_plant_by_date.csv` is the merged table: **9 rows × 352 columns**, one row per date
where both chemistry and microbiology exist, spanning 2022-06-14 to 2025-07-22.

Columns are suffixed by sampling port — `*_3310` (187 influent), `*_3320` (19 blend tank),
`*_3340` (128 effluent) — plus 15 `micro_*` genus relative abundances and two diversity metrics.
The effluent columns are retained for reference but are deliberately **excluded from the
predictor set**, since they sit downstream of the outcome and would leak it into any model.

## Status and findings

**The community–performance link is not supported by the current data.** Only nine dates have
both chemistry and microbiology, and on those nine rows with 43 candidate features the random
forest scores a negative leave-one-out R² — worse than predicting the mean. No correlation
survives correction for multiple comparisons. *Pseudonocardia* abundance against bioreactor-stage
removal comes out at r ≈ 0.05.

Three structural problems explain why, and they matter more than the null result itself:

- **Almost no outcome variance.** Removal sits between 99.14% and 99.93% on every date on
  record. The plant essentially always works, so there is little performance variation for
  community composition to explain.
- **Mismatched units of observation.** The 16S samples come from reactor R1, while removal is
  measured on the blended effluent of parallel trains.
- **Relative, not absolute, abundance.** A change in a genus's share of reads cannot distinguish
  "more degraders" from "fewer of everything else." Confirming activity would need qPCR of the
  dioxane monooxygenase genes or a functional readout.

By contrast, the long-term chemistry record on its own is strong: performance is remarkably
robust, holding above 99% removal across large swings in influent loading. That record, rather
than the nine-date microbial join, is where this dataset currently has the most to say.

## Running the notebooks

```bash
python -m venv .venv
source .venv/bin/activate
pip install pandas numpy matplotlib seaborn openpyxl scikit-learn jupyter
```

Notebooks locate the project root by walking up until they find a `data/processed` directory, so
they should be run from within the repository.

**Note on data availability.** Notebooks 01 and 02 read raw source files — the chemistry Excel
workbooks and the 16S ASV table — that are **not included here**, as the underlying site data is
confidential. Only the derived `integrated_plant_by_date.csv` is published. The two pipeline
notebooks are included for methodological transparency and will not execute end-to-end without
those raw inputs.

## Related work

Romero, J. L., Ratliff, J. H., Carlson, C. J., Griffiths, D. R., Miller, C. S., Mosier, A. C., &
Roane, T. M. (2025). Community and functional stability in a working bioreactor degrading
1,4-dioxane at the Lowry Landfill Superfund Site. *Applied and Environmental Microbiology*,
91(10). https://doi.org/10.1128/aem.00574-25

That paper characterizes the microbial community and its degradation genes at this facility. The
present work is the process-performance counterpart, asking whether that community explains how
the plant actually behaves.

## Collaborators

Prof. Christopher Miller (UC Denver) — 16S microbial data and process expertise.
Prof. Farnoush Banaei-Kashani (UC Denver) — computer science and modeling.
