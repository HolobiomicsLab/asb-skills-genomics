---
name: tsne-dimensionality-reduction
description: Use when you have a per-cell feature matrix from MIRIAM output (autoencoder
  shape latent vectors or compartment-quantified marker intensities) and want a 2D
  embedding to explore clustering of morphologically or phenotypically similar cells.
when_to_use_negative:
- Input is raw multiplexed image stacks rather than a per-cell feature table — segmentation
  and quantification via MIRIAM must be run first
- The autoencoder latent vectors are not available and cells being embedded were part
  of autoencoder training — the published analysis explicitly excluded those cells
- The goal is quantitative segmentation benchmarking (e.g., DSC/JSC comparison against
  Mesmer or Voronoi) rather than exploratory visualization
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Upstream pipeline that generates the per-cell autoencoder shape latent vectors
    and compartment-quantified marker intensities used as t-SNE inputs
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation of the MIRIAM pipeline producing the latent vectors and intensity
    tables fed into t-SNE
  repo: github.com/Coffey-Lab/MIRIAM
- name: Rtsne
  role: R package used to run the t-SNE embedding on the shape latent vectors or marker
    intensities
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:bd8b9c62db718dcff934142d31fff10584d80f73fe0680f0b65f3e6d83eb2e05
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Python
  - Rtsne
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tsne-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply t-Stochastic Neighbor Embedding (t-SNE) to high-dimensional per-cell feature vectors — either autoencoder-encoded cell-shape latent vectors or per-compartment marker intensities from the MIRIAM pipeline — to obtain 2D embeddings for visualizing cellular populations in multiplexed tissue imaging data.

## When to use

Use when you have a per-cell feature matrix from MIRIAM output (autoencoder shape latent vectors or compartment-quantified marker intensities) and want a 2D embedding to explore clustering of morphologically or phenotypically similar cells.

## When NOT to use

- Input is raw multiplexed image stacks rather than a per-cell feature table — segmentation and quantification via MIRIAM must be run first
- The autoencoder latent vectors are not available and cells being embedded were part of autoencoder training — the published analysis explicitly excluded those cells
- The goal is quantitative segmentation benchmarking (e.g., DSC/JSC comparison against Mesmer or Voronoi) rather than exploratory visualization

## Inputs

- Autoencoder latent vectors for cell shape from MIRIAM output table (held-out cells excluded from autoencoder training)
- Per-cell marker intensity table (compartment-quantified intensities, e.g., p-ERK) produced by MIRIAM quantification

## Outputs

- 2D t-SNE embedding with one coordinate pair per cell
- 2D scatter plot of the shape-based or marker-intensity-based embedding
- Marker-intensity-annotated embedding for testing shape/marker correspondence

## How to apply

Load the per-cell feature table produced by MIRIAM quantification: either the marker intensity values (whole cell plus nucleus, membrane, cytoplasm compartments) or the autoencoder latent vectors for cell shape, taking care to exclude latent vectors from cells used to train the autoencoder (i.e., use held-out colon adenoma cells). Run t-SNE on the feature matrix using the Rtsne package in R to embed each cell into 2 dimensions. Render the resulting scatter plot: in the MIRIAM paper, t-SNE on marker intensities alone produced distinct clusters (Figure 3A), while t-SNE on shape latent vectors grouped cells with similar geometries (Figure 3B). Overlay or annotate each cell's marker intensities (e.g., p-ERK) on the shape-based embedding to test whether shape clusters correspond to marker-expression clusters. Validate against the published result: the shape-based t-SNE should reveal no discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5).

## Related tools

- **MIRIAM** (Upstream pipeline that generates the per-cell autoencoder shape latent vectors and compartment-quantified marker intensities used as t-SNE inputs) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation of the MIRIAM pipeline producing the latent vectors and intensity tables fed into t-SNE) — github.com/Coffey-Lab/MIRIAM
- **Rtsne** (R package used to run the t-SNE embedding on the shape latent vectors or marker intensities)

## Examples

```
library(Rtsne); emb <- Rtsne(shape_latent_vectors, dims = 2)$Y
```

## Evaluation signals

- Output file contains exactly one 2D coordinate pair per input cell from the held-out set
- Marker-intensity t-SNE should show discernible clusters (as in Figure 3A), confirming the t-SNE run is functioning
- Shape-latent-vector t-SNE should group cells with similar geometries but reveal no discernible cellular populations or marker-expression patterns (Supplemental Figure 5)
- Cells used to train the autoencoder must be excluded from the shape-based t-SNE input
- Embedding is computed independently for shape latent vectors vs. marker intensities so the two panels can be compared (Figures 3A vs. 3B)

## Limitations

- In the published analysis, the t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5) — shape clustering may not map to biological phenotypes
- Latent vectors from cells used to train the autoencoder are excluded, limiting the analysis to held-out cells
- Training details/weights for the autoencoder cell-shape model are not provided in the available text, constraining reproducibility of the latent-space input

## Evidence
