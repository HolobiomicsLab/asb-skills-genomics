---
name: fluorescence-model-nuclei-segmentation
description: Use when processing Visium HD 2 um bin AnnData where morphology-based
  nucleus segmentation failed to label cell-associated expression islands (e.g. nuclei
  absent from the tissue slice or unusually shaped), and you need to salvage them
  as additional cells.
when_to_use_negative:
- Primary morphology-based nucleus detection already labels the bins of interest —
  salvage_secondary_labels retains only secondary labels whose bins carry no primary
  label
- Dense tissue regions where expression-grid segmentation is unreliable — the secondary
  stage is documented as less robust than morphology-based nucleus detection in dense
  tissue
- You only need standard 8 um SpaceRanger output rather than cell reconstruction from
  2 um bins
tools:
- name: bin2cell
  role: Provides grid_image, stardist, insert_labels, salvage_secondary_labels and
    bin_to_cell to render the expression grid, segment it, salvage unclaimed secondary
    labels, and assemble the final cell object
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Fluorescence model used to segment the gene-expression grid image; requires
    bin2cell's optional [stardist] dependency group and TensorFlow
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: Source data platform capturing full-scale transcriptomics at 2 um bins paired
    with a reference morphology image
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:eabf309aae354866b1ea40642b336dc14160d17a3b9bc851151d26cc53ddf4b1
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# fluorescence-model-nuclei-segmentation

## Summary

A bin2cell secondary segmentation stage that segments a grid-rendered image of total per-bin gene expression (e.g. 2 um Visium HD bins) with StarDist's fluorescence model to recover putative cells that morphology-based nucleus detection missed.

## When to use

Use when processing Visium HD 2 um bin AnnData where morphology-based nucleus segmentation failed to label cell-associated expression islands (e.g. nuclei absent from the tissue slice or unusually shaped), and you need to salvage them as additional cells.

## When NOT to use

- Primary morphology-based nucleus detection already labels the bins of interest — salvage_secondary_labels retains only secondary labels whose bins carry no primary label
- Dense tissue regions where expression-grid segmentation is unreliable — the secondary stage is documented as less robust than morphology-based nucleus detection in dense tissue
- You only need standard 8 um SpaceRanger output rather than cell reconstruction from 2 um bins

## Inputs

- Destriped Visium HD bin-count AnnData at 2 um bins
- Scaled fluorescence morphology image (e.g. from bin2cell.scaled_if_image)
- StarDist fluorescence segmentation model (installed via bin2cell[stardist] plus TensorFlow)

## Outputs

- Label array augmented with secondary (expression-grid) labels for bins lacking primary labels
- Final reconstructed cell AnnData (cells.h5ad) with counts, spatial coordinates, morphology image crop, and per-cell bin counts

## How to apply

Load the destriped 2 um bin-count AnnData and the scaled fluorescence morphology image, then call bin2cell.grid_image to render total transcript counts per bin onto the array grid as a gene-expression visualisation image. Segment this grid image with bin2cell.stardist using the fluorescence model, keeping image resolution within the recommended mpp range of 0.2-0.5 (models were trained around mpp 0.3; changing mpp from 0.5 to 0.35 with prob_thresh back at 0.1 drastically improved segmentation of a Visium HD lung IF demo image) and tuning prob_thresh so detected objects correspond to cell-associated expression islands. Insert the secondary labels into the shared label array with bin2cell.insert_labels, then call bin2cell.salvage_secondary_labels so that only labels whose bins carry no primary (morphology-based) label are retained. Finally run bin2cell.bin_to_cell to assemble the final AnnData with all counts, spatial locations, morphology image crops, and per-cell bin counts for each grouped cell, and write it to cells.h5ad.

## Related tools

- **bin2cell** (Provides grid_image, stardist, insert_labels, salvage_secondary_labels and bin_to_cell to render the expression grid, segment it, salvage unclaimed secondary labels, and assemble the final cell object) — https://github.com/Teichlab/bin2cell
- **StarDist** (Fluorescence model used to segment the gene-expression grid image; requires bin2cell's optional [stardist] dependency group and TensorFlow) — https://github.com/Teichlab/bin2cell
- **Visium HD** (Source data platform capturing full-scale transcriptomics at 2 um bins paired with a reference morphology image)

## Examples

```
b2c.grid_image(adata, mpp=0.35); b2c.stardist(adata, lambda_general=0.0, lambda_bound=0.0, prob_thresh=0.1); b2c.insert_labels(adata); b2c.salvage_secondary_labels(adata); b2c.bin_to_cell(adata)
```

## Evaluation signals

- Rendered grid image shows cell-associated expression islands on the array grid at the chosen mpp
- Image resolution falls within the recommended mpp range of 0.2-0.5 (trained around 0.3)
- Salvaged labels are disjoint from primary morphology-based labels (no bin carries both)
- Output cells.h5ad loads as a valid AnnData with counts, spatial locations, morphology crops, and per-cell bin counts
- Segmentation objects correspond to plausible cell-sized expression islands rather than over- or under-merged blobs after prob_thresh tuning

## Limitations

- Secondary segmentation is less robust than morphology-based nucleus detection and struggles in denser tissue regions
- It can only recover putative cells where the morphology stage failed (e.g. nuclei not appearing in the tissue slice or with unusual shapes)
- StarDist is an optional dependency group (plus TensorFlow), so default pip install bin2cell installs lack the segmentation model dependency
- StarDist performance is highly dependent on input image resolution; wrong mpp can drastically degrade segmentation

## Evidence

- [readme] Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression.: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
- [methods] StarDist mpp recommendation: [evidence span withheld — anchor status: not_found]
- [methods] mpp/prob_thresh tuning example: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [readme] StarDist optional dependency: [evidence span withheld — anchor status: external_doc]
- [methods] Secondary segmentation rationale: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
