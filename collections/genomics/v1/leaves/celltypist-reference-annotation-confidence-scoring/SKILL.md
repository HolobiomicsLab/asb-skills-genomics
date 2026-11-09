---
name: celltypist-reference-annotation-confidence-scoring
description: Use when comparing two representations of the same Visium HD capture
  (e.g. bin2cell-reconstructed cells vs 8 μm bins) and you need a reference-based
  cell type label plus a per-object confidence score to judge which representation
  better recapitulates biology.
when_to_use_negative:
- Input is already cell-level annotated data (no 2 μm bin vs cell reconstruction comparison
  is at stake) — confidence scoring adds no comparison value
- The dataset is not Visium HD or lacks a reference morphology image, so bin2cell
  reconstruction and the bin-vs-cell comparison underlying the confidence metric are
  not applicable
- The analysis targets the mouse brain demo or probe design (gene2probe) rather than
  the colorectal cancer bin-representation comparison
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellTypist
  role: Reference-based cell type annotation of both the reconstructed-cell and 8
    μm bin objects; its prediction confidence is recorded as the comparison metric
- name: bin2cell
  role: Builds the reconstructed cell comparator (destripe, insert_labels, expand_labels,
    bin_to_cell, salvage_secondary_labels) whose annotations are scored against 8
    μm bins
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Segments the morphology image to produce the labels inserted into the bin
    object before CellTypist annotation
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: Source platform providing 2 μm subcellular-resolution transcriptomic data
    paired with a reference morphology image from FFPE blocks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:9409beb0396e0eeb4ab5663f2386997821a3e0f1606f5f1d03be2e9dc9516cd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - CellTypist
  - StarDist
  - Visium HD
  - bin2cell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# celltypist-reference-annotation-confidence-scoring

## Summary

Annotate Visium HD objects (bin2cell reconstructed cells versus default 8 μm bins) with CellTypist reference-based cell type labels and use prediction confidence as a quantitative metric of representation quality. Confidence differences, together with per-cell gene coverage and CMS2 subtype calls, discriminate reconstructed cells from coarse bins on human colorectal cancer data.

## When to use

Use when comparing two representations of the same Visium HD capture (e.g. bin2cell-reconstructed cells vs 8 μm bins) and you need a reference-based cell type label plus a per-object confidence score to judge which representation better recapitulates biology.

## When NOT to use

- Input is already cell-level annotated data (no 2 μm bin vs cell reconstruction comparison is at stake) — confidence scoring adds no comparison value
- The dataset is not Visium HD or lacks a reference morphology image, so bin2cell reconstruction and the bin-vs-cell comparison underlying the confidence metric are not applicable
- The analysis targets the mouse brain demo or probe design (gene2probe) rather than the colorectal cancer bin-representation comparison

## Inputs

- 10X demo Visium HD human colorectal cancer data (2 μm bin counts plus reference morphology image)
- bin2cell-reconstructed cell AnnData object (post destripe, insert_labels, expand_labels, salvage_secondary_labels)
- Default 8 μm SpaceRanger bin AnnData object (comparator)
- CellTypist reference model

## Outputs

- celltypist_confidence_comparison.csv with per-representation CellTypist confidence and per-cell gene coverage
- CMS2 subtype prediction per representation (correct for reconstructed cells, errant for 8 μm bins)
- Visual inspection of fine morphological structure of reconstructed cells vs 8 μm bin aggregation

## How to apply

Follow the demo notebook notebooks/destripe_celltypist.ipynb: first load the 10X demo Visium HD colorectal cancer data (2 μm bin counts plus the reference morphology image) and run bin2cell.destripe to correct the technical effect from variable bin dimensions, then build the reconstructed cell object with bin2cell.bin_to_cell after bin2cell.insert_labels and bin2cell.expand_labels on StarDist-derived labels, keeping the default 8 μm bin object as the comparator. Recover transcripts from bins filtered out during segmentation with bin2cell.salvage_secondary_labels. Annotate both objects with CellTypist, record the prediction confidence and per-cell gene coverage for each representation, and record the CMS2 subtype call per representation. Validate that the reconstructed cells show higher CellTypist confidence and gene coverage and that the 8 μm bins yield errant predictions where reconstructed cells give the correct CMS2 call.

## Related tools

- **CellTypist** (Reference-based cell type annotation of both the reconstructed-cell and 8 μm bin objects; its prediction confidence is recorded as the comparison metric)
- **bin2cell** (Builds the reconstructed cell comparator (destripe, insert_labels, expand_labels, bin_to_cell, salvage_secondary_labels) whose annotations are scored against 8 μm bins) — https://github.com/Teichlab/bin2cell
- **StarDist** (Segments the morphology image to produce the labels inserted into the bin object before CellTypist annotation) — https://github.com/Teichlab/bin2cell
- **Visium HD** (Source platform providing 2 μm subcellular-resolution transcriptomic data paired with a reference morphology image from FFPE blocks)

## Examples

```
import bin2cell as b2c, celltypist; b2c.destripe(adata); pred = celltypist.annotate(adata, model=model); # then record pred.confidence + per-cell gene coverage for reconstructed vs 8um bins
```

## Evaluation signals

- Reconstructed cell object must show higher CellTypist prediction confidence than the default 8 μm bins
- Reconstructed cells must show higher per-cell gene count coverage than the 8 μm bins
- Bin grouping from reconstructed cells must yield the correct CMS2 subtype prediction while 8 μm bins produce errant predictions
- Fine morphological structure of reconstructed cells must visually recapitulate the reference morphology image, unlike the errant 8 μm bin aggregation
- Comparison table celltypist_confidence_comparison.csv must contain per-representation confidence and gene coverage rows for both objects

## Limitations

- CellTypist confidence improvements from bin2cell reconstruction are described as modest, not dramatic
- StarDist is an optional dependency group, so a default pip install bin2cell may lack the segmentation dependency needed before annotation
- No versioned release, DOI-tagged artifact, or dataset accession is stated for the comparison notebooks; only the GitHub repository is referenced
- The destripe correction addresses a novel technical effect from variable bin dimensions; skipping it invalidates the destriped-vs-nondestriped comparison in notebooks/destripe_celltypist.ipynb

## Evidence

- [abstract] Reconstructed cells improve downstream analysis over default 8 μm bins: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [abstract] Bin2cell reconstruction basis: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [readme] README comparison notebooks: [evidence span withheld — anchor status: external_doc]
- [readme] README destripe/CellTypist notebook: [evidence span withheld — anchor status: not_found]
