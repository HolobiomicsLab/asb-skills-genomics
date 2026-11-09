---
name: per-cell-type-confidence-aggregation
description: Use when comparing cell-type annotation quality between two representations
  of Visium HD data (e.g. bin2cell cells vs 8 μm bins) and you need per-cell-type
  tallies of median confidence and mean genes detected.
when_to_use_negative:
- Input is a single AnnData representation with no matched comparison object (e.g.
  only b2c cells or only 8 μm bins) — no comparison table can be built.
- The CellTypist model was not trained on a matching species/tissue atlas (e.g. a
  human atlas applied to mouse brain data) — confidence scores would not be interpretable
  across 113 mouse brain cell types.
- Expression matrices are raw counts without library-size normalization and log1p
  — gene detection counts would not be comparable across representations.
edam_operation: http://edamontology.org/operation_3800
edam_topics: []
tools:
- name: bin2cell
  role: Produces the reconstructed cell AnnData from 2 μm bins (bin2cell.destripe,
    bin2cell.bin_to_cell) that is compared against 8 μm bins
  repo: https://github.com/Teichlab/bin2cell
- name: CellTypist
  role: Annotates both objects with per-cell prediction probabilities (confidence)
    and predicted labels used for per-cell-type aggregation
- name: Visium HD
  role: Source spatial transcriptomics platform providing 2 μm and 8 μm bin data from
    mouse brain
- name: Python
  role: Implementation environment for normalization, CellTypist annotation, and per-cell-type
    aggregation
- name: numpy
  role: Numerical aggregation of per-cell-type medians and means
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:15bc4964d2c3c4e098c7468aa773fb7b91c6dd963d3d30c5e1c56dd8b780b25a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - CellTypist
  - Python
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_3800
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# per-cell-type-confidence-aggregation

## Summary

Aggregate CellTypist annotation confidence and gene detection per cell type to compare two Visium HD representations (bin2cell-reconstructed cells vs default 8 μm bins), quantifying how many of the 113 Yao et al. 2023 atlas cell types favor each representation.

## When to use

Use when comparing cell-type annotation quality between two representations of Visium HD data (e.g. bin2cell cells vs 8 μm bins) and you need per-cell-type tallies of median confidence and mean genes detected.

## When NOT to use

- Input is a single AnnData representation with no matched comparison object (e.g. only b2c cells or only 8 μm bins) — no comparison table can be built.
- The CellTypist model was not trained on a matching species/tissue atlas (e.g. a human atlas applied to mouse brain data) — confidence scores would not be interpretable across 113 mouse brain cell types.
- Expression matrices are raw counts without library-size normalization and log1p — gene detection counts would not be comparable across representations.

## Inputs

- bin2cell-reconstructed cell AnnData (.h5ad) from grouped 2 μm Visium HD bins
- default 8 μm SpaceRanger bin AnnData (.h5ad)
- CellTypist model trained on the Yao et al. 2023 mouse brain atlas

## Outputs

- celltypist_b2c_vs_8um_comparison.csv with per-cell-type median confidence and mean genes detected for b2c vs 8 μm bins
- tally of cell types favoring b2c per metric (expected: 90/113 confidence, 113/113 genes)

## How to apply

Load the bin2cell-reconstructed cell AnnData (built from grouped 2 μm bins via bin2cell.bin_to_cell after bin2cell.destripe correction) and the default 8 μm bin AnnData from the mouse brain Visium HD sample. Normalize each matrix with library-size normalization and log1p so gene detection is comparable. Annotate both objects with a CellTypist model trained on the Yao et al. 2023 mouse brain atlas to obtain per-cell prediction probabilities and predicted labels across the 113 recovered cell types. Group by predicted label and compute median confidence and mean number of genes detected per cell type in each representation, then build a side-by-side table and tally wins. The expected reference result for the mouse brain benchmark: 90/113 cell types have higher median confidence in b2c and 113/113 have higher mean genes detected in b2c.

## Related tools

- **bin2cell** (Produces the reconstructed cell AnnData from 2 μm bins (bin2cell.destripe, bin2cell.bin_to_cell) that is compared against 8 μm bins) — https://github.com/Teichlab/bin2cell
- **CellTypist** (Annotates both objects with per-cell prediction probabilities (confidence) and predicted labels used for per-cell-type aggregation)
- **Visium HD** (Source spatial transcriptomics platform providing 2 μm and 8 μm bin data from mouse brain)
- **Python** (Implementation environment for normalization, CellTypist annotation, and per-cell-type aggregation)
- **numpy** (Numerical aggregation of per-cell-type medians and means)

## Examples

```
import scanpy as sc, celltypist; sc.pp.normalize_total(adata); sc.pp.log1p(adata); pred = celltypist.annotate(adata, model='Yao_2023_mousebrain'); med = pred.probabilities.groupby(pred.predicted_labels).median()
```

## Evaluation signals

- Reproduced table shows higher median CellTypist confidence in b2c for 90 of 113 cell types.
- Reproduced table shows higher mean genes detected in b2c for all 113 cell types.
- Both AnnData objects are normalized (library-size + log1p) before gene counting, verified by checking matrix scale.
- Per-cell-type rows cover all 113 cell types recovered by the Yao et al. 2023 atlas model in both representations.
- Output file celltypist_b2c_vs_8um_comparison.csv contains paired median-confidence and mean-gene columns for each cell type.

## Limitations

- Confidence tally (90/113) is dataset- and model-specific; other tissues or atlases may shift which cell types favor b2c.
- numpy is only loosely capped at 1.x with no exact pin, a reproducibility gap for the numerical aggregation.
- No versioned release artifacts or dataset accessions are stated for the mouse brain benchmark objects; only the source repository is referenced.

## Evidence

- [results] b2c vs 8um confidence and gene count benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] improvements over default bins: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [intro] bin2cell grouping of 2um bins: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
- [methods] bin_to_cell function: "bin2cell.bin_to_cell"
- [readme] demo and downstream notebooks: [evidence span withheld — anchor status: external_doc]
