---
name: visium-hd-bin-count-destriping
description: Use when working with raw 2 μm bin count matrices from Visium HD (e.g.,
  the 10X demo lung dataset) that show a striped per-bin count-total appearance caused
  by variable bin dimensions, prior to segmentation or cell reconstruction.
when_to_use_negative:
- Input is already an 8 μm bin matrix or a reconstructed cell-by-gene object — destriping
  targets the 2 μm bin-level striping artifact only
- The count matrix shows no striped per-bin total structure from variable bin dimensions;
  applying quantile rescaling would alter counts without benefit
- The goal is cell segmentation or bin-to-cell grouping (use bin2cell.stardist / bin2cell.bin_to_cell
  instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bin2cell
  role: Provides the destripe function that divides counts by combined row/column
    0.99-quantile factors and rescales the count matrix
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: Source platform producing the 2 μm bin count matrix with the variable-bin-dimension
    striping effect
- name: numpy
  role: Array operations used to divide the count matrix by factors and rescale it;
    version capped at 1.x
  repo: https://github.com/Teichlab/bin2cell
- name: Python
  role: Runtime environment compatible with established Python single cell and spatial
    transcriptomics software (scanpy/AnnData)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:42474117abc1a9e21322cf9395b7b4b3a526d2d89b62276aa471ea551dafc5ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# visium-hd-bin-count-destriping

## Summary

Corrects the striped count-total artifact in 2 μm Visium HD bins that arises from variable bin dimensions, by dividing each bin's counts by per-array-row and per-array-column 0.99 quantile factors (bin2cell.destripe) and rescaling the count matrix so corrected per-bin totals are preserved.

## When to use

Use when working with raw 2 μm bin count matrices from Visium HD (e.g., the 10X demo lung dataset) that show a striped per-bin count-total appearance caused by variable bin dimensions, prior to segmentation or cell reconstruction.

## When NOT to use

- Input is already an 8 μm bin matrix or a reconstructed cell-by-gene object — destriping targets the 2 μm bin-level striping artifact only
- The count matrix shows no striped per-bin total structure from variable bin dimensions; applying quantile rescaling would alter counts without benefit
- The goal is cell segmentation or bin-to-cell grouping (use bin2cell.stardist / bin2cell.bin_to_cell instead)

## Inputs

- Raw 2 μm Visium HD bin-by-gene count matrix (spaceranger output, loaded into an AnnData with scanpy)
- Quantile parameter (default 0.99) for row and column factor computation

## Outputs

- Destriped bin-by-gene AnnData count matrix with corrected per-bin count totals (e.g., destriped_counts.h5ad)
- Per-row and per-column quantile-derived scaling factors rescaled into count space

## How to apply

Load the raw 2 μm bin count matrix into an AnnData object with scanpy. Compute, for each bin, the 0.99 quantile (user-specified, default 0.99) of counts across genes within its array row and rescale the factor into count space; repeat the same computation per array column. Divide the count matrix by the combined row/column factors and rescale the matrix so the corrected per-bin count totals are preserved, using bin2cell.destripe (numpy array operations). Save the result as destriped_counts.h5ad. The rationale is that the novel technical effect from variable bin dimensions inflates count totals in systematic row/column stripes, and row/column quantile normalization removes this structure without altering gene identities.

## Related tools

- **bin2cell** (Provides the destripe function that divides counts by combined row/column 0.99-quantile factors and rescales the count matrix) — https://github.com/Teichlab/bin2cell
- **Visium HD** (Source platform producing the 2 μm bin count matrix with the variable-bin-dimension striping effect)
- **numpy** (Array operations used to divide the count matrix by factors and rescale it; version capped at 1.x) — https://github.com/Teichlab/bin2cell
- **Python** (Runtime environment compatible with established Python single cell and spatial transcriptomics software (scanpy/AnnData))

## Examples

```
import bin2cell as b2c; adata = b2c.destripe(adata, quantile=0.99); adata.write('destriped_counts.h5ad')
```

## Evaluation signals

- The corrected matrix retains the original bins-by-genes shape and gene identifiers
- Applied per-row and per-column quantile factors equal 0.99-quantile-derived values rescaled into count space
- Per-bin count totals after correction match the rescaled corrected totals (count totals preserved after rescaling)
- Visual inspection of per-bin count totals shows the striped row/column structure is removed
- Output file (e.g., destriped_counts.h5ad) loads back as a valid AnnData with same var_names as the input

## Limitations

- numpy is only loosely constrained (capped at 1.x), with no exact version pin, which is a reproducibility gap
- The quantile is user-specified (default 0.99); no default performance characterization is given for alternative quantile choices
- Only the source repository is referenced; no versioned release artifacts, DOIs, or dataset accessions are stated

## Evidence

- [intro] Novel technical effect correction objective: "The package also corrects for a novel technical effect in the data stemming from variable bin dimensions."
- [readme] README correction statement: "The package also corrects for a novel technical effect in the data stemming from variable bin dimensions."
- [discussion] numpy cap caveat: "cap `numpy` at 1.x for the time being"
- [methods] demo dataset context: "One of the 10X demo Visium HD datasets has an IF image of the lung."
