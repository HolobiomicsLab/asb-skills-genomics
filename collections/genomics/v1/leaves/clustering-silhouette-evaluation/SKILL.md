---
name: clustering-silhouette-evaluation
description: Use when benchmarking feature-level normalization of MTI data (e.g.,
  PRAD-CyCIF .pkl/.h5ad) and you need average silhouette scores, alongside kBET, to
  judge clustering consistency of raw vs. normalized datasets.
when_to_use_negative:
- Input is mIHC (chromogenic) data, where UniFORM's linear normalization factors are
  unsuitable due to non-linear enzyme-catalyzed signal amplification
- Input is pixel-level OME-TIFF data, for which the UniFORM pixel-level tutorial and
  example dataset are not yet available
- You only need intensity-distribution alignment, not downstream clustering consistency
  assessment
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: UniFORM
  role: Provides the normalized feature-level datasets (and baselines it is compared
    against) whose silhouette scores are evaluated
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-learn
  role: Python library used for the silhouette score and k-nearest-neighbor-based
    batch effect tests across the 7 CyCIF batches
- name: Python
  role: Environment in which the benchmark metrics (silhouette, kBET, chi-squared,
    p-value) are computed and compiled into benchmark_metrics.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:f1076cc60eb13c25c09acfe319edd69c22e533d95566be321985c7f8f36017ab
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# clustering-silhouette-evaluation

## Summary

Computes average silhouette scores on normalized multiplex tissue imaging (MTI) feature-level single-cell data to quantify whether a normalization method (e.g., UniFORM) improves clustering consistency across batches compared to raw data and baselines like MxNorm, Z score, ComBat, and mean division.

## When to use

Use when benchmarking feature-level normalization of MTI data (e.g., PRAD-CyCIF .pkl/.h5ad) and you need average silhouette scores, alongside kBET, to judge clustering consistency of raw vs. normalized datasets.

## When NOT to use

- Input is mIHC (chromogenic) data, where UniFORM's linear normalization factors are unsuitable due to non-linear enzyme-catalyzed signal amplification
- Input is pixel-level OME-TIFF data, for which the UniFORM pixel-level tutorial and example dataset are not yet available
- You only need intensity-distribution alignment, not downstream clustering consistency assessment

## Inputs

- Feature-level single-cell MTI data as pandas DataFrame (*.pkl) or AnnData (*.h5ad), e.g., PRAD-CyCIF (20 samples, 7 batches) from Zenodo record 16383766
- Per-batch or per-slide batch labels for each cell
- Marker intensity matrices for each normalization condition (raw, UniFORM, MxNorm registration, Z score, ComBat, mean division)

## Outputs

- Average silhouette score per normalization condition (raw and each method)
- Rows in benchmark_metrics.csv combining silhouette, kBET acceptance, chi-squared statistic, and p-value
- Comparison supporting claims of improved clustering consistency and batch mixing

## How to apply

Load the feature-level single-cell data (pickle or AnnData .h5ad, e.g., PRAD-CyCIF from Zenodo record 16383766), one table per normalization condition: raw, UniFORM, MxNorm (registration option), Z score, ComBat (pyComBat), and mean division. For each condition, cluster or label cells across the 7 CyCIF batches and compute the average silhouette score using scikit-learn-based methods, evaluating how coherently cells group independent of batch. Compare the silhouette scores against the kBET acceptance rates to jointly assess batch mixing and biological signal fidelity; UniFORM reported the highest average kBET acceptance (0.60843 vs. raw 0.00098) on PRAD-CyCIF. Compile silhouette scores together with kBET, chi-squared, and p-value metrics into benchmark_metrics.csv so all four metric families can be compared per method.

## Related tools

- **UniFORM** (Provides the normalized feature-level datasets (and baselines it is compared against) whose silhouette scores are evaluated) — https://github.com/kunlunW/UniFORM
- **scikit-learn** (Python library used for the silhouette score and k-nearest-neighbor-based batch effect tests across the 7 CyCIF batches)
- **Python** (Environment in which the benchmark metrics (silhouette, kBET, chi-squared, p-value) are computed and compiled into benchmark_metrics.csv)

## Examples

```
from sklearn.metrics import silhouette_score; s = silhouette_score(X_normalized, labels)
```

## Evaluation signals

- Silhouette scores are computed identically for raw and every normalized condition, enabling a like-for-like comparison table
- A correct application should reproduce the paper's conclusion that UniFORM outperforms raw, MxNorm registration, Z score, ComBat, and mean division on clustering/batch metrics
- Silhouette results should be cross-checked against average kBET acceptance rates (e.g., UniFORM 0.60843 vs. raw 0.00098 on PRAD-CyCIF) for consistency
- All four metric families (kBET, chi-squared, p-value, silhouette) are present per method in benchmark_metrics.csv
- Metrics are averaged across the 7 CyCIF batches rather than reported per-batch only

## Limitations

- Highly heterogeneous or irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the automatic workflow assumptions and are flagged via a Kullback-Leibler divergence-based metric, requiring manual fine-tuning before trustworthy clustering metrics
- UniFORM effectiveness depends on MTI dataset and cell/nuclei segmentation mask quality; compromised signals cannot be recovered
- No software version numbers are stated for any of the tools used, complicating exact reproduction of metric values

## Evidence

- [abstract] Improved clustering/batch metrics underpin the skill: "This is evidenced by improved marker distribution alignment and positive population preservation, enhanced k-nearest neighbor batch effect test (kBET) and silhouette scores"
- [intro] Silhouette score is an explicit evaluation metric of the benchmark: "We evaluated each method’s performance by assessing its signal alignment, k-nearest neighbor batch effect test (kBET), silhouette score, positive population change"
- [readme] README lists clustering consistency improvement as a highlight: "Improving clustering consistency"
