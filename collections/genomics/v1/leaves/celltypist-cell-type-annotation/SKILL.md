---
name: celltypist-cell-type-annotation
description: Use when Visium HD data exists in both b2c-reconstructed cell form (grouped
  2 μm bins) and default 8 μm bin form, and you need automated cell-type labels with
  confidence scores to quantify which representation yields better annotation.
when_to_use_negative:
- Input is already annotated single-cell scRNA-seq (e.g. the Yao et al. 2023 atlas
  itself) rather than Visium HD 2 μm/8 μm bin data — there is no b2c-vs-bin comparison
  to make
- Only one representation is available (b2c cells or 8 μm bins alone), so the paired
  confidence and gene-detection comparison cannot be computed
- Tissue has no matching CellTypist reference atlas (the demonstrated model is built
  from the Yao et al. 2023 mouse brain atlas; a human colorectal cancer dataset would
  need a different reference)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: bin2cell
  role: Produces the reconstructed-cell AnnData (bin_to_cell on 2 μm bins after destripe)
    that is one arm of the CellTypist annotation comparison
  repo: https://github.com/Teichlab/bin2cell
- name: CellTypist
  role: Automated cell-type annotation providing per-cell predicted labels and confidence
    scores using a model trained on the Yao et al. 2023 mouse brain atlas
- name: Visium HD
  role: Source platform capturing subcellular 2 μm resolution data; its default 8
    μm bins form the comparison arm
- name: Python
  role: Environment for running normalization, CellTypist annotation, and per-cell-type
    aggregation (bin2cell is compatible with established Python single-cell software)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:cde988896a6fb1dc7946bb769e265d3eaf249c6227463437a5ceba67fb9a7088
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# celltypist-cell-type-annotation

## Summary

Annotate Visium HD bin-level data (default 8 μm Space Ranger bins and bin2cell-reconstructed cells) with a CellTypist model trained on the Yao et al. 2023 mouse brain atlas, comparing per-cell-type annotation confidence and gene detection between the two representations.

## When to use

Use when Visium HD data exists in both b2c-reconstructed cell form (grouped 2 μm bins) and default 8 μm bin form, and you need automated cell-type labels with confidence scores to quantify which representation yields better annotation.

## When NOT to use

- Input is already annotated single-cell scRNA-seq (e.g. the Yao et al. 2023 atlas itself) rather than Visium HD 2 μm/8 μm bin data — there is no b2c-vs-bin comparison to make
- Only one representation is available (b2c cells or 8 μm bins alone), so the paired confidence and gene-detection comparison cannot be computed
- Tissue has no matching CellTypist reference atlas (the demonstrated model is built from the Yao et al. 2023 mouse brain atlas; a human colorectal cancer dataset would need a different reference)

## Inputs

- b2c-reconstructed cell AnnData (.h5ad) built from grouped 2 μm bins via bin2cell.bin_to_cell after bin2cell.destripe correction
- Default 8 μm bin AnnData (.h5ad) from Space Ranger Visium HD output
- CellTypist model trained on the Yao et al. 2023 mouse brain atlas

## Outputs

- Per-cell predicted cell-type labels and CellTypist confidence scores for both the b2c and 8 μm objects
- Comparison CSV (celltypist_b2c_vs_8um_comparison.csv) with per-cell-type median confidence and mean genes detected for b2c vs 8 μm bins
- Tally of cell types favoring b2c for each metric (e.g. 90/113 higher confidence, 113/113 higher gene detection)

## How to apply

Load the b2c cell AnnData (built with bin2cell.bin_to_cell on grouped 2 μm bins after bin2cell.destripe correction) and the default 8 μm bin AnnData, then normalize each matrix with library-size normalization and log1p so gene detection is comparable across representations. Annotate both objects with a CellTypist model trained on the Yao et al. 2023 mouse brain atlas to obtain per-cell predicted labels across the 113 recovered cell types and their prediction probabilities (confidence). Aggregate per cell type, computing median confidence and mean number of genes detected separately for b2c cells and 8 μm bins. Build a side-by-side comparison table and tally how many cell types favor b2c on each metric; the paper's demo notebooks (e.g. the destripe_celltypist notebook) show this CellTypist usage downstream of bin2cell. Save the table (e.g. celltypist_b2c_vs_8um_comparison.csv) for reuse in downstream analysis notebooks.

## Related tools

- **bin2cell** (Produces the reconstructed-cell AnnData (bin_to_cell on 2 μm bins after destripe) that is one arm of the CellTypist annotation comparison) — https://github.com/Teichlab/bin2cell
- **CellTypist** (Automated cell-type annotation providing per-cell predicted labels and confidence scores using a model trained on the Yao et al. 2023 mouse brain atlas)
- **Visium HD** (Source platform capturing subcellular 2 μm resolution data; its default 8 μm bins form the comparison arm)
- **Python** (Environment for running normalization, CellTypist annotation, and per-cell-type aggregation (bin2cell is compatible with established Python single-cell software))

## Examples

```
import celltypist; pred_b2c = celltypist.annotate(adata_b2c, model="Yao_2023_Mouse_Brain"); pred_8um = celltypist.annotate(adata_8um, model="Yao_2023_Mouse_Brain")
```

## Evaluation signals

- Reproduced comparison table shows higher median CellTypist confidence in b2c for 90 of 113 cell types, matching the reported result
- All 113 cell types show higher average number of genes detected in b2c than in 8 μm bins
- Both objects were library-size normalized and log1p-transformed before annotation, so gene detection is comparable across representations
- Both representations are annotated with the same CellTypist model (Yao et al. 2023 mouse brain atlas), recovering 113 cell types in each
- Output CSV (celltypist_b2c_vs_8um_comparison.csv) contains per-cell-type rows with median confidence and mean genes detected for both arms

## Limitations

- Confidence comparison depends on destriping: the repository provides a dedicated notebook comparing destriped and non-destriped expression data with CellTypist, indicating the technical correction affects annotation outcomes
- The 90/113 confidence result is atlas- and dataset-specific (mouse brain Visium HD with the Yao et al. 2023 atlas); it may not transfer to other tissues or references
- Comparing median confidence across representations with different cell/bin sizes can be sensitive to how bins group into cells; raw unnormalized counts must not be fed directly to CellTypist

## Evidence

- [abstract] b2c reconstruction basis: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [abstract] improvement over 8 μm bins: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [intro] destripe correction matters: "The package also corrects for a novel technical effect in the data stemming from variable bin dimensions."
- [readme] CellTypist demo notebook: [evidence span withheld — anchor status: external_doc]
- [intro] bin grouping objective: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
