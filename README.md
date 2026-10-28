# Average reservoir pressure in shale wells without extended shut-in

Code and data for the study *Average reservoir pressure in multi-fractured shale wells without extended shut-in: closed-form
buildup theory, physics-constrained machine learning and application to HFTS-1*, by Emmanuel Fred. The paper is being prepared for
journal submission; the citation will be added here once it is published.

## The problem

Average reservoir pressure is rarely measured in shale wells. A buildup long enough to stabilise costs months of production, so
operators stop after one to four days and the last shut-in pressure is often treated as the static pressure. This study asks how
wrong that pressure is and whether the missing part can be recovered from data an operator already has.

It does four things:

1. **Defines the target.** Under pad symmetry, each multi-fractured horizontal well drains closed elements whose stabilised shut-in
   pressure equals their mass-equivalent average pressure. The target is the average over the stimulated slab between fractures.
2. **Solves the slab buildup in closed form.** After boundary-dominated drawdown the recovered fraction of the pressure deficit
   follows a universal law of dimensionless shut-in time, and a short buildup identifies only the ratio of the deficit to the square
   root of the slab diffusion time. The decline constant of flowing material balance supplies that time, which gives an estimator
   with no rock, fluid or geometric input.
3. **Tests estimators on 2,997 verified finite-volume simulations** (finite-conductivity fractures, outer rock, wellbore storage,
   declining bottomhole pressure) under case-grouped cross-validation, including a physics-constrained gradient-boosted model
   (ML-P) with conformal prediction intervals.
4. **Applies them to public bottomhole-gauge data** from the Hydraulic Fracturing Test Site (HFTS-1), Midland Basin.

## Main results

- A 72-h shut-in recovers a median 42 % of the deficit and understates the stimulated-slab average by 695 psi (4.8 MPa).
- The universal recovery law gives 50 % and 90 % recovery at dimensionless shut-in times of 0.036 and 0.192.
- ML-P cuts the median 72-h error of the deficit to 14 % (120 psi), against 37–41 % for the Horner, square-root-decline and
  flowing-material-balance estimates; its 80 % intervals cover 80 % of cases.
- At HFTS-1, ML-P places the slab average of nine wells at 1,490–2,370 psia, between the buildup-end and core-well pressures.
- Open issue: in eight of ten wells the estimate from the first 24 h is 120–850 psi higher than the 72-h estimate. The simulations
  do not show this drift, and the study does not explain it; a well with a stabilised buildup is needed to settle it.

## Repository layout

| Folder | Contents |
| --- | --- |
| `03_Raw_Data/field` | HFTS-1 bottomhole-gauge records at native resolution (31 Jul–16 Aug 2016), gauge inventory, approximate buildup windows, testing program, and raw monthly lease production from the Railroad Commission of Texas. |
| `03_Raw_Data/synthetic` | Raw outputs of all 3,356 simulations (parameters and targets, production histories, buildups) for the main, nonlinear-compressibility and extrapolation sets. |
| `04_Clean_Data` | Feature tables used for learning, cleaned field inputs, field estimates, cross-validated and out-of-distribution predictions, all metrics, the trained models (`models_final.pkl`) and run logs. `DATA_DICTIONARY.md` describes every column of every raw and clean file. |
| `05_Code` | `Average_Pressure_Pipeline.ipynb`: theory, simulator, data generation, estimators, learning, field application and figures, saved with its outputs. `requirements.txt`. `outputs/`: files written by the last run. |
| `06_Scenarios` | `Scenarios_and_Results.xlsx`: every scenario, its parameter ranges and results, and the field Monte Carlo results. |
| `07_Figures` | The nine figures as PNG (300 dpi) and vector PDF, with `FIGURE_MAP.md`. |

## Running the code

```bash
pip install -r 05_Code/requirements.txt
cd 05_Code
jupyter notebook Average_Pressure_Pipeline.ipynb   # then run all cells
```

Run the notebook from inside `05_Code`; its paths are relative to the repository root. With the default switches it takes about
15 minutes, rebuilds the tables and all nine figures into `05_Code/outputs`, and checks itself against the stored results. In the
last test run every metric and every field estimate was reproduced exactly. scikit-learn 1.9.1 is needed to load the saved models
exactly.

Switches at the top of the notebook:

- `REGENERATE_SYNTHETIC = True` reruns all 3,356 simulations (several hours; seeds 20261024, 777 and 999).
- `REBUILD_FEATURE_TABLES = True` rebuilds the feature tables from the raw simulations.
- `RETRAIN = False` loads the saved models instead of retraining them.

## Data sources and licences

- **Code:** MIT licence (`LICENSE`).
- **Data and figures produced for this study** (synthetic data, clean data, model outputs, scenarios, figures): CC BY 4.0
  (`LICENSE-DATA.md`).
- **HFTS-1 data** in `03_Raw_Data/field`: GTI–NETL Hydraulic Fracturing Test Site (HFTS) Project #DE-FE0024292, *HFTS-1 Phase 1 BH
  Pressure Data*, 2020, NETL Energy Data eXchange, https://edx.netl.doe.gov/dataset/hfts-1-phase-1-bh-pressure-data, released
  under Creative Commons Attribution. Please cite it if you use these files.
- **Lease production:** Railroad Commission of Texas, Production Data Query, oil lease 16507 (public record), accessed 24 October 2026.

## Citation

See `CITATION.cff`, or use GitHub's "Cite this repository" button.

## Author

Emmanuel Fred
