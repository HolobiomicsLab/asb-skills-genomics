---
name: multiplexed-imaging-single-cell-quantification
description: Use when you have per-cell autoencoder latent shape vectors and compartment-quantified
  marker intensities from multiplexed imaging, and want to test whether shape-space
  structure predicts molecular phenotypes.
when_to_use_negative:
- When the goal is marker-expression-based clustering itself — use t-SNE directly
  on marker intensities (Figure 3A), not on shape latent vectors
- When latent vectors include cells used to train the autoencoder — the embedding
  would reflect training data and not generalize to held-out cells
- When the data are not single-cell outputs from a multiplexed imaging segmentation/quantification
  pipeline (e.g., already-aggregated bulk features)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Produces the segmented-cell latent shape vectors and compartment-quantified
    marker intensity tables that are the inputs to the t-SNE analysis
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation environment in which the t-SNE embedding and marker-overlay
    analysis are run
- name: Rtsne
  role: R package used to compute the 2D t-SNE embedding from autoencoder latent vectors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:770e0c6f934975b778e8ec9aaebf7e5fa1f6088b0f08e3530acb8b664a40b62b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  repo_url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Python
  - Rtsne
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multiplexed-imaging-single-cell-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies t-SNE (Rtsne) to autoencoder-derived cell shape latent vectors from the MIRIAM pipeline on held-out colonic adenoma cells, then overlays per-cell marker intensities (e.g., p-ERK) to test whether shape-space clusters correspond to marker-expression patterns. Reproduces the shape-based t-SNE panel of Figure 3 and the negative result of Supplemental Figure 5.

## When to use

Use when you have per-cell autoencoder latent shape vectors and compartment-quantified marker intensities from multiplexed imaging, and want to test whether shape-space structure predicts molecular phenotypes.

## When NOT to use

- When the goal is marker-expression-based clustering itself — use t-SNE directly on marker intensities (Figure 3A), not on shape latent vectors
- When latent vectors include cells used to train the autoencoder — the embedding would reflect training data and not generalize to held-out cells
- When the data are not single-cell outputs from a multiplexed imaging segmentation/quantification pipeline (e.g., already-aggregated bulk features)

## Inputs

- MIRIAM pipeline output table of autoencoder latent shape vectors for held-out colon adenoma cells
- MIRIAM per-cell marker intensity table (compartment-quantified intensities, e.g., p-ERK in nucleus/membrane/cytoplasm)

## Outputs

- 2D t-SNE embedding with one coordinate pair per cell (shape-latent space)
- Shape-based t-SNE scatter plot (reproduces Figure 3B shape panel)
- Marker-intensity overlays on the embedding (tests Supplemental Figure 5 expectation)
- Assessment of whether shape clusters correspond to marker-expression clusters

## How to apply

Load the autoencoder latent vectors for held-out colon adenoma cells (those excluded from autoencoder training) from the MIRIAM pipeline output table, and separately load the per-cell marker intensity table quantified by MIRIAM over nucleus, membrane, and cytoplasm compartments. Run t-SNE on the latent shape vectors using the Rtsne package to obtain a 2D embedding with one coordinate pair per cell. Render the 2D shape-embedding scatter plot to reproduce the Figure 3B shape-based panel showing grouping of morphologically similar cells. Overlay or annotate each cell's marker intensities (e.g., p-ERK) onto the embedding and test whether shape clusters align with marker-expression clusters. Critically, restrict the t-SNE input to latent vectors from cells excluded from autoencoder training to avoid trivially good embeddings from overfitted shapes.

## Related tools

- **MIRIAM** (Produces the segmented-cell latent shape vectors and compartment-quantified marker intensity tables that are the inputs to the t-SNE analysis) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation environment in which the t-SNE embedding and marker-overlay analysis are run)
- **Rtsne** (R package used to compute the 2D t-SNE embedding from autoencoder latent vectors)

## Examples

```
library(Rtsne); tsne_out <- Rtsne(latent_vectors_heldout, dims=2); plot(tsne_out$Y, col=cut(marker_pERK, 10))
```

## Evaluation signals

- Output file contains exactly one 2D coordinate pair per held-out cell (row-for-row match with the latent-vector table)
- Shape-based t-SNE scatter visually groups cells with similar geometries, as in the Figure 3B shape panel
- Reproduces the reported negative result: no discernible cellular populations and no clear marker-expression patterns (Supplemental Figure 5)
- Latent vectors used for t-SNE exclude cells that were used to train the autoencoder
- Marker intensities are correctly joined per-cell to the embedding coordinates (cell IDs match across MIRIAM output tables)

## Limitations

- The t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression — a negative result should be expected on colonic adenoma data (Supplemental Figure 5)
- Only latent vectors from cells excluded from autoencoder training are valid inputs; including training cells would inflate apparent structure
- t-SNE embeddings are stochastic; random seeds and perplexity settings affect the 2D layout and are not specified in the article

## Evidence
