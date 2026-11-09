---
name: batch-effect-correction-benchmarking
description: Use when comparing batch-effect correction methods for multiplex tissue
  imaging (MTI) single-cell intensity data across samples/batches, especially with
  right-skewed, heterogeneous, or multimodal (bimodal) marker distributions.
when_to_use_negative:
- Input is pixel-level OME-TIFF image data rather than feature-level single-cell tables
  (the pixel-level tutorial and example dataset were not yet available).
- Data are mIHC (chromogenic) samples with non-linear signal amplification — linear
  normalization factors are unsuitable and a warping function is not implemented.
- Markers with highly heterogeneous/irregular distributions (e.g. CD45 in PRAD-CyCIF)
  that violate the automatic workflow's assumptions without manual guided fine-tuning.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3391
tools:
- name: UniFORM
  role: Primary non-parametric Python pipeline for feature-level MTI normalization
    via automatic rigid-landmark functional registration
  repo: https://github.com/kunlunW/UniFORM
- name: MxNorm
  role: Benchmark normalization method in R using the registration option and mean
    division
  repo: https://github.com/ColemanRHarris/mxnorm
- name: pyComBat
  role: Benchmark ComBat batch-correction method
  repo: https://epigenelabs.github.io/pyComBat/
- name: scikit-fda
  role: Converts histograms to functional data curves and performs automatic rigid-landmark
    registration
- name: scipy
  role: FFT-based maximum cross-correlation for aligning marker distributions to a
    reference
- name: anndata
  role: Loads .h5ad single-cell feature data into the pipeline
- name: plotly
  role: Generates before/after intensity distribution and downstream-analysis plots
  repo: https://github.com/plotly/plotly.py
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:de3b76871818066b2756e0a48e2b548b47b9a766cfc1afa7c163825da34aa3e2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - MxNorm
  - Python
  - UniFORM
  - anndata
  - numpy
  - pandas
  - plotly
  - pyComBat
  - scikit-fda
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# batch-effect-correction-benchmarking

## Summary

Benchmark normalization methods (UniFORM, MxNorm registration, Z score, ComBat, mean division) for multiplex tissue imaging batch correction on PRAD-CyCIF, CRC-ORION, and TMA-Lunaphore datasets, using distribution alignment, kBET, silhouette score, positive-population preservation, UMAP, and Leiden clustering as evaluation metrics.

## When to use

Use when comparing batch-effect correction methods for multiplex tissue imaging (MTI) single-cell intensity data across samples/batches, especially with right-skewed, heterogeneous, or multimodal (bimodal) marker distributions.

## When NOT to use

- Input is pixel-level OME-TIFF image data rather than feature-level single-cell tables (the pixel-level tutorial and example dataset were not yet available).
- Data are mIHC (chromogenic) samples with non-linear signal amplification — linear normalization factors are unsuitable and a warping function is not implemented.
- Markers with highly heterogeneous/irregular distributions (e.g. CD45 in PRAD-CyCIF) that violate the automatic workflow's assumptions without manual guided fine-tuning.

## Inputs

- PRAD-CyCIF single-cell feature data (.h5ad or .pkl from Zenodo record 16383766; 20 samples across 7 batches)
- CRC-ORION single-cell feature data from HTAN (participant HTA7_926; 6 samples, single batch)
- TMA-Lunaphore feature data (available from authors upon request; 3 batches on Lunaphore COMET)
- Per-marker log-transformed cell mean intensities in feature tables

## Outputs

- Normalized feature matrices (e.g. normalized_feature_data.pkl) per method
- Before/after intensity distribution plots per marker per method (Figure 2A-style layout)
- kBET and silhouette scores per method
- UMAP visualizations and Leiden clustering results per method

## How to apply

Load feature-level single-cell data (AnnData .h5ad or .pkl, e.g. PRAD-CyCIF from Zenodo record 16383766 and CRC-ORION from HTAN participant HTA7_926) into a Python 3.10 environment. For each marker, compute histograms of log-transformed cell intensities over a shared global range, convert them to functional data curves with scikit-fda, and align them to a reference via FFT-based maximum cross-correlation (UniFORM's automatic rigid-landmark registration), then apply the estimated shift/scale factors to raw intensities. Run the benchmarks in parallel: MxNorm with the registration option (R package via ColemanRHarris/mxnorm), Z score, pyComBat, and mean division. Evaluate each method on signal alignment (peak alignment of negative populations), positive-population change, kBET, silhouette score, and coherence of UMAP/Leiden clustering. Judge success by whether the method aligns negative population peaks while preserving overall distribution shape and positive-population proportions — UniFORM alone succeeds on bimodal ECAD (PRAD-CyCIF), while MxNorm, Z score, ComBat, and mean division fail in peak alignment and shape preservation.

## Related tools

- **UniFORM** (Primary non-parametric Python pipeline for feature-level MTI normalization via automatic rigid-landmark functional registration) — https://github.com/kunlunW/UniFORM
- **MxNorm** (Benchmark normalization method in R using the registration option and mean division) — https://github.com/ColemanRHarris/mxnorm
- **pyComBat** (Benchmark ComBat batch-correction method) — https://epigenelabs.github.io/pyComBat/
- **scikit-fda** (Converts histograms to functional data curves and performs automatic rigid-landmark registration)
- **scipy** (FFT-based maximum cross-correlation for aligning marker distributions to a reference)
- **anndata** (Loads .h5ad single-cell feature data into the pipeline)
- **plotly** (Generates before/after intensity distribution and downstream-analysis plots) — https://github.com/plotly/plotly.py

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git && python run_uniform.py --input prad_cycif.h5ad --output normalized_feature_data.pkl
```

## Evaluation signals

- Bimodal ECAD (PRAD-CyCIF): negative-population peaks align across batches AND overall distribution shape is preserved (UniFORM only; MxNorm, Z score, ComBat, mean division fail both criteria).
- Unimodal PD-L1 (CRC-ORION): distributions align after normalization without shape distortion.
- Mutually exclusive markers remain mutually exclusive after normalization.
- kBET scores improve (enhanced batch mixing) and silhouette scores improve versus raw and benchmark-method outputs.
- UMAP/Leiden clustering is more coherent (batch mixing without loss of biological clustering).

## Limitations

- TMA-Lunaphore dataset is not publicly available — only from authors upon request — limiting full reproduction.
- Right-skewed, heterogeneous expression patterns typical of MTI data violate the fundamental assumptions of Z score and ComBat.
- No software version numbers are stated for any of the tools used; any additional information required for reanalysis is available only from the lead contact upon request.
- Effectiveness depends on the quality of the MTI dataset and its cell/nuclei segmentation masks; compromised signals cannot be recovered.
- UniFORM uses linear normalization factors, which are unsuitable for mIHC due to non-linear chromogenic signal amplification; a warping function for non-linear mIHC normalization is not implemented.

## Evidence

- [abstract] Benchmarked on three MTI platforms, UniFORM consistently outperforms existing methods in mitigating batch effects while maintaining biological signal fidelity.: "Benchmarked on three MTI platforms, UniFORM consistently outperforms existing methods in mitigating batch effects while maintaining biological signal fidelity."
- [intro] The process begins with histogram calculation, followed by signal alignment using functional data registration, and concludes with normalization.: "The process begins with histogram calculation, followed by signal alignment using functional data registration, and concludes with normalization."
- [results] These distributions are then converted into functional data curves, which are aligned to an appropriate reference distribution using a maximum cross-correlation algorithm.: "These distributions are then converted into functional data curves, which are aligned to an appropriate reference distribution using a maximum cross-correlation algorithm"
- [intro] We evaluated each method's performance by assessing its signal alignment, k-nearest neighbor batch effect test (kBET), silhouette score, positive population change.: "We evaluated each method’s performance by assessing its signal alignment, k-nearest neighbor batch effect test (kBET), silhouette score, positive population change"
- [readme] UniFORM normalizes both feature- and pixel-level multiplex-imaging data.: "**UniFORM** normalizes both feature‑ and pixel‑level multiplex‐imaging data."
- [readme] Preserving overall distribution shape (preserve biology) is one of the criteria by which UniFORM outperforms existing methods.: "Preserving overall distribution shape (preserve biology)"
