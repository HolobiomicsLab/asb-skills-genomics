---
name: spatial-transcriptomics-matrix-processing
description: Use when working with 10X Visium HD 2 μm bin count matrices (AnnData/h5ad)
  that show a striped pattern in per-bin count totals, or when preparing bins for
  downstream cell reconstruction and neighbourhood analyses.
when_to_use_negative:
- Input is already aggregated to 8 μm bins or reconstructed cells — the destripe correction
  targets the variable-dimension artifact of 2 μm bins
- Data are not Visium HD spatial count matrices (e.g. scRNA-seq or non-spatial bulk
  data)
- You only need image segmentation (StarDist) rather than count-matrix correction
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: bin2cell
  role: Provides destripe/destripe_counts for row/column quantile count-total correction
    of 2 μm Visium HD bins, and bin_to_cell for grouping bins into cells used in downstream
    comparisons
  repo: https://github.com/Teichlab/bin2cell
- name: numpy
  role: Array operations underlying the destripe rescaling of count-space factors
    and the count matrix
- name: Visium HD
  role: Source platform producing the 2 μm and 8 μm binned spatial transcriptomics
    matrices being processed
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:9a9b7282b02cc739fd8743462b46389f0b921bf388f178df8625dd2aa09443c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spatial-transcriptomics-matrix-processing

## Summary

Processing of high-resolution Visium HD count matrices in Python, most notably the bin2cell 'destripe' correction that removes the striped artifact in 2 μm bin count totals caused by variable bin dimensions, using per-row and per-column quantile normalization.

## When to use

Use when working with 10X Visium HD 2 μm bin count matrices (AnnData/h5ad) that show a striped pattern in per-bin count totals, or when preparing bins for downstream cell reconstruction and neighbourhood analyses.

## When NOT to use

- Input is already aggregated to 8 μm bins or reconstructed cells — the destripe correction targets the variable-dimension artifact of 2 μm bins
- Data are not Visium HD spatial count matrices (e.g. scRNA-seq or non-spatial bulk data)
- You only need image segmentation (StarDist) rather than count-matrix correction

## Inputs

- Raw Visium HD 2 μm bin count matrix (SpaceRanger output) loaded as an AnnData object
- Destripe quantile parameter (default 0.99)
- Optionally an 8 μm binned object for comparison analyses
- Reference cell-type annotations (e.g. Zhang et al. 2023 MERFISH mouse brain) for downstream validation

## Outputs

- Destriped bins-by-genes count matrix saved as destriped_counts.h5ad, retaining original bins, genes, and corrected per-bin totals
- Comparison metrics such as neighbourhood-proportion MSE (e.g. b2c cells 0.00086 vs 8 μm bins 0.00167) written to tables like mse_comparison.csv

## How to apply

Load the raw 2 μm bin count matrix into an AnnData object (e.g. via scanpy from the SpaceRanger output). For the variable-bin-dimension ('striped') artifact, compute the 0.99 quantile factor per array row across genes and per array column across bins, rescale both factor sets into count space, then divide the count matrix by the combined row/column factors and rescale the matrix so the corrected per-bin count totals are preserved (bin2cell.destripe / destripe_counts, implemented with numpy array operations). Save the result as an h5ad (e.g. destriped_counts.h5ad). Destriping is done on the 2 μm bins before bin_to_cell grouping, and corrected matrices feed downstream comparisons such as neighbourhood-proportion MSE against MERFISH references.

## Related tools

- **bin2cell** (Provides destripe/destripe_counts for row/column quantile count-total correction of 2 μm Visium HD bins, and bin_to_cell for grouping bins into cells used in downstream comparisons) — https://github.com/Teichlab/bin2cell
- **numpy** (Array operations underlying the destripe rescaling of count-space factors and the count matrix)
- **Visium HD** (Source platform producing the 2 μm and 8 μm binned spatial transcriptomics matrices being processed)

## Examples

```
import bin2cell as b2c; adata = sc.read_10x_h5('.../cell_feature_bc_matrix.h5'); b2c.destripe(adata); adata.write('destriped_counts.h5ad')
```

## Evaluation signals

- Destriped matrix retains the original bins-by-genes shape and gene identifiers of the input AnnData
- Applied per-row and per-column factors equal the 0.99-quantile-derived values rescaled into count space
- Per-bin count totals after correction no longer show the striped row/column pattern visible in the raw 2 μm bins
- For neighbourhood-proportion validation, recomputed MSE for b2c cells should be close to 0.00086 and lower than the 8 μm bin MSE close to 0.00167 against the MERFISH ground truth
- Output h5ad (e.g. destriped_counts.h5ad) loads correctly and preserves original gene identifiers

## Limitations

- The quantile (default 0.99) is user-specified and its value is a tunable choice without a characterized default-performance analysis
- numpy is only loosely pinned (capped at 1.x), which is a reproducibility gap
- StarDist is an optional dependency group, so a plain 'pip install bin2cell' may lack segmentation dependencies (relevant when extending the workflow to cell reconstruction)

## Evidence

- [methods] The correction works by dividing each 2 μm bin's count total by a user-specified quantile (default 0.99) from its array row, repeating the procedure per column, then rescaling the count matrix per bin to the corrected count totals.: [evidence span withheld — anchor status: not_found]
- [intro] The package also corrects for a novel technical effect in the data stemming from variable bin dimensions.: "The package also corrects for a novel technical effect in the data stemming from variable bin dimensions."
- [readme] Visium HD captures gene expression data at a subcellular 2um resolution. It should be possible to use this data to reconstruct cells more accurately than just using the next resolution up (8um): "Visium HD captures gene expression data at a subcellular 2um resolution."
- [results] Bin2cell's mean neighbourhood proportions match the MERFISH reference better (MSE 0.00086) than default 8 μm bins (MSE 0.00167) on the mouse brain.: [evidence span withheld — anchor status: not_found]
- [discussion] StarDist is an optional dependency group, global import removed: "StarDist moved to optional dependency group, global import removed"
