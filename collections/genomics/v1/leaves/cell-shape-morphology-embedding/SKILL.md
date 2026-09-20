---
name: cell-shape-morphology-embedding
description: Use when you have per-cell autoencoder latent shape vectors from MIRIAM
  segmentation plus a compartment-quantified marker intensity table, and want a 2D
  morphology embedding to check for shape-driven cellular populations.
when_to_use_negative:
- The question is whether marker intensities alone separate populations — use the
  intensity-based t-SNE of Figure 3A instead of shape latent vectors.
- Your cells were part of the autoencoder training set — the article restricts this
  analysis to held-out cells, since training-cell latents would trivially cluster.
- Your images are not single-cell segmented/quantified by MIRIAM (no per-cell latent
  vectors or compartment intensities exist yet) — run the MIRIAM segmentation pipeline
  first.
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: MIRIAM
  role: Produces the autoencoder latent cell-shape vectors and compartment-quantified
    marker intensity table that feed the t-SNE embedding
  repo: github.com/Coffey-Lab/MIRIAM
- name: Rtsne
  role: R package used to compute the 2D t-SNE embedding from the shape latent vectors
- name: Python
  role: Alternative MIRIAM implementation producing equivalent latent vectors; results
    are largely similar between Matlab and Python implementations
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:9dbe850a713e7613d4a9e5391f7b0444a008de38f54d073243f91d5d4f00d6ed
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
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cell-shape-morphology-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embeds autoencoder-derived cell-shape latent vectors from the MIRIAM pipeline into a 2D t-SNE map to test whether morphology alone separates cellular populations and correlates with per-cell marker intensities. In the source study (colorectal adenoma multiplexed immunofluorescence), shape-only t-SNE grouped geometrically similar cells but revealed no marker-expression patterns.

## When to use

Use when you have per-cell autoencoder latent shape vectors from MIRIAM segmentation plus a compartment-quantified marker intensity table, and want a 2D morphology embedding to check for shape-driven cellular populations.

## When NOT to use

- The question is whether marker intensities alone separate populations — use the intensity-based t-SNE of Figure 3A instead of shape latent vectors.
- Your cells were part of the autoencoder training set — the article restricts this analysis to held-out cells, since training-cell latents would trivially cluster.
- Your images are not single-cell segmented/quantified by MIRIAM (no per-cell latent vectors or compartment intensities exist yet) — run the MIRIAM segmentation pipeline first.

## Inputs

- MIRIAM pipeline output table of autoencoder latent shape vectors for held-out cells (cells excluded from autoencoder training)
- Per-cell marker intensity table from MIRIAM quantification (intensities over entire cell and in nucleus, membrane, cytoplasm compartments, e.g., p-ERK)
- Multiplexed immunofluorescence image-derived single-cell segmentation masks from MIRIAM (e.g., colorectal adenoma data)

## Outputs

- 2D t-SNE embedding with one coordinate pair per cell, generated via Rtsne
- Shape-embedding scatter plot reproducing the Figure 3B morphology panel
- Marker-intensity-overlaid embedding for testing shape/marker correlation (cf. Supplemental Figure 5)

## How to apply

Run MIRIAM's segmentation-and-quantification pipeline (random forest pixel classification, membrane extension, autoencoder shape characterization) to obtain latent shape vectors for held-out cells — exclude the cells used to train the autoencoder. Load the latent vectors and the per-cell, compartment-quantified (nucleus, membrane, cytoplasm) marker intensity table (e.g., p-ERK intensities in colonic adenoma cells). Run t-SNE on the latent vectors using the Rtsne R package to obtain one 2D coordinate pair per cell, and render the shape-embedding scatter plot (analogous to Figure 3B, complementing the intensity-based t-SNE of Figure 3A). Overlay each cell's marker intensities on the embedding to test whether shape clusters correspond to marker-expression clusters. In the source study this served as a negative-control analysis: the shape t-SNE showed no discernible populations or marker patterns (Supplemental Figure 5), so validation means reproducing both the shape-clustered scatter and the null marker correlation.

## Related tools

- **MIRIAM** (Produces the autoencoder latent cell-shape vectors and compartment-quantified marker intensity table that feed the t-SNE embedding) — github.com/Coffey-Lab/MIRIAM
- **Rtsne** (R package used to compute the 2D t-SNE embedding from the shape latent vectors)
- **Python** (Alternative MIRIAM implementation producing equivalent latent vectors; results are largely similar between Matlab and Python implementations) — github.com/Coffey-Lab/MIRIAM

## Examples

```
Rscript -e 'library(Rtsne); lat <- read.csv("miriam_latent_vectors.csv"); emb <- Rtsne(lat[,1:ncol(lat)], dims=2); write.csv(emb$Y, "shape_tsne_2d.csv", row.names=FALSE)'
```

## Evaluation signals

- Output file contains exactly one 2D coordinate pair per held-out cell — no rows for autoencoder training cells.
- Scatter plot shows clustering of morphologically similar cells, reproducing the shape-based t-SNE panel (Figure 3B).
- Overlay of marker intensities (e.g., p-ERK) shows no clear pattern across the embedding, consistent with Supplemental Figure 5 as the reported negative result.
- Pipeline provenance is MIRIAM's autoencoder output; Matlab vs Python implementations of MIRIAM give largely similar results (Supplemental Figure 6), so latent vectors from either are acceptable.
- Intensity-based t-SNE (Figure 3A) and shape-based t-SNE (Figure 3B) are kept as separate analyses on distinct input matrices.

## Limitations

- In the source study, t-SNE of shape latent vectors did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5) — expect a negative/null result on colonic adenoma data.
- Only held-out (non-autoencoder-training) cells may be embedded; training-cell latents are excluded by design.
- Latent vectors encode only cell geometry, not intensity; they cannot capture marker-expression-driven structure.
- MIRIAM example data omits image stacks (too large); users must regenerate them and the autoencoder training details/weights are not provided in the source text.

## Evidence
