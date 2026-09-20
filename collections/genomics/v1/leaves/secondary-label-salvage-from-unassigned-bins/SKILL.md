---
name: secondary-label-salvage-from-unassigned-bins
description: Use when a Visium HD 2 um bin object has been segmented on the scaled
  morphology image but regions of clear gene-expression signal lack a primary nucleus
  label, and you want to recover those cells before bin_to_cell assembly.
when_to_use_negative:
- 'When tissue regions are dense: the secondary grid-image segmentation is less robust
  than morphology-based nucleus detection and struggles in denser tissue regions'
- When you have not yet produced destriped 2 um bins and a scaled morphology image
  — the grid image and salvage step depend on both
- When the goal is probe design for Visium HD/FFPE panels — that is the separate gene2probe
  tool
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: bin2cell
  role: Provides grid_image, stardist, insert_labels, salvage_secondary_labels, and
    bin_to_cell used to render expression, segment it, and merge secondary labels
    into the final cell object
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Fluorescence StarDist model segments the gene-expression grid image; optional
    dependency installed via pip install bin2cell[stardist] and requires TensorFlow
  repo: https://github.com/Teichlab/bin2cell
- name: numpy
  role: Array backend for the shared label array and grid image manipulation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:dcd53372f28cced9f88dfb8b0005398860c59fe84ae30e87280e14af4845f764
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - StarDist
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# secondary-label-salvage-from-unassigned-bins

## Summary

A bin2cell secondary segmentation stage that recovers putative Visium HD cells missed by morphology-based nucleus detection: total per-bin gene expression is rendered onto the array grid, segmented with StarDist's fluorescence model, and resulting labels are salvaged only where no primary morphology label exists.

## When to use

Use when a Visium HD 2 um bin object has been segmented on the scaled morphology image but regions of clear gene-expression signal lack a primary nucleus label, and you want to recover those cells before bin_to_cell assembly.

## When NOT to use

- When tissue regions are dense: the secondary grid-image segmentation is less robust than morphology-based nucleus detection and struggles in denser tissue regions
- When you have not yet produced destriped 2 um bins and a scaled morphology image — the grid image and salvage step depend on both
- When the goal is probe design for Visium HD/FFPE panels — that is the separate gene2probe tool

## Inputs

- Destriped Visium HD bin-count AnnData with 2 um bins (after bin2cell.destripe)
- Scaled fluorescence/H&E morphology image (from bin2cell.scaled_he_image or bin2cell.scaled_if_image)
- Primary morphology-based segmentation labels already inserted into the label array

## Outputs

- Label array with retained secondary labels only where no primary label exists
- Final cell AnnData (cells.h5ad) with per-cell counts, spatial locations, morphology image crops, and per-cell bin counts

## How to apply

Load the destriped Visium HD bin-count AnnData (2 um bins) and the scaled morphology image, then call bin2cell.grid_image to render total transcript counts per bin onto the array grid as a gene-expression visualisation image. Segment this grid image with the StarDist fluorescence model, keeping the image resolution in the recommended mpp range of 0.2-0.5 (models were trained around mpp 0.3) and tuning prob_thresh so detected objects correspond to cell-associated expression islands. Insert the secondary labels into the shared label array with bin2cell.insert_labels, then run bin2cell.salvage_secondary_labels so that only secondary labels whose bins carry no primary (morphology-based) label are retained as additional cells. Finally assemble the cell object with bin2cell.bin_to_cell, which groups bins, accumulates counts, spatial locations, the morphology image crop, and per-cell bin counts, and write the result to cells.h5ad. The rationale is that morphology segmentation fails when nuclei do not appear in the tissue slice or have unusual shapes, whereas expression islands on the grid still mark the cell.

## Related tools

- **bin2cell** (Provides grid_image, stardist, insert_labels, salvage_secondary_labels, and bin_to_cell used to render expression, segment it, and merge secondary labels into the final cell object) — https://github.com/Teichlab/bin2cell
- **StarDist** (Fluorescence StarDist model segments the gene-expression grid image; optional dependency installed via pip install bin2cell[stardist] and requires TensorFlow) — https://github.com/Teichlab/bin2cell
- **numpy** (Array backend for the shared label array and grid image manipulation)

## Examples

```
b2c.grid_image(adata, mpp=0.35); b2c.stardist(adata, 'he', prob_thresh=0.1, model='2D_versatile_fluo'); b2c.insert_labels(adata); b2c.salvage_secondary_labels(adata); b2c.bin_to_cell(adata)
```

## Evaluation signals

- Secondary labels retained after salvage_secondary_labels must have no overlap with primary morphology-based labels in their bins
- Detected StarDist objects on the grid image should correspond visually to cell-associated expression islands, with mpp in the 0.2-0.5 range
- Final cells.h5ad must contain, per cell, accumulated counts, spatial locations, a morphology image crop, and per-cell bin counts for the grouped 2 um bins
- Segmentation performance should improve relative to a poorly chosen mpp (e.g. dropping from mpp=0.5 to 0.35 with prob_thresh=0.1 'drastically improved' segmentation in the article)

## Limitations

- Secondary segmentation is less robust than morphology-based nucleus detection and struggles in denser tissue regions
- StarDist model performance is highly dependent on input image resolution; models were trained around mpp 0.3, so wrong mpp degrades labels
- StarDist is an optional dependency group and TensorFlow must be installed separately for segmentation
- numpy is capped at 1.x; no exact version pin is given

## Evidence

- [methods] Secondary segmentation rationale (salvage of morphology-missed nuclei): [evidence span withheld — anchor status: not_found]
- [methods] Secondary segmentation robustness limitation: [evidence span withheld — anchor status: not_found]
- [methods] Recommended mpp range for StarDist: "we would recommend using the ``mpp`` (microns per pixel) range of 0.2-0.5"
- [methods] Salvage function availability in the package: "bin2cell.salvage_secondary_labels"
- [intro] Bin2cell purpose: group bins via segmentation on morphology and/or expression: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
- [readme] StarDist and TensorFlow install requirement from README: [evidence span withheld — anchor status: external_doc]
