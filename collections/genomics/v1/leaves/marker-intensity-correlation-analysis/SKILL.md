---
name: marker-intensity-correlation-analysis
description: Use when per-cell shape latent vectors from MIRIAM's autoencoder have
  been embedded via t-SNE and you need to determine whether the resulting cell clusters
  align with compartment-quantified marker intensities.
when_to_use_negative:
- Marker intensity tables have not yet been produced by MIRIAM quantification (no
  compartment-level intensities to overlay).
- Latent vectors from cells used to train the autoencoder are included — the analysis
  requires held-out cells only, as training cells bias the embedding.
- The goal is t-SNE on marker intensities themselves (Figure 3A-style), not correlating
  intensities against the shape embedding.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIRIAM
  role: Produces the autoencoder latent shape vectors and per-cell compartment marker
    intensity tables used as inputs to the t-SNE embedding and overlay
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation language used to run the t-SNE shape analysis and overlay marker
    intensities (MIRIAM has Matlab and Python implementations)
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:6c0fee4efda39331e76bd4593c8912e998cd24e39b8ed2b33f81dcb13d2d541d
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# marker-intensity-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Overlay per-cell marker intensities (e.g., p-ERK compartment quantification from the MIRIAM pipeline) onto a 2D t-SNE embedding of autoencoder-derived cell-shape latent vectors to test whether shape clusters correspond to marker-expression patterns in colonic adenoma cells.

## When to use

Use when per-cell shape latent vectors from MIRIAM's autoencoder have been embedded via t-SNE and you need to determine whether the resulting cell clusters align with compartment-quantified marker intensities.

## When NOT to use

- Marker intensity tables have not yet been produced by MIRIAM quantification (no compartment-level intensities to overlay).
- Latent vectors from cells used to train the autoencoder are included — the analysis requires held-out cells only, as training cells bias the embedding.
- The goal is t-SNE on marker intensities themselves (Figure 3A-style), not correlating intensities against the shape embedding.

## Inputs

- MIRIAM output table of autoencoder latent vectors for cell shape (held-out cells excluded from autoencoder training)
- Per-cell marker intensity table with compartment-quantified intensities (entire cell, nucleus, membrane, cytoplasm) from MIRIAM quantification of multiplexed immunofluorescence images of colorectal adenomas

## Outputs

- 2D t-SNE embedding with one coordinate pair per cell (shape-based, Figure 3B-style panel)
- Overlay/annotation of per-cell marker intensities on the shape embedding
- Validation output files containing one 2D coordinate pair per cell, checked against Supplemental Figure 5 expectations

## How to apply

Load the MIRIAM pipeline output table of autoencoder latent vectors for held-out colon adenoma cells (cells excluded from autoencoder training) and the per-cell compartment-quantified marker intensity table (nucleus, membrane, cytoplasm, e.g., p-ERK). Run t-SNE on the latent shape vectors with the Rtsne package to obtain one 2D coordinate pair per cell. Render the shape-embedding scatter plot reproducing the Figure 3B panel, then overlay/annotate each cell's marker intensities and test whether shape clusters correspond to marker-expression clusters. Note that in the source study this analysis produced a negative result: the t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5), so a correct run confirms this absence of pattern rather than expecting positive cluster-marker correspondence.

## Related tools

- **MIRIAM** (Produces the autoencoder latent shape vectors and per-cell compartment marker intensity tables used as inputs to the t-SNE embedding and overlay) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation language used to run the t-SNE shape analysis and overlay marker intensities (MIRIAM has Matlab and Python implementations)) — github.com/Coffey-Lab/MIRIAM

## Examples

```
library(Rtsne); tsne_out <- Rtsne(shape_latent_vectors, dims=2)
```

## Evaluation signals

- Output files exist and contain one 2D coordinate pair per held-out cell (verified against the task validation step).
- The shape-based t-SNE scatter plot groups cells with similar geometries, reproducing the Figure 3B panel.
- As reported (Supplemental Figure 5), the embedding reveals no discernible cellular populations and no clear patterns in marker expression — a strong visible pattern would indicate a deviation from the reference result.
- Only latent vectors from held-out cells (excluded from autoencoder training) are used as t-SNE input.
- Overlay of compartment-quantified marker intensities (e.g., p-ERK) is present on the embedding for the correlation test.

## Limitations

- In the source study, t-SNE of autoencoder shape latent vectors did not reveal discernible cellular populations or clear patterns in marker expression, so the skill may yield negative results.
- Only cells excluded from autoencoder training can be used, limiting sample size to the held-out set.
- The MIRIAM repository README notes example data excludes image stacks (too large); image stacks must be regenerated by the user, which affects reproducibility of upstream latent vectors.

## Evidence
