# TF-Li-Ion v1.0.0 (2026-09-28)

This is the first public release of the reproducibility package for the article *What Tempering Can and Cannot
Guarantee for Fractional-Order State-of-Charge Observers: Rate Ceiling, Certified Finite Memory and a Negative
Offline Test on CALCE Data* (under review).

## Contents

- `src/tfso/`: the Python package. It contains the tempered fractional calculus, the model with a tempered
  polarisation branch, parameter identification, the certified observer (LP/diagonal-LMI gain design, finite
  memory), the Kalman benchmarks (FOEKF, EKF-2RC, AEKF-2RC), metrics, OCV and relaxation analysis.
- `scripts/`: the numbered pipeline 00-13 and `reproduce_all.py` (full run, `--skip-heavy`, `--from-results`,
  `--dry-run`).
- `config/config.yaml`: every setting, split and seed, declared before the runs.
- `results/`: all saved results, from which every table, figure and report is rebuilt.
- `figures/`, `tables/`, `reports/`: generated outputs and the findings of each study.
- `technical_report/`: the complete theory (75 pages) and the extended methods and results (108 pages).
- `tests/`: 206 tests. Without the CALCE data, 109 pass and 97 data-layer tests are skipped with a reason.

## Main results

- **Proved.** Certified gains make the continuous full-memory observer exponentially stable at every rate up to
  lambda. No nonnegative gains give a faster rate for a nonzero initial polarisation error. A finite memory is
  certified without tempering; tempering lowers the truncation bound to order `exp(-lambda L) L^(-1-alpha)`.
- **Negative offline test** (CALCE INR18650-20R, 18 held-out records, 0/25/45 C).
  - lambda was not identifiable.
  - No lambda > 0 beat lambda = 0 on all records of any temperature for memories up to 800 s.
  - At L = 400 s the mean SOC RMSE was 1.375 % (tempered) against 1.368 % (untempered).
  - Three Kalman filters were more accurate.

## Data

The CALCE records are not redistributed. See README section 4 for download, checksums (`data_manifest.json`) and
the required citations. The shipped result archives omit the measured voltage, current and time channels, and the
model voltages from which the current could be recovered. The SOC trajectories of the test records are kept on a
60 s grid. Without the data, every shipped table, 12 of the 14 figures of the extended technical report and article
figures 3-5 are rebuilt. The draft figures of steps 02-04 and the figures drawn from measured channels need the
data (README section 6).

## Changes in the scripts for the public release

- `scripts/reproduce_all.py --from-results` and `scripts/09_figures_tables.py` now detect result archives without
  the measured channels. They skip the steps and figures that need those channels and print a message instead of
  failing.
- `scripts/make_release.py` removes the pdfTeX `/PTEX.*` entries (local file paths of included figures) from the
  shipped PDFs. It also keeps the SOC trajectories on a 60 s grid, drops the current-derived model voltages and
  ships the envelope of article Fig. 3(b) precomputed.
- `scripts/10_paper_figures.py` skips figure 2 with a message when the measured channels are absent.
- `scripts/05_design_tune.py --report-only` keeps the recorded freeze time of `results/frozen_config.json`.

## Licence

MIT. The CALCE data remain subject to the terms on the CALCE data page.
