---
name: quantile-based-count-normalization
description: Use when raw Visium HD 2 μm bin count data show a striped pattern in
  per-bin count totals from variable bin dimensions, prior to bin_to_cell grouping.
when_to_use_negative:
- Input consists of aggregated default 8 μm SpaceRanger output rather than 2 μm bins,
  which do not exhibit the variable-bin-dimension striping being corrected
- Input is already a destriped count matrix or a cell-level object produced after
  bin2cell.bin_to_cell grouping
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: bin2cell
  role: Provides bin2cell.destripe, which applies the row/column quantile factor correction
    to the 2 μm bin count matrix
  repo: https://github.com/Teichlab/bin2cell
- name: numpy
  role: Array operations used inside destripe to divide the count matrix by combined
    row/column factors and rescale corrected totals
- name: Visium HD
  role: Source platform whose 2 μm bins carry the variable-bin-dimension striping
    effect being corrected
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:450a4f133513aa59718bf170c16929e82c867e922e3d7c3be73203510c8768b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# quantile-based-count-normalization

## Summary

Destripes raw Visium HD 2 μm bin counts by dividing each bin's count total by row-wise and column-wise 0.99-quantile factors (bin2cell.destripe), correcting a novel striping artifact caused by variable bin dimensions before 2 μm bins are grouped into cells.

## When to use

Use when raw Visium HD 2 μm bin count data show a striped pattern in per-bin count totals from variable bin dimensions, prior to bin_to_cell grouping.

## When NOT to use

- Input consists of aggregated default 8 μm SpaceRanger output rather than 2 μm bins, which do not exhibit the variable-bin-dimension striping being corrected
- Input is already a destriped count matrix or a cell-level object produced after bin2cell.bin_to_cell grouping

## Inputs

- Raw Visium HD 2 μm bin count matrix (bins × genes) loaded as an AnnData object (h5ad)
- Bin array row and column coordinates within the Visium HD capture grid

## Outputs

- Destriped 2 μm bin count matrix saved as destriped_counts.h5ad
- Corrected per-bin count totals preserving the original bins-by-genes shape and gene identifiers

## How to apply

Load the raw 2 μm bin count matrix of the Visium HD dataset into an AnnData object with scanpy. Compute, for each array row, the 0.99 quantile of per-bin count totals across genes and rescale these factors into count space; repeat the procedure per column. Divide the count matrix by the combined row/column factors using bin2cell.destripe (implemented with numpy array operations), with the quantile a user-specified parameter defaulting to 0.99. Rescale the matrix so the corrected per-bin count totals are preserved, then save the result as destriped_counts.h5ad. The rationale is that bins at row/column edges have smaller capture areas, so row- and column-wise high quantiles absorb the systematic striping while preserving per-bin totals in count space.

## Related tools

- **bin2cell** (Provides bin2cell.destripe, which applies the row/column quantile factor correction to the 2 μm bin count matrix) — https://github.com/Teichlab/bin2cell
- **numpy** (Array operations used inside destripe to divide the count matrix by combined row/column factors and rescale corrected totals)
- **Visium HD** (Source platform whose 2 μm bins carry the variable-bin-dimension striping effect being corrected)

## Examples

```
import bin2cell as b2c; b2c.destripe(adata, quantile=0.99)
```

## Evaluation signals

- The corrected matrix retains the original bins-by-ges shape and gene identifiers of the raw 2 μm bin AnnData
- The applied per-row and per-column factors equal 0.99-quantile-derived values rescaled into count space
- The striped pattern in per-bin count totals across array rows and columns is removed after correction
- Per-bin totals of the output matrix match the corrected count totals after rescaling

## Limitations

- The quantile cutoff is user-specified (default 0.99), so correction strength depends on the chosen value with no characterization given
- numpy is only loosely constrained (capped at 1.x), a reproducibility gap for the numpy array operations in destripe
- The correction targets the variable-bin-dimension striping specifically and is not a general-purpose count normalization

## Evidence

- [intro] The package also corrects for a novel technical effect in the data stemming from variable bin dimensions.: "The package also corrects for a novel technical effect in the data stemming from variable bin dimensions."
- [readme] Visium HD captures gene expression data at a subcellular 2um resolution.: "Visium HD captures gene expression data at a subcellular 2um resolution."
- [abstract] Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
