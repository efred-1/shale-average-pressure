# Data dictionary

Units: pressure psia (psi for differences), time in days (d) or hours (h) as named, rates bbl/d, permeability md
(0.001 md = 1 μD), lengths ft, compressibility 1/psi, viscosity cp. Missing values are blank.

## 03_Raw_Data/field

| File | Column | Meaning |
| --- | --- | --- |
| HFTS1_BHP_raw_2016-07-31_to_2016-08-16.csv | well | Well name (3SU … 8SU producers; 6TW slant core well) |
| | time | Gauge timestamp (local time as released) |
| | p_psia | Gauge pressure, psia, at native resolution (10 s or 60 s for producers) |
| | t_degF | Gauge temperature, °F |
| | gauge | Gauge identifier (6TW carries several gauges at different depths) |
| HFTS1_gauge_inventory.csv | well, first, last, n | Gauge record span and number of readings for the full release (Dec 2015–May 2017) |
| | median_dt_s, gaps_over_1d | Median sampling interval (s); number of gaps longer than one day |
| | p_min, p_median, p_max | Pressure range over the full record, psia |
| HFTS1_aug2016_buildup_windows_approx.csv | well, shut_in, reopened, note | Approximate windows of the August 2016 buildups that the exact shut-in detector searches (Section 7.1) |
| RRC_lease_16507_monthly_production_raw.csv | month | Production month (e.g. "Jan 2016") |
| | oil_bbl, oil_disp_bbl | Lease oil production and disposition, bbl/month |
| | gas_mcf, gas_disp_mcf | Lease gas production and disposition, Mcf/month |
| HFTS1_testing_program.xlsx | — | GTI testing-program diagram (stages, cluster designs, diagnostics) as released |

Full-period raw archives (about 600 MB) are not repackaged; they are public on NETL EDX (GTI–NETL, 2020): datasets
"HFTS-1 Phase 1 BH Pressure Data", "HFTS-1 Phase 1 Test Wells Flowback Reports" and "HFTS-1 Phase 1 Individual Well Files".

## 03_Raw_Data/synthetic/{main, nonlinear, extrap}

| File | Column | Meaning |
| --- | --- | --- |
| cases_parameters_and_targets.csv | set, case | Scenario (main / nonlinear / extrap) and case number (seed index) |
| | pbar_S_psia | **Target**: stimulated-slab average pressure at shut-in, Eq. (4) |
| | pbar_E_psia | Element average pressure at shut-in |
| | k, phi, mu, ct | Matrix permeability (md), porosity, viscosity (cp), total compressibility (1/psi) |
| | L, xf, Ye, h | Half fracture spacing, fracture half-length, half well spacing, height (ft) |
| | Fc | Fracture conductivity (md·ft) |
| | lateral, n_elem | Lateral length (ft); number of quarter-fracture elements = 4·lateral/(2L) |
| | C_well, Cw | Wellbore storage of the whole well and per element (bbl/psi) |
| | pi | Initial pressure (psia) |
| | tp | Production time before shut-in (d) |
| | p_start, p_end, tau | Bottomhole-pressure schedule p_wf(t) = p_end + (p_start − p_end)·exp(−t/tau) (psia, psia, d) |
| | choke_steps_t_days, choke_steps_dp_psi | Times (d) and sizes (psi) of the tanh-smoothed choke steps, ";"-separated |
| | pb, ct_gas_mult | Nonlinear set only: bubble point (psia) and compressibility multiplier, Eq. (15) |
| | cells | Number of grid cells |
| production_histories.csv | case, t_days | Case and time since start of production (d) |
| | pwf_psia | Bottomhole pressure (psia) |
| | q_sandface_bbl_d, q_surface_bbl_d | Well rate at the sandface and at surface (bbl/d); the first steps include wellbore-storage unloading |
| | pbar_S_psia, pbar_E_psia | Slab and element averages, stored every fifth step and at shut-in (blank otherwise) |
| buildups.csv | case, dt_h, pws_psia | Shut-in pressure at 94 times from 0.01 h to 504 h |

## 04_Clean_Data/synthetic/feature_table_{main, nonlinear, extrap, main_pierr15}.csv

One row per case and buildup duration N (24, 72, 168 h). `main_pierr15` is the main set observed with a 15 % initial-pressure error.

| Column | Meaning |
| --- | --- |
| case, N | Case number; buildup duration used (h) |
| pwf, pN | Last flowing pressure; pressure at the end of the N-hour buildup (the proxy), psia |
| pbar_srv, pbar_el | True slab and element averages (psia); pbar_srv is the target |
| pi | True initial pressure (psia) |
| z | Learning target, Eq. (16): ln[(pbar_srv − pwf)/(pN − pwf)] |
| tauL_h, dtD, tpD | Slab diffusion time L²/η (h); Δt_D = ηN/L²; t_pD = ηt_p/L² |
| par_* | Case parameters (see the raw parameter file) |
| logN, logtp | ln N (h); ln t_p (d) |
| expo | Late log–log slope of the buildup |
| log_pi_ratio | ln[(observed p_i − pwf)/(pN − pwf)] |
| n_months | Number of monthly rate observations |
| s0 … s7 | Normalised buildup shape (p_ws(fN) − pwf)/(pN − pwf) at f = 0.01, 0.02, 0.05, 0.1, 0.2, 0.35, 0.5, 0.7 |
| q0 … q5 | Last six monthly rates normalised by the peak monthly rate |
| pw0 … pw2 | Flowing pressure at t_p/4, t_p/2, 3t_p/4, normalised by (observed p_i − pwf) |
| logDN, logDtp | ln(D·N) and ln(D·t_p), with D the FMB decline constant (1/d) |
| z_horner, z_fmb, z_sqrt, z_slab | z implied by each physics estimator |
| est_proxy, est_horner, est_fmb, est_sqrt, est_slab | Physics estimates of p̄ (psia) |
| est_D | FMB decline constant D (1/d) |

## 04_Clean_Data/model_outputs

| File | Content |
| --- | --- |
| predictions_cv_and_ood.csv | Out-of-fold predictions (set = cv, fold 0–4) and out-of-distribution predictions (set = nonlinear / extrap). Columns: identifiers and truths as above; proxy, horner, fmb, sqrt, slab, ml_data (ML-D), ml_phys (ML-P), ml_p10 and ml_p90 (80 % conformal interval), psia |
| metrics_all.csv | Metrics per set, fold, N and method: n, fail (undefined fraction), bias (median e), mae_rel (median abs e), p90_rel, mae_psi, within10, within25, coverage, width_rel (interval width / deficit) |
| models_final.pkl | Final ML-D, ML-P and quantile models trained on all main cases (scikit-learn 1.9.1), with Qc (conformal widening), physics base and feature lists |
| cv_log.txt, model_selection_log.txt | Console logs of the cross-validation and model-selection runs |

## 04_Clean_Data/field

| File | Column | Meaning |
| --- | --- | --- |
| HFTS1_hourly_median_BHP.csv | time, one column per gauge | Hourly median pressure (psia), December 2015–May 2017 |
| HFTS1_buildups_summary.csv | order | Processing order (keeps the Monte Carlo draws reproducible) |
| | well, shut_in, reopened_or_truncated, duration_h | Detected shut-in, end of the usable buildup, and its length (h) |
| | pwf_last_flowing_psia | Median pressure over the last flowing hour |
| | production_start, tp_days | First gauge reading and production time before shut-in (d) |
| | p_core_well_6TW_psia | Core-well pressure at the shut-in time |
| HFTS1_buildup_curves.csv | well, dt_h, pws_psia | Buildup pressures at the simulator's report times |
| HFTS1_flowing_pressure_histories.csv | well, t_days, pwf_psia | Daily median flowing pressure from production start to shut-in |
| HFTS1_pad_production_monthly.csv | month, oil_bbl, …, days | RRC lease data for January–September 2016 |
| | q_lease | Lease oil rate (bbl/d) |
| | q_pad_base_slow / mid / fast | Pad rate = lease rate minus the extrapolated older-well base (decline × 0.5 / 1 / 1.5) |
| HFTS1_field_estimates.csv | well, N, dur_h, tp_days, pwf, pN, p_core_well | Identifiers and observations (psia) |
| | {proxy, horner, fmb, sqrt, slab, ml_data, ml_phys}_{p10, p50, p90} | Percentiles of each estimate over 300 Monte Carlo draws (psia) |
| | ml_phys_lo_q, ml_phys_hi_q | 10th percentile of the conformal lower bounds and 90th percentile of the upper bounds |
| | quantile_inconsistent | True when the ML-P median lies outside that band (4SM at 72 h) |
| | ml_phys_lo, ml_phys_hi | Reported 80 % interval (band widened to include the median when flagged) |
| | D_p50 | Median FMB decline constant (1/d) |
