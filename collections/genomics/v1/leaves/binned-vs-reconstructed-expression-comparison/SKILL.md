---
name: binned-vs-reconstructed-expression-comparison
description: Use when you have Visium HD mouse brain data processed both as default
  8 μm bins and as bin2cell-reconstructed cells (2 μm bins grouped after destripe
  + segmentation), and need to benchmark which representation yields better cell-type
  annotation.
when_to_use_negative:
- You only have one representation (either b2c cells or 8 μm bins) — the comparison
  requires matched objects from the same section
- Your data is not Visium HD (e.g., low-resolution Visium or single-cell data) — 2
  μm bin grouping and the 8 μm default comparison do not apply
- Your tissue/cell types are not covered by the reference CellTypist atlas (Yao et
  al. 2023 mouse brain) — confidence scores would be uninterpretable
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_2229
tools:
- name: bin2cell
  role: reconstructs cells by grouping 2 μm bins via morphology-image segmentation
    and gene expression, after destripe correction; the reconstruction side of the
    comparison
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: source platform capturing full transcriptomic data with reference morphology
    images at 2 μm subcellular bin resolution
- name: CellTypist
  role: annotates both b2c cells and 8 μm bins, yielding per-cell prediction probabilities
    and labels used for the confidence comparison
- name: StarDist
  role: segments the morphology image (recommended mpp range 0.2–0.5, models trained
    around mpp 0.3) to define cell boundaries for bin grouping
- name: Python
  role: implementation environment compatible with established single-cell and spatial
    transcriptomics software
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:27a2b6b722f290c68dc61364a85e3d214d3f985607c9eefd7b5c0ed5f7202d79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - CellTypist
  - Python
  - StarDist
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# binned-vs-reconstructed-expression-comparison

## Summary

Compares default 8 μm SpaceRanger bins against bin2cell-reconstructed cells (grouped 2 μm bins) from Visium HD data using CellTypist annotation confidence and per-cell gene detection, to quantify whether cell reconstruction improves downstream cell-type annotation.

## When to use

Use when you have Visium HD mouse brain data processed both as default 8 μm bins and as bin2cell-reconstructed cells (2 μm bins grouped after destripe + segmentation), and need to benchmark which representation yields better cell-type annotation.

## When NOT to use

- You only have one representation (either b2c cells or 8 μm bins) — the comparison requires matched objects from the same section
- Your data is not Visium HD (e.g., low-resolution Visium or single-cell data) — 2 μm bin grouping and the 8 μm default comparison do not apply
- Your tissue/cell types are not covered by the reference CellTypist atlas (Yao et al. 2023 mouse brain) — confidence scores would be uninterpretable

## Inputs

- b2c-reconstructed cell AnnData (2 μm bins grouped via bin2cell.bin_to_cell after destripe correction and StarDist segmentation)
- default 8 μm SpaceRanger bin AnnData from the same Visium HD mouse brain section
- CellTypist model trained on the Yao et al. 2023 mouse brain atlas

## Outputs

- celltypist_b2c_vs_8um_comparison.csv with per-cell-type median confidence and mean genes detected for b2c vs 8 μm bins
- tally of cell types favoring b2c per metric (expected: 90/113 confidence, 113/113 gene count)

## How to apply

Load the b2c-reconstructed AnnData — built via bin2cell.bin_to_cell after bin2cell.destripe correction and StarDist segmentation of the morphology image — and the default 8 μm bin AnnData. Normalize each expression matrix with library-size normalization and log1p so gene detection is comparable across representations. Annotate both objects with a CellTypist model trained on the Yao et al. 2023 mouse brain atlas to obtain per-cell prediction probabilities (confidence) and predicted labels for all 113 recovered cell types. Aggregate per cell type: median annotation confidence and mean number of genes detected, separately for b2c cells and 8 μm bins. Build a side-by-side comparison table and tally how many cell types favor b2c per metric; the reported result is 90/113 cell types with higher median confidence and 113/113 with higher mean genes detected in b2c. Save as celltypist_b2c_vs_8um_comparison.csv.

## Related tools

- **bin2cell** (reconstructs cells by grouping 2 μm bins via morphology-image segmentation and gene expression, after destripe correction; the reconstruction side of the comparison) — https://github.com/Teichlab/bin2cell
- **Visium HD** (source platform capturing full transcriptomic data with reference morphology images at 2 μm subcellular bin resolution)
- **CellTypist** (annotates both b2c cells and 8 μm bins, yielding per-cell prediction probabilities and labels used for the confidence comparison)
- **StarDist** (segments the morphology image (recommended mpp range 0.2–0.5, models trained around mpp 0.3) to define cell boundaries for bin grouping)
- **Python** (implementation environment compatible with established single-cell and spatial transcriptomics software)

## Examples

```
import bin2cell as b2c; b2c.destripe(adata); b2c.bin_to_cell(adata); labels_adata = b2c.read_labels('stardist_labels.npy'); b2c.insert_labels(adata, labels_adata)
```

## Evaluation signals

- The comparison table shows higher median CellTypist confidence in b2c for 90 of 113 cell types
- The comparison table shows higher mean genes detected in b2c for all 113 cell types
- Both AnnData objects are normalized (library-size + log1p) before counting genes per cell/bin so detection is comparable
- The same 113 cell types from the Yao et al. 2023 atlas model are recovered for both representations
- Output file celltypist_b2c_vs_8um_comparison.csv contains per-cell-type rows with both metrics for both representations

## Limitations

- numpy is only loosely constrained (capped at 1.x), an exact version pin is missing, which is a reproducibility gap
- StarDist is an optional dependency group (pip install bin2cell[stardist]) plus TensorFlow, so a default install lacks the segmentation dependency
- CellTypist confidence depends on the reference atlas (Yao et al. 2023 mouse brain); results do not transfer to tissues outside the atlas scope
- Expansion parameters (k, max_call_distance) are user-controlled without default performance characterization

## Evidence

- [abstract] Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [abstract] improvements over 8 μm bins demonstrated: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [methods] bin_to_cell grouping: "bin2cell.bin_to_cell"
- [methods] StarDist mpp guidance: "we would recommend using the ``mpp`` (microns per pixel) range of 0.2-0.5"
- [readme] README notebooks comparing b2c to 8um SpaceRanger output: [evidence span withheld — anchor status: external_doc]
- [discussion] numpy versioning caveat: "cap `numpy` at 1.x for the time being"
