---
name: cross-method-runtime-benchmarking
description: Use when two normalization methods must be compared on identical data
  and the claim of interest is computational efficiency, e.g. timing UniFORM's automatic
  feature-level pipeline against MxNorm's B-spline registration on the same PRAD-CyCIF
  feature table.
when_to_use_negative:
- The comparison targets normalization quality (kBET, silhouette, distribution alignment)
  rather than computational cost — use the article's benchmarking-of-accuracy metrics
  instead.
- The dataset is pixel-level OME-TIFF data; the pixel-level UniFORM example dataset
  was not yet available and this runtime claim was established on feature-level PRAD-CyCIF
  data only.
- Only one method is installed or the two methods are run on different input formats/hardware
  — timing comparisons are only valid under identical conditions on the same data.
tools:
- name: UniFORM
  role: 'Timed method under test: executes automatic feature-level normalization (histogram
    calculation, functional-data registration, normalization) on PRAD-CyCIF data'
  repo: https://github.com/kunlunW/UniFORM
- name: MxNorm
  role: 'Comparison method under test: executes the registration (B-spline) normalization
    option on the same PRAD-CyCIF data'
  repo: https://github.com/ColemanRHarris/mxnorm
- name: Python
  role: Runtime environment (conda, Python 3.10) used to install UniFORM dependencies
    and time its repeated runs
  repo: https://github.com/kunlunW/UniFORM.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:19749fb2899dca76f839bfa5a90a6f95cd43f55579a7e93bcdbbc89f7de3a87d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - MxNorm
  - Python
  - UniFORM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-method-runtime-benchmarking

## Summary

Benchmark the wall-clock runtime of two competing normalization pipelines (UniFORM vs MxNorm) on the same dataset under identical conditions, aggregating repeated-run timings into mean ± SD to quantify computational efficiency (e.g., a ~4-fold speedup on PRAD-CyCIF).

## When to use

Use when two normalization methods must be compared on identical data and the claim of interest is computational efficiency, e.g. timing UniFORM's automatic feature-level pipeline against MxNorm's B-spline registration on the same PRAD-CyCIF feature table.

## When NOT to use

- The comparison targets normalization quality (kBET, silhouette, distribution alignment) rather than computational cost — use the article's benchmarking-of-accuracy metrics instead.
- The dataset is pixel-level OME-TIFF data; the pixel-level UniFORM example dataset was not yet available and this runtime claim was established on feature-level PRAD-CyCIF data only.
- Only one method is installed or the two methods are run on different input formats/hardware — timing comparisons are only valid under identical conditions on the same data.

## Inputs

- PRAD-CyCIF feature-level dataset (Pickle *.pkl and AnnData *.h5ad) from Zenodo record 16383766
- Installed UniFORM Python 3.10 conda environment (github.com/kunlunW/UniFORM)
- Installed MxNorm R package (github.com/ColemanRHarris/mxnorm)

## Outputs

- runtime_benchmark.csv with per-method repeated-run timings
- mean ± SD runtime per method (e.g., UniFORM 30.58 ± 0.30 s; MxNorm 114.75 ± 4.35 s)
- speedup ratio (UniFORM ≈ 3–4× faster than MxNorm)

## How to apply

Install both competitors independently: clone kunlunW/UniFORM, create a Python 3.10 conda environment, and pip install -r requirements.txt; install MxNorm via R (install.packages('mxnorm') from CRAN or devtools::install_github('ColemanRHarris/mxnorm'), noting its CMake dependency for lme4/nloptr). Download the PRAD-CyCIF feature-level dataset (Pickle *.pkl and AnnData *.h5ad) from Zenodo record 16383766 so both methods consume the same input. Run UniFORM's automatic feature-level normalization (histogram calculation, functional-data registration, normalization) several repeated times, then run MxNorm's registration (B-spline) option an equal number of times, timing each run identically. Aggregate per-run timings into mean ± SD per method (reported: UniFORM 30.58 ± 0.30 s vs MxNorm 114.75 ± 4.35 s, ~4-fold improvement) and write runtime_benchmark.csv for tolerance-checked validation.

## Related tools

- **UniFORM** (Timed method under test: executes automatic feature-level normalization (histogram calculation, functional-data registration, normalization) on PRAD-CyCIF data) — https://github.com/kunlunW/UniFORM
- **MxNorm** (Comparison method under test: executes the registration (B-spline) normalization option on the same PRAD-CyCIF data) — https://github.com/ColemanRHarris/mxnorm
- **Python** (Runtime environment (conda, Python 3.10) used to install UniFORM dependencies and time its repeated runs) — https://github.com/kunlunW/UniFORM.git

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git && conda create -n uniform python=3.10 && pip install -r requirements.txt  # then time repeated UniFORM and MxNorm runs on the PRAD-CyCIF Zenodo data
```

## Evaluation signals

- runtime_benchmark.csv contains mean ± SD runtimes consistent with UniFORM 30.58 ± 0.30 s and MxNorm 114.75 ± 4.35 s (value_in_range tolerance check)
- UniFORM completes roughly 3–4× faster than MxNorm (ratio of mean runtimes ≈ 114.75/30.58)
- Both methods were run an equal number of repeated runs on the identical PRAD-CyCIF input files
- Environment setup matches the protocol: Python 3.10 conda env with UniFORM requirements.txt installed, and MxNorm installed per its GitHub guidelines

## Limitations

- The runtime claim is specific to the PRAD-CyCIF dataset and hardware; no software version numbers are stated for any of the tools used, complicating exact reproduction.
- TMA-Lunaphore data are available only upon request from the authors, so the runtime comparison cannot be trivially extended to that platform without new access.
- Timing comparisons of a Python pipeline against an R package (MxNorm, which depends on lme4/nloptr/CMake and reticulate/scikit-image) are sensitive to interpreter startup and environment overhead.

## Evidence

- [readme] MxNorm install route: [evidence span withheld — anchor status: not_found]
