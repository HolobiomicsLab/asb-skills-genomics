---
name: cell-type-neighbourhood-proportion-computation
description: Use when comparing the fidelity of cell-to-cell neighbourhood (cell-cell
  association) structure between two spatial transcriptomics representations — e.g.
  bin2cell-reconstructed cells vs default 8 μm Visium HD bins — against an expert-annotated
  MERFISH reference.
when_to_use_negative:
- The spatial objects carry no cell-type labels and none can be transferred from an
  annotated reference — neighbourhood proportions are undefined without per-cell/bin
  type assignments.
- The question is expression-level accuracy (e.g. destriped vs non-destriped expression)
  rather than spatial cell-cell association structure — other bin2cell comparisons
  cover that.
- The tissue has no expert-annotated high-resolution ground truth (the MSE validation
  here is specific to the MERFISH mouse brain reference from Zhang et al. 2023).
tools:
- name: bin2cell
  role: Provides the reconstructed cell objects (grouped 2 μm bins) whose neighbourhood
    proportions are evaluated; bin_to_cell produces the cells from segmentation-based
    groupings.
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: Source assay generating the 2 μm bin-resolution data used to build both the
    b2c cell object and the default 8 μm bin object.
- name: numpy
  role: Numerical computation of the mean squared error between neighbourhood-proportion
    vectors and the MERFISH ground truth; kept capped at 1.x per the repository guidance.
  repo: https://github.com/Teichlab/bin2cell
- name: Python
  role: Compatible with established Python single cell and spatial transcriptomics
    software; used to build neighbour graphs, transfer labels, and compute/aggregate
    MSE values.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:6a87512ecb1c5b3acc29fd5ae6330028a24e90fd85aa269263fd66fcf7cf09ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cell-type-neighbourhood-proportion-computation

## Summary

Quantifies cell-cell association patterns by computing, per cell type, the mean fraction of spatial neighbours belonging to each cell type on b2c-reconstructed Visium HD cells versus default 8 μm bins, scoring each against Zhang et al. 2023 MERFISH mouse brain ground truth by mean squared error (b2c MSE 0.00086 vs 8 μm bins MSE 0.00167).

## When to use

Use when comparing the fidelity of cell-to-cell neighbourhood (cell-cell association) structure between two spatial transcriptomics representations — e.g. bin2cell-reconstructed cells vs default 8 μm Visium HD bins — against an expert-annotated MERFISH reference.

## When NOT to use

- The spatial objects carry no cell-type labels and none can be transferred from an annotated reference — neighbourhood proportions are undefined without per-cell/bin type assignments.
- The question is expression-level accuracy (e.g. destriped vs non-destriped expression) rather than spatial cell-cell association structure — other bin2cell comparisons cover that.
- The tissue has no expert-annotated high-resolution ground truth (the MSE validation here is specific to the MERFISH mouse brain reference from Zhang et al. 2023).

## Inputs

- b2c-reconstructed cell AnnData object (2 μm bins grouped via bin2cell.bin_to_cell) for the mouse brain Visium HD sample
- Default 8 μm bin AnnData object for the same Visium HD sample
- Expert cell-type annotations from Zhang et al. 2023 MERFISH mouse brain data (ground truth)

## Outputs

- Per-object mean neighbourhood-proportion matrices (cell type x neighbour cell type)
- Per-object MSE values against the MERFISH reference (b2c cells: ~0.00086; 8 μm bins: ~0.00167)
- mse_comparison.csv comparison table

## How to apply

Load the b2c-reconstructed cell object (grouped 2 μm bins via bin2cell.bin_to_cell) and the default 8 μm SpaceRanger bin object for the mouse brain Visium HD sample, plus expert cell-type annotations from the Zhang et al. 2023 MERFISH mouse brain data. Transfer the MERFISH-derived cell-type labels onto each Visium HD object so every cell/bin carries a ground-truth-consistent label. Build a spatial neighbour graph over each object and compute, per cell type, the mean neighbourhood proportion (fraction of neighbours belonging to each cell type). Compute the MSE of these proportions against the MERFISH ground truth per object and assemble a comparison table written to mse_comparison.csv. Accept the result when the recomputed b2c-cell MSE is close to 0.00086 and below the 8 μm bin MSE (~0.00167), demonstrating the paper's claim that reconstructed cells improve downstream analysis over default 8 μm bins.

## Related tools

- **bin2cell** (Provides the reconstructed cell objects (grouped 2 μm bins) whose neighbourhood proportions are evaluated; bin_to_cell produces the cells from segmentation-based groupings.) — https://github.com/Teichlab/bin2cell
- **Visium HD** (Source assay generating the 2 μm bin-resolution data used to build both the b2c cell object and the default 8 μm bin object.)
- **numpy** (Numerical computation of the mean squared error between neighbourhood-proportion vectors and the MERFISH ground truth; kept capped at 1.x per the repository guidance.) — https://github.com/Teichlab/bin2cell
- **Python** (Compatible with established Python single cell and spatial transcriptomics software; used to build neighbour graphs, transfer labels, and compute/aggregate MSE values.)

## Examples

```
import numpy as np; mse_b2c = np.mean((props_b2c - props_merfish)**2)
```

## Evaluation signals

- Recomputed MSE for the b2c cell object is close to 0.00086.
- Recomputed MSE for the default 8 μm bin object is close to 0.00167.
- The b2c-cell MSE is lower than the 8 μm-bin MSE, reproducing the reported improvement.
- mse_comparison.csv is written with per-object MSE rows and matches the in-memory values.
- Neighbourhood-proportion rows are non-negative and each per-cell-type proportion vector sums to 1.

## Limitations

- The MSE thresholds (0.00086 / 0.00167) are specific to the mouse brain Visium HD sample paired with the Zhang et al. 2023 MERFISH annotations; other tissues lack a validated ground truth here.
- numpy is only loosely constrained (capped at 1.x) with no exact pin, a reproducibility gap when recomputing the metrics.
- Results depend on the b2c reconstruction parameters (segmentation, bin_to_cell grouping); no default performance characterization is given for user-controlled parameters.

## Evidence

- [abstract] Bin2cell's improvements are validated by better cell-cell association fidelity against MERFISH ground truth: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [intro] Reconstructed cell objects carry spatial information enabling neighbour-graph construction: "The end result is an object with cells, created from grouped 2um bins assigned to the same object after segmentation, carrying spatial information and sharper morphology images for visualisation."
- [readme] Repository provides notebooks comparing bin2cell output to standard 8um SpaceRanger output for the mouse brain, the same contrast this skill quantifies: [evidence span withheld — anchor status: external_doc]
