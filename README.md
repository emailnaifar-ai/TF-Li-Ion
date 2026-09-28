# TF-Li-Ion: tempered fractional-order SOC observers, reproducibility package

Code, configuration, saved results, figures, tables, reports and technical reports for the article

> **What Tempering Can and Cannot Guarantee for Fractional-Order State-of-Charge Observers: Rate Ceiling,
> Certified Finite Memory and a Negative Offline Test on CALCE Data**
> (manuscript under review; the article reference will be added here once it is published).

Repository: https://github.com/emailnaifar-ai/TF-Li-Ion

## 1. Summary

Fractional-order battery models capture slow relaxation. However, their power-law memory rules out exponential
observer stability, and the truncation error of a finite memory decays only algebraically. The study asks what
tempering the kernel by `exp(-lambda t)` can and cannot guarantee for a state-of-charge (SOC) observer, and whether
it improves accuracy.

**Proved.**
(i) *Rate ceiling.* A lossless linear-programming/diagonal-LMI reformulation of a 2x2 positive comparison
certificate gives gains that make the continuous full-memory observer exponentially stable at every rate up to
lambda, uniformly over bounded OCV slopes. For any nonnegative gains, no trajectory with a nonzero initial
polarisation error decays faster than rate lambda.
(ii) *Certified finite memory.* Under a discrete certificate, a finite memory L makes the homogeneous error
recursion exponentially stable even without tempering. Tempering lowers the truncation bound to order
`exp(-lambda L) L^(-1-alpha)`. The implemented recursion is certified with the truncation and the plant residual
treated as bounded inputs.

**What the data showed (negative result).** The offline test used one dynamic CALCE INR18650-20R cell (SP2),
with the OCV taken from two sister cells (SP1, SP3), and 18 held-out FUDS/US06/BJDST records at 0, 25 and 45 C.
- The relaxation tails did not support a tempered law, and lambda was not identifiable.
- No re-identified, re-tuned lambda > 0 beat lambda = 0 on all records of any temperature for memories up to
  L = 800 s.
- At L = 400 s the tempered and untempered observers were practically equivalent: mean SOC RMSE 1.375 % (TFO) and
  1.368 % (FOO); the paired difference is +0.007 % SOC, TFO worse in 17 of 18 records.
- Fractional memory beat integer-order memory in the same observer (removing it: median +0.841 % SOC, worse in
  18 of 18 records).
- Three Kalman filters were more accurate: mean SOC RMSE 0.899 % (FOEKF), 1.019 % (EKF-2RC) and 1.236 % (AEKF-2RC).
- The 400 s memory is certified for stability but not for a 0.5 % SOC truncation tolerance: the truncation bound
  is 12 to 13 times the actual effect.

Sources: `reports/validation_main_findings.md`, `reports/ablation_findings.md`, `reports/relaxation_findings.md`,
`reports/identification_findings.md`, `reports/conservatism_findings.md`, `reports/claims_to_evidence.md`.

## 2. Repository layout

```
src/tfso/            Python package: data layer (checks), tempered calculus, model, identification, observer,
                     Kalman benchmarks, metrics, OCV, relaxation, experiments, plot style, table formatting
scripts/             numbered pipeline 00 ... 13, reproduce_all.py, make_release.py;
                     README_pipeline.md lists every script with inputs, outputs and recorded runtime
config/config.yaml   parameter bounds, data splits, seeds and every experiment setting (declared before the runs)
results/             machine-readable results (JSON/NPZ) from which every figure, table and report is built;
                     results/archive/ holds the pre-review snapshots
figures/, tables/    generated figures (PDF) and LaTeX tables (paper_* = article, supp_tab_* = supplementary
                     material, fig_*/tab_* = extended technical report)
reports/             findings report of every study, dataset audit, literature search,
                     figure_table_manifest.md (figure/table -> code -> result fields)
tests/               pytest suites (data layer, tempered calculus, model and identification, estimators)
technical_report/    technical_report_theory.pdf, technical_report_extended.pdf (section 9)
data_manifest.json   the 33 logical records: archive, workbook, md5 of workbook and processed CSV, split
environment.yml, requirements.txt   pinned environment
RELEASE_MANIFEST.txt file list of this release (scripts/make_release.py)
```

Not in the repository: `dataset/`, `data_raw/`, `data_processed/` (the CALCE records, section 4) and the manuscript
sources (the article and its supplementary material are distributed by the journal).

## 3. Installation

Python 3.12 (results produced with 3.12.10 on Windows 11).

```
conda env create -f environment.yml && conda activate tfso
# or
python -m pip install -r requirements.txt
```

Pinned packages: numpy, scipy, pandas, matplotlib, statsmodels, mpmath, cvxpy with Clarabel/SCS, cma, PyYAML,
psutil, xlrd, openpyxl, PyMuPDF, pytest. All commands below are run from the repository root.

## 4. CALCE data (not redistributed)

The measurements belong to the CALCE Battery Research Group (University of Maryland) and are not included here.

1. Open https://calce.umd.edu/battery-data, section *INR 18650-20R* (Samsung, 2000 mAh; samples SP1, SP2, SP3).
2. Download the 21 archives listed in `data_manifest.json` (field `source_zip`). These are the incremental-OCV
   tests of SP1/SP3 at 0, 25 and 45 C, the initial-capacity tests of SP1/SP2/SP3, and the DST, FUDS, US06 and
   BJDST profiles of SP2 at 0, 25 and 45 C. Place them, unmodified, in `dataset/`.
3. `python scripts/reproduce_all.py --extract --dry-run` extracts the workbooks into `data_raw/<archive>/` and
   checks them. `python scripts/00_convert_raw.py` then writes one CSV per workbook to `data_processed/`.
4. Before running, `reproduce_all.py` checks every workbook and CSV against the md5 sums in `data_manifest.json`.
   Every analysis script also calls `tfso.data.verify_file` before using a file. It checks columns, value ranges,
   missing values, current sign, monotone test time and step indices.

**Citation requirement.** Use of the data is subject to the terms on the CALCE data page. Cite the publications
CALCE requests for this dataset:

- F. Zheng, Y. Xing, J. Jiang, B. Sun, J. Kim, M. Pecht, Influence of different open circuit voltage tests on state
  of charge online estimation for lithium-ion batteries, *Applied Energy* 183 (2016) 513-525,
  https://doi.org/10.1016/j.apenergy.2016.09.010
- Y. Xing, W. He, M. Pecht, K. L. Tsui, State of charge estimation of lithium-ion batteries using the open-circuit
  voltage at various ambient temperatures, *Applied Energy* 113 (2014) 106-115,
  https://doi.org/10.1016/j.apenergy.2013.07.008
- W. He, N. Williard, C. Chen, M. Pecht, State of charge estimation for Li-ion batteries using neural network
  modeling and unscented Kalman filter-based error cancellation, *International Journal of Electrical Power &
  Energy Systems* 62 (2014) 783-791, https://doi.org/10.1016/j.ijepes.2014.04.059

## 5. Reproducing the results

```
python scripts/reproduce_all.py --from-results             # rebuild tables, figures and reports from results/
python scripts/reproduce_all.py --from-results --dry-run   # print the plan and the data check only
python scripts/reproduce_all.py --skip-heavy               # re-run the analyses except the five longest steps (data needed)
python scripts/reproduce_all.py                            # full run from the data (recorded step times: about 2.7 h)
```

The article-specific outputs are rebuilt from the saved results by:

```
python scripts/10_paper_figures.py [--only 2,3,4,5]   # figures/paper_fig2-5_*.pdf, results/paper_figures.json
python scripts/10_paper_tables.py                     # tables/paper_tab*.tex
python scripts/11_clustered_stats.py --tables-only    # tables/paper_tab_stats.tex, supp_tab_stats_full.tex
python scripts/11_conservatism.py --report-only       # reports/conservatism_findings.md
python scripts/12_supp_tables_B.py                    # supp/B_tab_*.tex (supplement tables, see below)
python scripts/13_review_checks.py                    # results/review_checks.json
```

`12_supp_tables_B.py` writes the record-level and conservatism tables of the article's supplementary material to
a folder `supp/`. The supplement itself is distributed by the journal, so `supp/` is not part of the repository and
is ignored by git.

`reproduce_all.py` writes the return code and wall time of every step to `reports/reproduce_log.json`.
`scripts/README_pipeline.md` lists the inputs, outputs and recorded runtime of every script.

## 6. What runs without the CALCE data

The result archives `results/identification_arrays.npz`, `results/validation_series.npz` and
`results/relaxation_curves.npz` are shipped without the measured voltage, current and time channels. Signals from
which the measured current could be recovered are also reduced (`scripts/make_release.py`, module docstring):

- The SOC trajectories of the test records in `validation_series.npz` (`t_soc`, reference SOC `zref*` and
  estimates `zhat|*`) are kept on a 60 s grid (every 60th sample of the 1 s grid). A coulomb-counted SOC on the
  1 s grid would give back the measured current through I = -Q dz/dt; on the 60 s grid only 60 s mean currents
  can be derived.
- The model output voltages (`<record>|Vt_*`, `acc__*__V`) and the reference SOC of the identification records
  (`acc__*__z`) are not shipped. They are used only together with the measured voltage.
- The median |SOC error| envelope of article Fig. 3(b), computed from the 1 s trajectories, is shipped as
  `paper_fig3_env|*` in `validation_series.npz`. The 10 s reference SOC of the robustness figure record
  (`robustness.json`, `figure.traj.zref`, not plotted) is removed.
- `figures/fig_soc_test.pdf` and `figures/paper_fig3_soc.pdf` are drawn from the 60 s trajectories. At the
  printed size they match the figures drawn from the 1 s grid.

Estimation errors, fitted parameters, statistics and fit grids are kept unchanged. The following was tested on a
fresh copy of this repository with no data (Python 3.12.10, Windows 11):

**Regenerated without the data**
- All generated tables (`tables/*.tex`), byte-identical to the shipped files. They are written by step 09 of
  `reproduce_all.py --from-results` and by `10_paper_tables.py` and `11_clustered_stats.py --tables-only`.
  `12_supp_tables_B.py` also runs and writes the three supplement tables to `supp/` (section 5).
- 12 of the 14 figures of the extended technical report (step 09), and the article figures 3-5
  (`10_paper_figures.py`, which skips figure 2 with a message; `--only 3,4,5` gives the same result).
- The reports of steps 02b, 04b, 05, 07, 08 and `11_conservatism.py --report-only`, and `results/review_checks.json`.
  These are identical to the shipped files except for generation time stamps and one code hash. Step 05 in this
  mode (`05_design_tune.py --report-only`) keeps the recorded freeze time (`generated_utc`, `frozen_utc`) of
  `results/frozen_config.json` (section 8).
- `python -m pytest tests -q`: 109 passed, 97 skipped. All skips are the data-layer tests in `tests/test_data.py`,
  each with the reason "CALCE records not found".

**Needs the data** (download them as in section 4, then run the full pipeline)
- Article figure 2 (`paper_fig2_relaxation.pdf`) and the report figures `fig:relax` (`fig_relax.pdf`) and
  `fig:vmodel` (`fig_vt_model.pdf`). They are drawn from the measured channels. Step 09 and
  `10_paper_figures.py` skip them with a message.
- The draft figures of steps 02-04 (`figures/draft_*.pdf` except `draft_p1_*` and `draft_p2_*`, and
  `fig_ident_corr`, `fig_ident_params_T`, `fig_ident_profile`, `fig_ident_timing`, `fig_vmodel_acf`,
  `fig_vmodel_fit`, `fig_vmodel_rmse`). They are written by the analysis steps themselves.
- The from-results variants of steps 03, 04 and 06. `reproduce_all.py` skips them with a message, because their
  tables are rebuilt by step 09.
- Every analysis step itself: 00-08 in the full mode, `--skip-heavy`, the auxiliary checks and
  `11_conservatism.py` without `--report-only`.

With the data present and the full pipeline run, the archives again contain the measured channels and the 1 s
trajectories, and every figure is rebuilt.

## 7. Tests

```
python -m pytest tests -q
```

`test_tempered.py` covers the tempered calculus. `test_model_identify.py` covers the model and identification, and
`test_estimators.py` covers the observers and Kalman benchmarks. These run without data. `test_data.py` tests the
data layer against the CALCE records; without the data it is skipped with a reason.

## 8. Seeds, splits and time conventions

- **Seeds.** All seeds are in `config/config.yaml`: base seed 20260927; the identification multi-start (LHS), DE
  and CMA-ES seeds; the bootstrap seeds; the tuning seeds; and the 20 Monte-Carlo noise seeds of the robustness
  study. Per-task streams come from `numpy.random.SeedSequence` or from seed offsets documented next to each seed.
- **Splits** (`config/config.yaml: splits`).
  - OCV and relaxation: SP1/SP3.
  - Identification: `SP2_*_DST_80SOC`.
  - Hyper-parameter selection and tuning: `SP2_*_DST_50SOC` only.
  - Test: the 18 FUDS, US06 and BJDST records of SP2. These were loaded only after the configuration was frozen
    (`results/frozen_config.json`).
- **Signal time.** The records are resampled to a uniform 1 s grid (h = 1 s), and inputs enter the model causally.
- **Runtime measurement.** Script runtimes are wall times from `time.perf_counter`; CPU times come from
  `time.process_time` (worker CPU summed over tasks).
- **Estimator cost.** The per-sample costs are `time.process_time` over the estimation loop only: median and IQR
  over 21 repeats in one process on a desktop CPU (`results/machine.json`). They are indicative, not embedded
  (MCU) timings.

## 9. Technical reports

`technical_report/` holds the public long-form documents, which complement the article:

- `technical_report_theory.pdf` (75 pages): the complete mathematical development, with full statements and
  proofs, worked examples and numerical cross-checks.
- `technical_report_extended.pdf` (108 pages): the extended methods, results and audit. It covers the data audit,
  extended identification, validation, ablation, robustness, cost and the literature search.

Their figures and tables are rebuilt by `scripts/09_figures_tables.py`. The map from each figure or table to its
code and result fields is in `reports/figure_table_manifest.md`.

## 10. How to cite

Use the metadata in `CITATION.cff` (GitHub shows it as "Cite this repository"). Until the article is published,
cite this repository by its URL, https://github.com/emailnaifar-ai/TF-Li-Ion, version 1.0.0. When you use the data,
also cite the CALCE publications in section 4.

## 11. Licence

The code and documentation are released under the MIT licence (`LICENSE`), Copyright (c) 2026 The TF-Li-Ion
authors. The CALCE data are not covered by this licence and remain subject to the terms on the CALCE data page.
