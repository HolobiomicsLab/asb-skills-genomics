---
name: mse-benchmark-evaluation
description: Use when comparing two spatial transcriptomics cell/bin representations
  against an expertly annotated spatial ground truth (e.g. Zhang et al. 2023 MERFISH
  mouse brain) and you need a scalar error metric such as MSE over mean neighbourhood
  cell-type proportions.
when_to_use_negative:
- No spatially resolved ground-truth annotation (e.g. MERFISH reference) exists for
  the tissue, so error against reference proportions cannot be computed
- The objects being compared already carry different cell-type label sets that cannot
  be mapped onto a common reference vocabulary
- The goal is to evaluate reconstruction quality itself (segmentation accuracy) rather
  than downstream neighbourhood-structure fidelity
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bin2cell
  role: Produces the reconstructed-cell object (2 μm bins grouped via bin_to_cell)
    that is one arm of the MSE comparison
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: Source data platform providing the 2 μm and 8 μm bin spatial transcriptomics
    objects being benchmarked
- name: Python
  role: Implementation environment for label transfer, neighbour-graph construction,
    MSE computation, and CSV output
- name: numpy
  role: 'Numerical computation of mean neighbourhood proportions and mean squared
    error (note: capped at 1.x)'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:db6fe4d036eecae17ca544237199a6c867b85f1b87750017b4dd0709963f167a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mse-benchmark-evaluation

## Summary

Quantitative benchmarking of Visium HD cell representations (bin2cell-reconstructed cells vs default 8 μm bins) by comparing cell-type neighbourhood proportions against MERFISH ground truth using mean squared error, quantifying the downstream-analysis improvement reported by bin2cell.

## When to use

Use when comparing two spatial transcriptomics cell/bin representations against an expertly annotated spatial ground truth (e.g. Zhang et al. 2023 MERFISH mouse brain) and you need a scalar error metric such as MSE over mean neighbourhood cell-type proportions.

## When NOT to use

- No spatially resolved ground-truth annotation (e.g. MERFISH reference) exists for the tissue, so error against reference proportions cannot be computed
- The objects being compared already carry different cell-type label sets that cannot be mapped onto a common reference vocabulary
- The goal is to evaluate reconstruction quality itself (segmentation accuracy) rather than downstream neighbourhood-structure fidelity

## Inputs

- bin2cell-reconstructed cell AnnData object (grouped 2 μm bins from bin2cell.bin_to_cell) for the mouse brain Visium HD sample
- Default 8 μm bin AnnData object from SpaceRanger for the same Visium HD sample
- Expert cell-type annotations from Zhang et al. 2023 MERFISH mouse brain data
- Spatial coordinates / neighbour graph construction inputs for each object

## Outputs

- Per-object mean squared error of mean neighbourhood cell-type proportions against MERFISH ground truth (b2c cells: ~0.00086; 8 μm bins: ~0.00167)
- Comparison table written to mse_comparison.csv

## How to apply

Load the bin2cell-reconstructed cell object (grouped 2 μm bins via bin2cell.bin_to_cell) and the default 8 μm bin object for the same Visium HD sample, alongside the expert cell-type annotations from the Zhang et al. 2023 MERFISH mouse brain reference. Map/transfer the MERFISH-derived cell-type labels onto each Visium HD object so every cell or bin carries a ground-truth-consistent label. Build a spatial neighbour graph over each object and compute, per cell type, the mean fraction of neighbours belonging to each cell type (mean neighbourhood proportions). Compute the MSE of these proportions against the MERFISH ground truth for each object and assemble a comparison table (e.g. mse_comparison.csv); in the source work this yielded MSE 0.00086 for b2c cells versus 0.00167 for 8 μm bins, so accept the reconstruction if its MSE is close to 0.00086 and lower than the 8 μm bin MSE.

## Related tools

- **bin2cell** (Produces the reconstructed-cell object (2 μm bins grouped via bin_to_cell) that is one arm of the MSE comparison) — https://github.com/Teichlab/bin2cell
- **Visium HD** (Source data platform providing the 2 μm and 8 μm bin spatial transcriptomics objects being benchmarked)
- **Python** (Implementation environment for label transfer, neighbour-graph construction, MSE computation, and CSV output)
- **numpy** (Numerical computation of mean neighbourhood proportions and mean squared error (note: capped at 1.x))

## Examples

```
import bin2cell as b2c, numpy as np; adata_b2c = b2c.read_h5ad('b2c_cells.h5ad'); adata_8um = b2c.read_h5ad('bins8.h5ad'); mse_b2c = np.mean((props_b2c - props_ref)**2)
```

## Evaluation signals

- Recomputed MSE for the b2c cell object is close to 0.00086
- Recomputed MSE for the b2c cell object is lower than the 8 μm bin MSE (close to 0.00167), demonstrating the reported improvement
- mse_comparison.csv contains per-object MSE rows for both representations
- Every cell/bin in both Visium HD objects carries a mapped cell-type label consistent with the MERFISH reference vocabulary before proportion computation
- Mean neighbourhood proportions are computed per cell type over a spatial neighbour graph for each object under identical graph parameters

## Limitations

- numpy is only loosely constrained (capped at 1.x) with no exact pin, a reproducibility gap for exact MSE reproduction
- No versioned release artifacts, DOIs, or dataset accessions are stated in the README, only the source repository, complicating exact reproduction of the benchmark numbers
- The benchmark is demonstrated on mouse brain (and human colorectal cancer) data only; generalization to other tissues/ground truths is not characterized

## Evidence

- [results] Bin2cell's mean neighbourhood proportions match the MERFISH reference better (MSE 0.00086) than default 8 μm bins (MSE 0.00167) on the mouse brain.: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [methods] The comparison requires transferring MERFISH-derived cell-type labels onto the Visium HD objects and computing per-cell-type mean neighbourhood proportions before MSE.: "The end result is an object with cells, created from grouped 2um bins assigned to the same object after segmentation, carrying spatial information and sharper morphology images for visualisation."
- [readme] The bin2cell repository ships dedicated mouse-brain comparison notebooks against standard 8um SpaceRanger output that this benchmark reproduces.: [evidence span withheld — anchor status: external_doc]
