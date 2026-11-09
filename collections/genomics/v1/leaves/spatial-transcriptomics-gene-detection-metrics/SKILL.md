---
name: spatial-transcriptomics-gene-detection-metrics
description: Use when comparing downstream annotation quality (CellTypist confidence)
  and per-cell gene detection between 2 μm-bin-derived reconstructed cells and default
  8 μm bins from Visium HD data.
when_to_use_negative:
- The input is raw spaceranger output that has not been processed by bin2cell (destripe,
  bin_to_cell) — no reconstructed-cell representation exists to compare against 8
  μm bins
- The tissue or CellTypist model is not mouse brain / Yao et al. 2023 atlas-derived,
  so the 113 cell types and expected 90/113, 113/113 tallies do not apply as reference
  invariants
- The goal is comparing destriped vs non-destriped expression within one representation
  rather than b2c cells vs 8 μm bins
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: bin2cell
  role: Produces the reconstructed cell objects (destripe, bin_to_cell on grouped
    2 μm bins) whose annotation confidence and gene detection are compared against
    8 μm bins
  repo: https://github.com/Teichlab/bin2cell
- name: CellTypist
  role: Annotates both b2c cells and 8 μm bins with the Yao et al. 2023 mouse brain
    atlas model, yielding per-cell prediction probabilities used for the confidence
    comparison
- name: numpy
  role: Numeric aggregation (medians/means) of per-cell confidence and gene-detection
    values per cell type
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:de404feca5dce81d554c50923d0d885fd07204b524e4b6bd43b7c17fe479343f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - CellTypist
  - Python
  - bin2cell
  - numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Spatial transcriptomics gene detection metrics

## Summary

Per-cell-type comparison of CellTypist annotation confidence and per-cell gene detection between bin2cell-reconstructed cells and default 8 μm SpaceRanger bins in mouse brain Visium HD data, quantifying how many of the 113 atlas cell types favor b2c on each metric.

## When to use

Use when comparing downstream annotation quality (CellTypist confidence) and per-cell gene detection between 2 μm-bin-derived reconstructed cells and default 8 μm bins from Visium HD data.

## When NOT to use

- The input is raw spaceranger output that has not been processed by bin2cell (destripe, bin_to_cell) — no reconstructed-cell representation exists to compare against 8 μm bins
- The tissue or CellTypist model is not mouse brain / Yao et al. 2023 atlas-derived, so the 113 cell types and expected 90/113, 113/113 tallies do not apply as reference invariants
- The goal is comparing destriped vs non-destriped expression within one representation rather than b2c cells vs 8 μm bins

## Inputs

- bin2cell-reconstructed mouse brain cell AnnData (grouped 2 μm bins after destripe correction)
- default 8 μm SpaceRanger bin AnnData from the same Visium HD section
- CellTypist model trained on the Yao et al. 2023 mouse brain atlas

## Outputs

- Per-cell-type comparison table of median CellTypist confidence and mean genes detected (b2c vs 8 μm bins), e.g. celltypist_b2c_vs_8um_comparison.csv
- Tally of cell types favoring b2c per metric (90/113 confidence, 113/113 gene detection)

## How to apply

Load the b2c cell AnnData (built via bin2cell.bin_to_cell after bin2cell.destripe on grouped 2 μm bins) and the default 8 μm bin AnnData from the mouse brain Visium HD run. Normalize each matrix with library-size normalization followed by log1p so gene detection counts are comparable across the two representations. Annotate both objects with a CellTypist model trained on the Yao et al. 2023 mouse brain atlas, capturing per-cell prediction probabilities and predicted labels across the 113 recovered cell types. Aggregate per cell type: median annotation confidence and mean genes detected per group, then build a side-by-side table and tally how many cell types favor b2c for each metric. The expected reference outcome is higher median confidence in b2c for 90 of 113 cell types and higher mean genes detected in b2c for all 113 cell types.

## Related tools

- **bin2cell** (Produces the reconstructed cell objects (destripe, bin_to_cell on grouped 2 μm bins) whose annotation confidence and gene detection are compared against 8 μm bins) — https://github.com/Teichlab/bin2cell
- **CellTypist** (Annotates both b2c cells and 8 μm bins with the Yao et al. 2023 mouse brain atlas model, yielding per-cell prediction probabilities used for the confidence comparison)
- **numpy** (Numeric aggregation (medians/means) of per-cell confidence and gene-detection values per cell type)

## Examples

```
import celltypist; pred_b2c = celltypist.annotate(adata_b2c, model=model); pred_8um = celltypist.annotate(adata_8um, model=model); df.groupby('cell_type')[['confidence','n_genes']].median()
```

## Evaluation signals

- The comparison table contains all 113 cell types from the Yao et al. 2023 model with non-null median confidence for both representations
- Reproduced tally: 90 of 113 cell types show higher median CellTypist confidence in b2c than in 8 μm bins
- Reproduced tally: all 113 of 113 cell types show higher average genes detected in b2c than in 8 μm bins
- Both AnnData objects were normalized (library-size normalization + log1p) before computing gene detection, so counts are comparable across representations
- Saved table (celltypist_b2c_vs_8um_comparison.csv) has side-by-side columns for b2c and 8 μm bins for both metrics

## Limitations

- numpy is only loosely constrained (capped at 1.x) with no exact pin, a reproducibility gap for re-running the aggregation
- The reference tallies (90/113 confidence, 113/113 gene detection) are specific to the mouse brain dataset and the Yao et al. 2023 atlas model; other tissues/models will not match them
- StarDist is an optional dependency of bin2cell, so default installs may lack the segmentation dependency needed to build the b2c objects used in this comparison
- No versioned release artifacts or dataset accessions are stated; only the source repository is referenced

## Evidence

- [abstract] Downstream-analysis improvement claim: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [readme] README framing of 2 μm vs 8 μm resolution: "Visium HD captures gene expression data at a subcellular 2um resolution. It should be possible to use this data to reconstruct cells more accurately than just using the next resolution up (8um)"
- [readme] README pointer to notebooks comparing b2c output against 8um SpaceRanger output: [evidence span withheld — anchor status: external_doc]
- [readme] README pointer to CellTypist destripe comparison notebook: [evidence span withheld — anchor status: not_found]
