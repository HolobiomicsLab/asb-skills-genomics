---
name: normalization-method-benchmarking
description: Use when comparing multiple batch-effect normalization methods on MTI
  feature data (e.g., PRAD-CyCIF .pkl/.h5ad across 7 batches) using kBET, chi-squared,
  and silhouette metrics to rank methods.
when_to_use_negative:
- 'Data are mIHC (chromogenic) samples: linear normalization factors are unsuitable
  because enzyme-catalyzed colorimetric amplification produces non-linear signal intensities'
- Markers with highly heterogeneous or irregular distributions (e.g., CD45 in PRAD-CyCIF)
  that violate the automatic workflow's assumptions and require manual guided fine-tuning
  flagged by KL divergence
- Pixel-level (OME-TIFF) data lacking nucleus/cell segmentation masks, since pixel-level
  normalization is restricted to pixels within segmentation masks and the example
  dataset/tutorial were not yet available
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: UniFORM
  role: Candidate method under test; automatic rigid-landmark functional registration
    pipeline producing normalized feature data
  repo: https://github.com/kunlunW/UniFORM
- name: MxNorm
  role: Baseline normalization method run with the registration option following the
    repository guidelines
  repo: https://github.com/ColemanRHarris/mxnorm
- name: pyComBat
  role: Baseline ComBat batch-correction method applied to the same feature data
  repo: https://epigenelabs.github.io/pyComBat/
- name: scikit-fda
  role: Functional data registration library used by UniFORM to align marker intensity
    distribution curves via maximum cross-correlation
- name: scikit-learn
  role: k-nearest-neighbor computations underlying kBET acceptance-rate and silhouette-score
    evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:9645b087216b46c7da4275dd4d3b4a8b69648b12908cb68968afeaf060bc9d13
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - MxNorm
  - Python
  - UniFORM
  - pyComBat
  - scikit-fda
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# normalization-method-benchmarking

## Summary

Benchmarks feature-level batch-effect normalization methods (UniFORM, MxNorm registration, Z score, ComBat, mean division) against raw data on multiplex tissue imaging (MTI) single-cell datasets, quantifying correction quality with kBET acceptance rates, chi-squared statistics, p-values, and silhouette scores.

## When to use

Use when comparing multiple batch-effect normalization methods on MTI feature data (e.g., PRAD-CyCIF .pkl/.h5ad across 7 batches) using kBET, chi-squared, and silhouette metrics to rank methods.

## When NOT to use

- Data are mIHC (chromogenic) samples: linear normalization factors are unsuitable because enzyme-catalyzed colorimetric amplification produces non-linear signal intensities
- Markers with highly heterogeneous or irregular distributions (e.g., CD45 in PRAD-CyCIF) that violate the automatic workflow's assumptions and require manual guided fine-tuning flagged by KL divergence
- Pixel-level (OME-TIFF) data lacking nucleus/cell segmentation masks, since pixel-level normalization is restricted to pixels within segmentation masks and the example dataset/tutorial were not yet available

## Inputs

- PRAD-CyCIF feature-level single-cell data as *.pkl pickle or *.h5ad AnnData (Zenodo record 16383766)
- Batch labels for the 7 CyCIF batches (20 prostate cancer whole-slide samples)
- Baseline normalization implementations: MxNorm (registration option), pyComBat, Z score, mean division

## Outputs

- benchmark_metrics.csv with average kBET acceptance score, p-value, chi-squared statistic, and silhouette score for raw, MxNorm registration, Z score, ComBat, mean division, and UniFORM

## How to apply

Load the feature-level single-cell data (e.g., PRAD-CyCIF *.pkl or *.h5ad AnnData from Zenodo record 16383766) into pandas/AnnData, keeping the 7 CyCIF batch labels. Run each normalization on identical inputs: UniFORM's automatic pipeline (histograms of log-transformed cell intensities per marker, aligned via automatic rigid-landmark functional data registration with scikit-fda using maximum cross-correlation, then applying the estimated normalization factors), MxNorm with the registration option per the ColemanRHarris/mxnorm guidelines, Z score scaling, ComBat via pyComBat, and mean division. For raw and every normalized version, compute average kBET acceptance rates and p-values plus average chi-squared statistics across batches using scikit-learn-based neighbor tests, and compute the average silhouette score for clustering consistency. Compile all metric families for all six conditions into benchmark_metrics.csv. Judge success by the acceptance-rate ranking: on PRAD-CyCIF, UniFORM should reach ~0.60843, mean division ~0.54240, MxNorm ~0.38788, Z score ~0.28570, ComBat ~0.27266, and raw ~0.00098.

## Related tools

- **UniFORM** (Candidate method under test; automatic rigid-landmark functional registration pipeline producing normalized feature data) — https://github.com/kunlunW/UniFORM
- **MxNorm** (Baseline normalization method run with the registration option following the repository guidelines) — https://github.com/ColemanRHarris/mxnorm
- **pyComBat** (Baseline ComBat batch-correction method applied to the same feature data) — https://epigenelabs.github.io/pyComBat/
- **scikit-fda** (Functional data registration library used by UniFORM to align marker intensity distribution curves via maximum cross-correlation)
- **scikit-learn** (k-nearest-neighbor computations underlying kBET acceptance-rate and silhouette-score evaluation)

## Examples

```
mx_norm = mx_normalize(mx_data = mx_dataset, transform = "log10_mean_divide", method="None")
```

## Evaluation signals

- Raw PRAD-CyCIF data must yield an average kBET acceptance score near 0.00098; if raw acceptance is high, batch labels or neighbor tests are misconfigured
- UniFORM should achieve the highest average kBET acceptance (~0.60843), exceeding mean division (0.54240), MxNorm registration (0.38788), Z score (0.28570), and ComBat (0.27266)
- Each of the four metric families (kBET acceptance, kBET p-value, average chi-squared, silhouette) must be populated for all six conditions in benchmark_metrics.csv
- Normalized data should preserve distribution shape and positive-population proportions rather than collapsing them, consistent with UniFORM's negative-population alignment rationale
- Silhouette scores should indicate maintained clustering consistency after correction, alongside more coherent UMAP/Leiden structure

## Limitations

- UniFORM's effectiveness depends on MTI dataset quality and cell/nuclei segmentation-mask quality; compromised signals cannot be recovered
- Highly heterogeneous or irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the automatic workflow's assumptions and require manual guided fine-tuning; such markers are flagged via a Kullback-Leibler divergence-based metric
- The TMA-Lunaphore dataset used for benchmarking is only available from the authors upon request, limiting full reproducibility across all three platforms
- No software version numbers are stated for any tools, so exact environment reproduction requires requesting additional information from the lead contact

## Evidence
