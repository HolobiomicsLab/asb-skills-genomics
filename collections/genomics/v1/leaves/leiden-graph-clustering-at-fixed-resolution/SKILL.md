---
name: leiden-graph-clustering-at-fixed-resolution
description: Use when you need to compare cluster structure and batch/sample balance
  of single-cell MTI feature data (e.g. .h5ad AnnData) before and after a normalization
  step such as UniFORM.
when_to_use_negative:
- Input is pixel-level imaging data (OME-TIFF) rather than a per-cell feature table
  — UniFORM pixel-level normalization operates on segmentation-mask pixels and requires
  a different downstream workflow
- The goal is normalization itself rather than downstream evaluation — run the UniFORM
  pipeline (histogram calculation, functional data registration, normalization) before
  clustering
- Marker distributions are highly heterogeneous/irregular (e.g. CD45 in PRAD-CyCIF)
  and UniFORM's automatic workflow flagged them via its KL-divergence metric — normalization
  factors should be manually fine-tuned before clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniFORM
  role: Provides the normalized (and raw) feature-level .h5ad inputs that the clustering
    comparison evaluates
  repo: https://github.com/kunlunW/UniFORM
- name: Python
  role: Language for the pegasuspy-based neighbors/Leiden/UMAP pipeline
- name: plotly
  role: Interactive rendering of the before/after UMAP and Leiden cluster comparison
    figure
  repo: https://github.com/plotly/plotly.py
- name: mxnorm
  role: Baseline R package whose UMAP reduction (run_reduce_umap) represents competing
    normalization workflows against which Leiden-based comparison is contextualized
  repo: https://github.com/ColemanRHarris/mxnorm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:61b9a1ff49f1e40c4a743dfd4c8443035d73ac7479336772de628d95d3feda2b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - Python
  - UniFORM
  - matplotlib
  - mxnorm
  - numpy
  - pandas
  - plotly
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# Leiden graph clustering at fixed resolution

## Summary

Builds a k-nearest-neighbor graph (k=50) on z-scored marker-intensity features of multiplex tissue imaging (MTI) single-cell data and runs Leiden community detection at resolution 0.3 with pegasuspy, then UMAP embedding, to assess how sample/batch composition of clusters changes after UniFORM normalization of datasets such as PRAD-CyCIF.

## When to use

Use when you need to compare cluster structure and batch/sample balance of single-cell MTI feature data (e.g. .h5ad AnnData) before and after a normalization step such as UniFORM.

## When NOT to use

- Input is pixel-level imaging data (OME-TIFF) rather than a per-cell feature table — UniFORM pixel-level normalization operates on segmentation-mask pixels and requires a different downstream workflow
- The goal is normalization itself rather than downstream evaluation — run the UniFORM pipeline (histogram calculation, functional data registration, normalization) before clustering
- Marker distributions are highly heterogeneous/irregular (e.g. CD45 in PRAD-CyCIF) and UniFORM's automatic workflow flagged them via its KL-divergence metric — normalization factors should be manually fine-tuned before clustering

## Inputs

- Feature-level single-cell AnnData (.h5ad) object with marker-intensity matrix and sample/batch metadata (raw, pre-normalization)
- UniFORM-normalized feature-level AnnData (.h5ad) object for the same cells/markers (PRAD-CyCIF from https://zenodo.org/records/16383766)

## Outputs

- Leiden cluster assignments per cell for raw and normalized data at resolution 0.3
- UMAP embeddings for both datasets
- Per-cluster batch/sample composition tables and batch-mixing (kBET) statistics
- Balance metrics written to leiden_balance_metrics.csv and before/after UMAP + Leiden comparison figure (matplotlib/plotly)

## How to apply

Load the raw (pre-normalization) and UniFORM-normalized feature-level AnnData (.h5ad) objects, e.g. the PRAD-CyCIF dataset from Zenodo record 16383766. Z-score scale the marker-intensity features in each object so markers contribute comparably to the graph. Build a k-nearest-neighbor graph with 50 neighbors on the scaled features using the pegasuspy neighbors step, then run Leiden community detection at resolution 0.3 on this graph for both the raw and normalized datasets. Compute UMAP embeddings with pegasuspy for visualization, tabulate per-cluster batch/sample composition before and after normalization, and compute batch-mixing statistics (kBET) plus silhouette scores. Rationale: the article uses 'more coherent UMAP and Leiden clustering' and enhanced kBET/silhouette scores as evidence that normalization reduces batch-driven sample separation while preserving biological signal.

## Related tools

- **UniFORM** (Provides the normalized (and raw) feature-level .h5ad inputs that the clustering comparison evaluates) — https://github.com/kunlunW/UniFORM
- **Python** (Language for the pegasuspy-based neighbors/Leiden/UMAP pipeline)
- **plotly** (Interactive rendering of the before/after UMAP and Leiden cluster comparison figure) — https://github.com/plotly/plotly.py
- **mxnorm** (Baseline R package whose UMAP reduction (run_reduce_umap) represents competing normalization workflows against which Leiden-based comparison is contextualized) — https://github.com/ColemanRHarris/mxnorm

## Examples

```
import pegasus as pg; adata = anndata.read_h5ad('prad_cycif_normalized.h5ad'); pg.neighbors(adata, k=50); pg.leiden(adata, resolution=0.3); pg.umap(adata)
```

## Evaluation signals

- Leiden clustering is run with the paper's exact parameters: 50-neighbor kNN graph and resolution 0.3 on both raw and normalized data — differing parameters invalidates the comparison
- Post-normalization, per-cluster sample composition should be more balanced, with each sample (20 PRAD-CyCIF samples across 7 batches) contributing more uniformly to each Leiden cluster
- kBET batch-mixing statistic should improve (better mixing) after normalization, alongside improved silhouette scores
- UMAP embeddings of the normalized data should show reduced sample/batch-driven separation compared with the raw data while preserving distribution shapes and positive-population proportions
- Metrics table leiden_balance_metrics.csv should contain per-cluster composition entries for both conditions, enabling a before/after diff

## Limitations

- Effectiveness depends on dataset quality and cell/nuclei segmentation-mask quality; compromised signals cannot be recovered by normalization or clustering downstream
- No software version numbers are stated for the analysis tools, so exact reproducibility of the pegasuspy clustering environment is uncertain
- The PRAD-CyCIF feature data are on Zenodo, but the TMA-Lunaphore dataset is only available from the authors upon request, limiting cross-dataset reuse of this clustering comparison

## Evidence

- [other] Run Leiden community detection at resolution 0.3 on the 50-neighbor graph for both the raw and normalized datasets (pegasuspy).: [evidence span withheld — anchor status: not_found]
- [other] Build a k-nearest-neighbor graph with 50 neighbors on the scaled features (pegasuspy neighbors step).: [evidence span withheld — anchor status: not_found]
