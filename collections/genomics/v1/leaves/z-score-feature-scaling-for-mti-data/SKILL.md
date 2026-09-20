---
name: z-score-feature-scaling-for-mti-data
description: Use when feature-level MTI single-cell data (e.g., PRAD-CyCIF .h5ad from
  Zenodo record 16383766) must be scaled to comparable per-marker ranges prior to
  kNN graph building (e.g., 50 neighbors), Leiden clustering, or UMAP visualization.
when_to_use_negative:
- 'The input has uncorrected batch effects and the goal is batch correction: Z-score
  scaling does not mitigate batch-driven sample separation (the paper shows it is
  outperformed by UniFORM on kBET, silhouette, and clustering metrics)'
- 'The data are pixel-level MTI (OME-TIFF) images or mIHC chromogenic data: UniFORM''s
  feature-level pipeline and this scaling workflow target feature-level (per-cell)
  data, and mIHC non-linear signal amplification falls outside scope'
- The analysis targets marker intensity distributions themselves (e.g., distribution
  alignment checks), where scaling would distort the raw intensity scales being compared
edam_operation: http://edamontology.org/operation_3432
tools:
- name: UniFORM
  role: Upstream normalization of feature-level MTI data whose output (.h5ad) is Z-score
    scaled before clustering; Z-score alone serves as a baseline comparison method
    in the paper
  repo: https://github.com/kunlunW/UniFORM
- name: anndata
  role: Loading and holding the raw and UniFORM-normalized PRAD-CyCIF .h5ad feature
    objects during scaling
- name: scikit-learn
  role: Provides the standardization/scaling utilities applied to marker intensity
    features
- name: pegasus
  role: 'Downstream consumers of the scaled features: kNN graph (50 neighbors), Leiden
    clustering (resolution 0.3), and UMAP computation'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:68521d8c30c37553646a91144034f5ff04f24e918147c5f7d16f4ef9f72a1e2c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - Python
  - UniFORM
  - anndata
  - matplotlib
  - numpy
  - pandas
  - pegasus
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# z-score-feature-scaling-for-mti-data

## Summary

Z-score scaling of marker intensity features in multiplex tissue imaging (MTI) AnnData objects to place all markers on a comparable scale before kNN graph construction, Leiden clustering, and UMAP. In the UniFORM paper this step is applied to both raw and UniFORM-normalized PRAD-CyCIF feature data to make before/after clustering comparisons fair.

## When to use

Use when feature-level MTI single-cell data (e.g., PRAD-CyCIF .h5ad from Zenodo record 16383766) must be scaled to comparable per-marker ranges prior to kNN graph building (e.g., 50 neighbors), Leiden clustering, or UMAP visualization.

## When NOT to use

- The input has uncorrected batch effects and the goal is batch correction: Z-score scaling does not mitigate batch-driven sample separation (the paper shows it is outperformed by UniFORM on kBET, silhouette, and clustering metrics)
- The data are pixel-level MTI (OME-TIFF) images or mIHC chromogenic data: UniFORM's feature-level pipeline and this scaling workflow target feature-level (per-cell) data, and mIHC non-linear signal amplification falls outside scope
- The analysis targets marker intensity distributions themselves (e.g., distribution alignment checks), where scaling would distort the raw intensity scales being compared

## Inputs

- Raw (pre-normalization) PRAD-CyCIF feature-level AnnData object (.h5ad) from Zenodo record 16383766
- UniFORM-normalized PRAD-CyCIF feature-level AnnData object (.h5ad)
- Per-cell marker intensity matrix (cells x markers) with sample/batch metadata

## Outputs

- Z-scored feature matrix (per-marker mean ~0, standard deviation ~1) stored in the AnnData object
- Scaled features feeding a 50-neighbor kNN graph, Leiden clustering (resolution 0.3), and UMAP embedding
- Per-cluster sample/batch composition tables and batch-mixing statistics (kBET), e.g., leiden_balance_metrics.csv
- Before/after UMAP and Leiden cluster comparison figures (matplotlib/plotly)

## How to apply

Load the raw (pre-normalization) and UniFORM-normalized PRAD-CyCIF feature-level AnnData objects (.h5ad) with anndata, then Z-score scale the marker intensity features so each marker contributes equally to distance computations — this matters because MTI markers are on incommensurate intensity scales and raw distributions are right-skewed and heterogeneous. After scaling, build a k-nearest-neighbor graph with 50 neighbors (pegasus neighbors step), run Leiden community detection at resolution 0.3, and compute UMAP embeddings for both the raw and normalized datasets. Tabulate per-cluster sample/batch composition before and after normalization and compute kBET batch-mixing statistics to verify that cluster representation across the 20 PRAD-CyCIF samples and 7 batches becomes more balanced post-normalization. Note the rationale boundary: Z score scaling alone is a baseline conditioning step, not a batch-correction method — the paper shows that Z score (like ComBat) fails on right-skewed MTI distributions, which is why it is applied downstream of UniFORM's negative-population-aligned normalization rather than as a replacement for it.

## Related tools

- **UniFORM** (Upstream normalization of feature-level MTI data whose output (.h5ad) is Z-score scaled before clustering; Z-score alone serves as a baseline comparison method in the paper) — https://github.com/kunlunW/UniFORM
- **anndata** (Loading and holding the raw and UniFORM-normalized PRAD-CyCIF .h5ad feature objects during scaling)
- **scikit-learn** (Provides the standardization/scaling utilities applied to marker intensity features)
- **pegasus** (Downstream consumers of the scaled features: kNN graph (50 neighbors), Leiden clustering (resolution 0.3), and UMAP computation)

## Examples

```
from sklearn.preprocessing import StandardScaler; adata.X = StandardScaler().fit_transform(adata.X)
```

## Evaluation signals

- After scaling, each marker column has mean approximately 0 and standard deviation approximately 1 (verify with adata.X.mean(axis=0) and .std(axis=0))
- The 50-neighbor kNN graph and Leiden clustering at resolution 0.3 run without errors on both raw and normalized AnnData objects
- Post-normalization, per-cluster sample composition across the 20 PRAD-CyCIF samples becomes more balanced than pre-normalization, confirmed by improved kBET batch-mixing statistics
- Silhouette scores improve after UniFORM normalization + scaling relative to raw + scaling
- UMAP embeddings show reduced sample/batch-driven separation in the normalized data compared to the raw data

## Limitations

- Z-score scaling does not itself correct batch effects and is unsuitable as a normalization method for right-skewed, heterogeneous MTI distributions — the paper benchmarks it (alongside ComBat and mean division) as an outperformed baseline
- Some markers with highly heterogeneous/irregular distributions (e.g., CD45 in PRAD-CyCIF) violate assumptions of the automatic normalization workflow and require manual fine-tuning; scaling such markers inherits their artifacts
- Effectiveness depends on MTI dataset quality and cell/nuclei segmentation mask quality; compromised signals cannot be recovered, and no software version numbers are stated for the tools

## Evidence
