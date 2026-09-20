---
name: bin-to-cell-object-reconstruction
description: Use when working with Visium HD 2 μm bin-count data and a scaled morphology
  image, and you need cell-level objects more accurate than default 8 μm bins.
when_to_use_negative:
- You only need the default SpaceRanger 8 μm bin output and do not have a high-resolution
  morphology image available
- Input is already a cell-level object (e.g. pre-grouped bins or scRNA-seq counts)
  rather than 2 μm bin-resolution Visium HD data
- You cannot install StarDist/TensorFlow, since segmentation-based bin grouping is
  the core of this skill (StarDist is an optional dependency group)
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bin2cell
  role: Core package providing destripe, scaled_he_image/scaled_if_image, stardist,
    insert_labels, expand_labels, grid_image, salvage_secondary_labels, and bin_to_cell
    for grouping 2 μm bins into cells
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Segmentation model (H&E or fluorescence variants) applied to the morphology
    image and/or gene-expression grid image to propose 2 μm bin-to-cell groupings
  repo: https://github.com/stardist/stardist
- name: Visium HD
  role: Data platform providing subcellular 2 μm resolution transcriptomic data paired
    with a reference morphology image
- name: schard
  role: R package for reading the cells.h5ad output when using bin2cell results from
    R
  repo: https://github.com/cellgeni/schard
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:5307bb82877e95443e8009d2d05fcbf663ef4e6424e38fb14f67d8e97303cec1
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
  - schard
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# bin-to-cell-object-reconstruction

## Summary

Reconstructs cells from 2 μm Visium HD bins by grouping bins via StarDist segmentation of the morphology image and/or a gene-expression grid image, optionally salvaging labels in bins missed by morphology-based detection, and assembling a final cells.h5ad AnnData with spatial information and morphology crops.

## When to use

Use when working with Visium HD 2 μm bin-count data and a scaled morphology image, and you need cell-level objects more accurate than default 8 μm bins.

## When NOT to use

- You only need the default SpaceRanger 8 μm bin output and do not have a high-resolution morphology image available
- Input is already a cell-level object (e.g. pre-grouped bins or scRNA-seq counts) rather than 2 μm bin-resolution Visium HD data
- You cannot install StarDist/TensorFlow, since segmentation-based bin grouping is the core of this skill (StarDist is an optional dependency group)

## Inputs

- Destriped Visium HD bin-count AnnData at 2 μm bins
- Scaled morphology image (H&E or immunofluorescence) at mpp 0.2–0.5
- Gene-expression grid image rendered from total per-bin counts (via bin2cell.grid_image) for secondary salvage
- Spaceranger spatial folder (via b2c.read_visium with spaceranger_image_path)

## Outputs

- Final AnnData object of reconstructed cells written to cells.h5ad, carrying expression counts, spatial information, morphology image crops, and per-cell bin counts

## How to apply

Load the destriped Visium HD 2 μm bin-count AnnData and the scaled fluorescence/H&E morphology image (bin2cell.scaled_he_image or scaled_if_image, keeping mpp in the 0.2–0.5 range since StarDist models were trained at ~0.3 mpp; e.g. mpp=0.35 with prob_thresh=0.1 drastically improved lung IF segmentation). Segment the morphology image with b2c.stardist, insert labels with b2c.insert_labels, optionally expand with b2c.expand_labels, then optionally render total per-bin transcript counts onto the array grid with b2c.grid_image, segment that grid image with the StarDist fluorescence model, and run b2c.salvage_secondary_labels so only grid labels whose bins carry no primary label are retained as extra cells. Finally call b2c.bin_to_cell to group all 2 μm bins per cell, accumulating counts, spatial locations, a morphology image crop, and per-cell bin counts, then write the resulting AnnData to cells.h5ad. Run b2c.destripe first, since bin2cell corrects for a novel technical effect stemming from variable bin dimensions.

## Related tools

- **bin2cell** (Core package providing destripe, scaled_he_image/scaled_if_image, stardist, insert_labels, expand_labels, grid_image, salvage_secondary_labels, and bin_to_cell for grouping 2 μm bins into cells) — https://github.com/Teichlab/bin2cell
- **StarDist** (Segmentation model (H&E or fluorescence variants) applied to the morphology image and/or gene-expression grid image to propose 2 μm bin-to-cell groupings) — https://github.com/stardist/stardist
- **Visium HD** (Data platform providing subcellular 2 μm resolution transcriptomic data paired with a reference morphology image)
- **schard** (R package for reading the cells.h5ad output when using bin2cell results from R) — https://github.com/cellgeni/schard

## Examples

```
pip install bin2cell[stardist]; import bin2cell as b2c; adata = b2c.read_visium(path, spaceranger_image_path=...); b2c.destripe(adata); b2c.stardist(mpp=0.35, prob_thresh=0.1); b2c.salvage_secondary_labels(adata); b2c.bin_to_cell(adata); adata.write('cells.h5ad')
```

## Evaluation signals

- Detected StarDist objects correspond to cell-associated nuclei/expression islands; changing mpp from 0.5 to 0.35 with prob_thresh=0.1 should visibly improve segmentation of IF images
- After salvage_secondary_labels, retained secondary labels occupy bins that carried no primary (morphology-based) label
- Final cells.h5ad contains per-cell counts, spatial coordinates, a morphology image crop, and per-cell bin counts for every grouped cell
- Downstream analysis on reconstructed cells improves over default 8 μm bins (as demonstrated on mouse brain and human colorectal cancer data)

## Limitations

- Secondary (gene-expression grid) segmentation is less robust than morphology-based nucleus detection and struggles in denser tissue regions
- StarDist model performance is highly dependent on input image resolution; models were trained at ~0.3 mpp, so resolutions outside 0.2–0.5 degrade segmentation
- StarDist and TensorFlow are optional dependencies; a plain `pip install bin2cell` lacks them (install via `pip install bin2cell[stardist]`)
- bin2cell runs on CPU in minutes without a GPU, but numpy is only loosely capped at 1.x, a reproducibility gap

## Evidence

- [abstract] Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [methods] StarDist mpp recommendation: [evidence span withheld — anchor status: not_found]
- [methods] bin2cell.salvage_secondary_labels function: "bin2cell.salvage_secondary_labels"
- [readme] README installation of optional StarDist dependency: [evidence span withheld — anchor status: external_doc]
- [readme] README on grouping bins via segmentation: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
