---
name: autoencoder-latent-space-visualization
description: Use when per-cell shape latent vectors (from an autoencoder) and compartment-quantified
  marker intensities exist and the goal is a 2D shape embedding to check for cell
  populations or shape–marker-expression patterns.
when_to_use_negative:
- The goal is to visualize marker-expression structure itself — use the marker-intensity-only
  t-SNE (Figure 3A), not the shape-latent embedding
- The latent vectors come from the same cells used to train the autoencoder — held-out
  cells are required for a meaningful embedding
- The input is a segmentation/quantification problem (e.g., incomplete membrane extension)
  — that is MIRIAM's segmentation stage, not this downstream visualization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Produces the autoencoder cell-shape latent vectors and per-cell compartment
    marker intensity table used as t-SNE inputs
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation used to load MIRIAM latent-vector and intensity tables and
    prepare the t-SNE inputs (paper offers Matlab and Python implementations)
- name: Rtsne
  role: R package used to compute the 2D t-SNE embedding of the autoencoder latent
    shape vectors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:a73a07b0b57e9c05b45f21906a4cf71071d5bba4f0f8bcf3a4e4066dcd177a54
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# autoencoder-latent-space-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embed autoencoder-derived cell-shape latent vectors from the MIRIAM pipeline into 2D with t-SNE (Rtsne) to visualize whether morphologically similar colonic adenoma cells form populations, and overlay per-cell marker intensities to test for shape–expression correlation.

## When to use

Use when per-cell shape latent vectors (from an autoencoder) and compartment-quantified marker intensities exist and the goal is a 2D shape embedding to check for cell populations or shape–marker-expression patterns.

## When NOT to use

- The goal is to visualize marker-expression structure itself — use the marker-intensity-only t-SNE (Figure 3A), not the shape-latent embedding
- The latent vectors come from the same cells used to train the autoencoder — held-out cells are required for a meaningful embedding
- The input is a segmentation/quantification problem (e.g., incomplete membrane extension) — that is MIRIAM's segmentation stage, not this downstream visualization

## Inputs

- Autoencoder latent vectors for held-out (non-training) colon adenoma cells from the MIRIAM pipeline output table
- Per-cell marker intensity table with compartment-quantified intensities (nucleus, membrane, cytoplasm; e.g., p-ERK) from MIRIAM quantification

## Outputs

- 2D t-SNE embedding with one coordinate pair per cell (shape-latent embedding)
- 2D scatter plot colored/annotated by per-cell marker intensity (shape-based t-SNE panel, cf. Figure 3B)
- Assessment of whether shape clusters correspond to marker-expression clusters (per Supplemental Figure 5: none found)

## How to apply

Load the autoencoder latent vectors from the MIRIAM pipeline output table for held-out colon adenoma cells — critically, exclude the cells used to train the autoencoder. Load the per-cell marker intensity table (compartment-quantified intensities, e.g., p-ERK in nucleus, membrane, cytoplasm) produced by MIRIAM quantification. Run t-SNE on the latent shape vectors with the Rtsne package in R to obtain a 2D coordinate pair per cell, reproducing the shape-based t-SNE panel of Figure 3. Render the 2D scatter plot, then overlay/annotate each cell's marker intensities on the embedding and test whether shape clusters correspond to marker-expression clusters. Validation per the paper: as in Supplemental Figure 5, the shape-based t-SNE revealed no discernible cellular populations and no clear patterns in marker expression, so a correct run confirms one 2D coordinate pair per cell and, for this dataset, a negative shape–marker correlation result. This contrasts with the paper's marker-intensity-only t-SNE (Figure 3A), which did show structure.

## Related tools

- **MIRIAM** (Produces the autoencoder cell-shape latent vectors and per-cell compartment marker intensity table used as t-SNE inputs) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation used to load MIRIAM latent-vector and intensity tables and prepare the t-SNE inputs (paper offers Matlab and Python implementations))
- **Rtsne** (R package used to compute the 2D t-SNE embedding of the autoencoder latent shape vectors)

## Examples

```
Rscript -e 'library(Rtsne); lat <- read.csv("latent_held_out.csv"); emb <- Rtsne(as.matrix(lat), dims = 2)$Y; write.csv(emb, "tsne_shape_2d.csv", row.names = FALSE)'
```

## Evaluation signals

- Output table contains exactly one 2D coordinate pair per held-out cell (no training cells included)
- Shape-based t-SNE scatter plot shows clustering of morphologically similar cells, reproducing the Figure 3B panel
- Overlay of marker intensities (e.g., p-ERK) reproduces the Supplemental Figure 5 result: no discernible cellular populations or clear marker-expression patterns
- Held-out filtering is verified: cells used for autoencoder training are excluded from the embedding input
- Segmentation/quantification inputs trace to MIRIAM output tables (not another segmentation method such as Mesmer or Voronoi)

## Limitations

- On the colonic adenoma dataset, the t-SNE shape analysis did not reveal discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5) — a negative result, not a population-discovery method
- Training details/weights for the autoencoder cell-shape model are not provided in the article section, limiting reproducibility of the latent vectors
- Embedding quality depends on MIRIAM's segmentation and autoencoder shape characterization; no version numbers are stated for the Matlab or Python implementations

## Evidence

- [results] When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis (Fig. 3B): "When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis"
- [results] However, our t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5).: "our t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5)"
- [intro] Finally, cell shapes are characterized using an autoencoder neural network.: "Finally, cell shapes are characterized using an autoencoder neural network."
- [intro] each marker is then quantified by image intensity over the entire cell as well as in the nucleus, membrane, and cytoplasm: "each marker is then quantified by image intensity over the entire cell as well as in the nucleus, membrane, and cytoplasm"
