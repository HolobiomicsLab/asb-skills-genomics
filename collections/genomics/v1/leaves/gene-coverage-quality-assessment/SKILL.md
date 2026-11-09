---
name: gene-coverage-quality-assessment
description: Use when comparing bin2cell-reconstructed cell objects against default
  8 μm Visium HD bins and you need a quantitative per-object quality signal such as
  per-cell gene count coverage or CellTypist prediction confidence.
when_to_use_negative:
- Data are 10X demo Visium HD lung IF images or mouse brain rather than the human
  colorectal cancer comparison benchmark (use the species/organ-specific comparison
  notebooks instead)
- Input is already cell-level data (e.g. scRNA-seq or FACS-derived cells) with no
  2 μm bins to group — there is no bin aggregation to evaluate
- The morphology image lacks the ~0.2-0.5 mpp resolution range needed for StarDist,
  since reconstruction quality and hence coverage gains depend on usable segmentation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: bin2cell
  role: Reconstructs cell objects from 2 μm bins (destripe, insert_labels, expand_labels,
    bin_to_cell, salvage_secondary_labels) that are the treatment arm of the coverage
    comparison
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Provides morphology-image segmentation labels inserted into the Visium HD
    object before bin_to_cell grouping
  repo: https://github.com/Teichlab/bin2cell
- name: CellTypist
  role: Annotates both the reconstructed-cell and 8 μm-bin objects; prediction confidence
    is recorded alongside gene coverage as a downstream quality metric
  repo: https://bin2cell.readthedocs.io/en/latest/notebooks/destripe_celltypist.html
- name: Visium HD
  role: Source platform providing 2 μm subcellular-resolution transcriptomic bins
    paired with a reference morphology image
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:088bea28cc122ae1c6208cf0ae65bf9d1eb09a1e4a1ba09f5ca8f904c6cac634
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
yes_rate: 0.6
n_claims: 10
---

# gene-coverage-quality-assessment

## Summary

Assess per-cell gene coverage (number of detected genes per cell/bin object) as a quality metric to judge whether bin2cell-reconstructed cells outperform default 8 μm Space Ranger bins on Visium HD human colorectal cancer data, alongside CellTypist confidence and CMS2 subtype calls.

## When to use

Use when comparing bin2cell-reconstructed cell objects against default 8 μm Visium HD bins and you need a quantitative per-object quality signal such as per-cell gene count coverage or CellTypist prediction confidence.

## When NOT to use

- Data are 10X demo Visium HD lung IF images or mouse brain rather than the human colorectal cancer comparison benchmark (use the species/organ-specific comparison notebooks instead)
- Input is already cell-level data (e.g. scRNA-seq or FACS-derived cells) with no 2 μm bins to group — there is no bin aggregation to evaluate
- The morphology image lacks the ~0.2-0.5 mpp resolution range needed for StarDist, since reconstruction quality and hence coverage gains depend on usable segmentation

## Inputs

- 10X demo Visium HD human colorectal cancer dataset (2 μm bin counts + reference morphology H&E image)
- Default 8 μm Space Ranger bin object
- StarDist segmentation labels derived from the morphology image
- CellTypist annotation outputs (prediction confidence per object)

## Outputs

- celltypist_confidence_comparison.csv with per-representation CellTypist confidence and per-cell gene coverage
- Per-cell gene coverage metrics for the bin2cell-reconstructed object versus 8 μm bins
- CMS2 subtype call per representation (correct for reconstructed cells, errant for 8 μm bins)

## How to apply

Load the 10X demo Visium HD human colorectal cancer data (2 μm bin counts plus the reference morphology image) and apply bin2cell.destripe to correct the technical effect stemming from variable bin dimensions. Build the reconstructed cell object with bin2cell.bin_to_cell using StarDist labels inserted via bin2cell.insert_labels and expanded via bin2cell.expand_labels, keeping the default 8 μm bin object as the comparator. Recover transcripts from bins filtered out during segmentation with bin2cell.salvage_secondary_labels so coverage comparisons are not biased by dropped bins. Annotate both objects with CellTypist following notebooks/destripe_celltypist.ipynb, then record CellTypist prediction confidence and per-cell gene coverage for each representation and write the comparison to celltypist_confidence_comparison.csv.

## Related tools

- **bin2cell** (Reconstructs cell objects from 2 μm bins (destripe, insert_labels, expand_labels, bin_to_cell, salvage_secondary_labels) that are the treatment arm of the coverage comparison) — https://github.com/Teichlab/bin2cell
- **StarDist** (Provides morphology-image segmentation labels inserted into the Visium HD object before bin_to_cell grouping) — https://github.com/Teichlab/bin2cell
- **CellTypist** (Annotates both the reconstructed-cell and 8 μm-bin objects; prediction confidence is recorded alongside gene coverage as a downstream quality metric) — https://bin2cell.readthedocs.io/en/latest/notebooks/destripe_celltypist.html
- **Visium HD** (Source platform providing 2 μm subcellular-resolution transcriptomic bins paired with a reference morphology image)

## Examples

```
import bin2cell as b2c; adata = b2c.bin_to_cell(adata_2um); b2c.salvage_secondary_labels(adata)
```

## Evaluation signals

- The bin2cell-reconstructed object shows higher per-cell gene coverage than the default 8 μm bin object
- The bin2cell-reconstructed object shows higher CellTypist prediction confidence than the 8 μm bins
- The reconstructed-cell grouping yields a correct CMS2 subtype prediction where the 8 μm bins produce errant predictions
- celltypist_confidence_comparison.csv contains per-representation rows for both the reconstructed and 8 μm bin objects
- Visual inspection confirms the reconstructed cells recapitulate fine morphological structure that is errantly aggregated by 8 μm bins

## Limitations

- numpy is capped loosely at 1.x with no exact pin, which is a reproducibility gap for the analysis environment
- StarDist is an optional dependency group (pip install bin2cell[stardist]) and TensorFlow must be installed separately, so default installs may fail at the segmentation step
- Confidence improvements over 8 μm bins are described as modest, not dramatic
- No versioned release artifacts, DOIs, or dataset accessions are stated; only the source repository is referenced

## Evidence

- [abstract] Reconstructed cells improve downstream analysis over default 8 μm bins on mouse brain and human colorectal cancer data: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [intro] Object carries spatial information and sharper morphology images after grouping 2 μm bins: "The end result is an object with cells, created from grouped 2um bins assigned to the same object after segmentation, carrying spatial information and sharper morphology images for visualisation."
- [methods] StarDist mpp guidance for segmentation quality: [evidence span withheld — anchor status: not_found]
- [readme] StarDist as optional dependency; install may lack segmentation support: [evidence span withheld — anchor status: external_doc]
- [readme] CRC comparison notebooks available in repository: [evidence span withheld — anchor status: external_doc]
