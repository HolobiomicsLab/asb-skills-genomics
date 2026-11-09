---
name: spatial-transcriptomics-anndata-construction
description: Use when starting from destriped Visium HD 2 um bin-count AnnData plus
  a scaled H&E or IF morphology image and you need cell-level objects with spatial
  coordinates instead of default 8 um binned counts.
when_to_use_negative:
- Input is already a cell-level or 8 um binned spatial object — bin_to_cell grouping
  is designed for 2 um bins
- No morphology image or segmentation is available and default SpaceRanger output
  is acceptable
- Tissue is very dense where the secondary (expression-based) segmentation struggles;
  rely on morphology-based nucleus detection instead
edam_operation: http://edamontology.org/operation_3222
edam_topics: []
tools:
- name: bin2cell
  role: Core package performing destriping, scaling, segmentation label insertion,
    salvage, and bin_to_cell grouping into a final AnnData object
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Nucleus segmentation on the scaled H&E/IF image and on the gene-expression
    grid image; optional dependency installed via pip install bin2cell[stardist]
  repo: https://github.com/stardist/stardist
- name: Visium HD
  role: Source platform providing subcellular 2 um resolution transcriptomic data
    paired with a reference morphology image
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:e8be10afcf45159e66fab0d5a36f11a123de0a91a88c4210708b823ec51fa370
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spatial-transcriptomics-anndata-construction

## Summary

Construct a cell-level spatial transcriptomics AnnData object (cells.h5ad) by grouping Visium HD 2 um bins into cells using bin2cell, after morphology-based StarDist segmentation and optional secondary gene-expression salvage, so that reconstructed cells outperform default 8 um bins.

## When to use

Use when starting from destriped Visium HD 2 um bin-count AnnData plus a scaled H&E or IF morphology image and you need cell-level objects with spatial coordinates instead of default 8 um binned counts.

## When NOT to use

- Input is already a cell-level or 8 um binned spatial object — bin_to_cell grouping is designed for 2 um bins
- No morphology image or segmentation is available and default SpaceRanger output is acceptable
- Tissue is very dense where the secondary (expression-based) segmentation struggles; rely on morphology-based nucleus detection instead

## Inputs

- Destriped Visium HD 2 um bin-count AnnData (output of bin2cell.destripe)
- Scaled H&E or immunofluorescence morphology image (bin2cell.scaled_he_image / bin2cell.scaled_if_image) at mpp 0.2-0.5
- Optionally, a gene-expression grid image from bin2cell.grid_image for secondary segmentation

## Outputs

- Cell-by-gene AnnData object (cells.h5ad / crc_bin2cell_cells.h5ad) with cell-level observations, spatial coordinates, morphology image crops, and per-cell bin counts

## How to apply

Load the destriped 2 um bin-count AnnData and the scaled morphology image generated with bin2cell.scaled_he_image or bin2cell.scaled_if_image at an mpp within the recommended 0.2-0.5 range (StarDist models were trained at ~0.3). Segment nuclei with bin2cell.stardist using the H&E or fluorescence model (e.g. prob_thresh=0.05 for H&E at mpp 0.3), then expand labels with bin2cell.expand_labels and map them onto the 2 um bin grid with bin2cell.insert_labels. Optionally render total per-bin counts onto the array grid with bin2cell.grid_image, segment that grid image with the StarDist fluorescence model, insert the secondary labels, and call bin2cell.salvage_secondary_labels so only bins carrying no primary label are retained as additional cells. Finally, group bins into cells with bin2cell.bin_to_cell — which accumulates counts, spatial locations, the morphology image crop, and per-cell bin counts — and write the resulting AnnData to cells.h5ad.

## Related tools

- **bin2cell** (Core package performing destriping, scaling, segmentation label insertion, salvage, and bin_to_cell grouping into a final AnnData object) — https://github.com/Teichlab/bin2cell
- **StarDist** (Nucleus segmentation on the scaled H&E/IF image and on the gene-expression grid image; optional dependency installed via pip install bin2cell[stardist]) — https://github.com/stardist/stardist
- **Visium HD** (Source platform providing subcellular 2 um resolution transcriptomic data paired with a reference morphology image)

## Examples

```
import bin2cell as b2c; b2c.salvage_secondary_labels(bdata); adata = b2c.bin_to_cell(bdata); adata.write('cells.h5ad')
```

## Evaluation signals

- The emitted h5ad loads as an AnnData object with cell-level observations and spatial coordinates
- Each cell carries a morphology image crop and per-cell bin counts from the grouped 2 um bins
- No cell groups both a primary (morphology) label and a salvaged secondary label on the same bins
- Reconstructed cells show improved per-cell gene count coverage over default 8 um bins (as reported for the 10X colorectal cancer demo, correctly predicted as CMS2)
- Segmentation used an mpp within the 0.2-0.5 range and prob_thresh tuned so StarDist objects correspond to cell-associated expression islands

## Limitations

- The secondary (expression-grid) segmentation is less robust than morphology-based nucleus detection and struggles in denser tissue regions
- StarDist is an optional dependency and TensorFlow is additionally required; default installs may lack it
- numpy is only loosely capped at 1.x, no exact pin given, a reproducibility gap; no versioned releases, DOIs, or dataset accessions are stated
- Expansion behavior depends on user-controlled parameters without default performance characterization

## Evidence

- [abstract] Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [methods] StarDist models trained at mpp ~0.3, recommended range 0.2-0.5: [evidence span withheld — anchor status: not_found]
- [readme] README install instructions for StarDist: [evidence span withheld — anchor status: external_doc]
- [readme] README: final object carries spatial information and morphology images: "The end result is an object with cells, created from grouped 2um bins assigned to the same object after segmentation, carrying spatial information and sharper morphology images for visualisation."
