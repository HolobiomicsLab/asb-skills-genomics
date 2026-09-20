---
name: gene-expression-grid-image-generation
description: Use when working with Visium HD 2 um bin data and you need to recover
  cells whose nuclei were not detected on the morphology image (e.g., nuclei absent
  from the tissue slice or with unusual shapes) by segmenting total gene expression
  rendered on the array grid.
when_to_use_negative:
- When morphology-based nucleus detection already succeeded on the H&E or IF image
  — the secondary grid-image segmentation is less robust and is only a salvage step
- In dense tissue regions where expression islands do not resolve into separable cell-associated
  clusters, since this segmentation struggles there
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3679
- http://edamontology.org/topic_6105
tools:
- name: bin2cell
  role: Provides bin2cell.grid_image to render total per-bin transcript counts onto
    the array grid, plus insert_labels and salvage_secondary_labels to merge the secondary
    labels
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Fluorescence StarDist model used to segment the gene-expression grid image;
    requires installing bin2cell[stardist] and TensorFlow
  repo: https://github.com/stardist/stardist
- name: Visium HD
  role: Source platform producing the subcellular 2 um resolution bin-count data that
    is rendered onto the array grid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:0c73d18522a01cb940212fac49b17b857b070a792e9640317e381a9caf109d6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - StarDist
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3679
  - http://edamontology.org/topic_6105
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-expression-grid-image-generation

## Summary

Renders total per-bin transcript counts from destriped Visium HD 2 um bin-count AnnData onto the spatial array grid as a gene-expression visualisation image (bin2cell.grid_image), enabling StarDist fluorescence-model segmentation of expression islands to recover putative cells missed by morphology-based nucleus detection.

## When to use

Use when working with Visium HD 2 um bin data and you need to recover cells whose nuclei were not detected on the morphology image (e.g., nuclei absent from the tissue slice or with unusual shapes) by segmenting total gene expression rendered on the array grid.

## When NOT to use

- When morphology-based nucleus detection already succeeded on the H&E or IF image — the secondary grid-image segmentation is less robust and is only a salvage step
- In dense tissue regions where expression islands do not resolve into separable cell-associated clusters, since this segmentation struggles there

## Inputs

- Destriped Visium HD bin-count AnnData at 2 um bins (e.g., output of bin2cell.destripe)
- Scaled fluorescence morphology image (output of bin2cell.scaled_if_image)

## Outputs

- Gene-expression grid image (array-grid visualisation of total counts per bin)
- Secondary StarDist labels for expression islands, subsequently filtered by bin2cell.salvage_secondary_labels to bins lacking primary labels

## How to apply

Load the destriped Visium HD bin-count AnnData (2 um bins) and the scaled fluorescence morphology image, then call bin2cell.grid_image to render total transcript counts per bin onto the array grid as a visualisation image. Segment this grid image with the StarDist fluorescence model (b2c.stardist), keeping the image resolution in the recommended mpp range of 0.2–0.5 (StarDist models were trained at mpp ~0.3) and tuning prob_thresh (e.g., setting it back to 0.1 after rescaling from 0.5 to 0.35 mpp drastically improved segmentation of a Visium HD lung IF image). Insert the resulting secondary labels into the shared label array with bin2cell.insert_labels, then call bin2cell.salvage_secondary_labels so only labels whose bins carry no primary (morphology-based) label are retained as additional cells. The rationale is that spatial clusters of total per-bin expression correspond to cell-associated expression islands, providing a secondary route to cell grouping when morphology-based nucleus detection fails.

## Related tools

- **bin2cell** (Provides bin2cell.grid_image to render total per-bin transcript counts onto the array grid, plus insert_labels and salvage_secondary_labels to merge the secondary labels) — https://github.com/Teichlab/bin2cell
- **StarDist** (Fluorescence StarDist model used to segment the gene-expression grid image; requires installing bin2cell[stardist] and TensorFlow) — https://github.com/stardist/stardist
- **Visium HD** (Source platform producing the subcellular 2 um resolution bin-count data that is rendered onto the array grid)

## Examples

```
import bin2cell as b2c; b2c.grid_image(adata, scaling=32); b2c.stardist(grid_im, mpp=0.35, prob_thresh=0.1, model='2D_versatile_fluo'); b2c.insert_labels(adata, labels); b2c.salvage_secondary_labels(adata)
```

## Evaluation signals

- The grid image resolution falls within the recommended mpp range of 0.2–0.5 (models trained at mpp ~0.3); rescaling from mpp 0.5 to 0.35 with prob_thresh 0.1 should visibly improve StarDist segmentation
- After salvage_secondary_labels, retained secondary labels overlap only bins that carry no primary (morphology-based) label
- Detected grid-image objects correspond to spatially coherent cell-associated expression islands rather than noise or merged multi-cell blobs
- Downstream bin_to_cell output (cells.h5ad) contains cells grouped from 2 um bins that were only recovered by the secondary pass, with per-cell counts, spatial locations, and morphology crops

## Limitations

- Secondary grid-image segmentation is less robust than morphology-based nucleus detection and struggles in denser tissue regions
- StarDist is an optional dependency (install via pip install bin2cell[stardist] plus TensorFlow), so default installs may lack the segmentation model dependency
- StarDist model performance is highly dependent on input image resolution; using an mpp outside 0.2–0.5 degrades segmentation

## Evidence

- [other] Secondary segmentation identifies spatial clusters of total per-bin gene expression via StarDist's fluorescence model on the array grid: [evidence span withheld — anchor status: not_found]
- [readme] Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression.: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
- [methods] StarDist models were trained on images with an mpp around 0.3; recommended mpp range is 0.2-0.5: [evidence span withheld — anchor status: not_found]
- [methods] Changing mpp from 0.5 to 0.35 drastically improved StarDist segmentation of a Visium HD lung IF image: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [readme] For b2c.stardist() to work, StarDist needs to be installed. Bin2cell comes with a dependency group to handle this: [evidence span withheld — anchor status: external_doc]
