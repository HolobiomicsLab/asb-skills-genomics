---
name: crc-cms-subtype-classification
description: Use when comparing downstream annotation quality of Visium HD cell representations
  (e.g. bin2cell reconstructed cells versus 8 μm SpaceRanger bins) on human colorectal
  cancer data, especially for CellTypist confidence, gene coverage, and CMS subtype
  prediction.
when_to_use_negative:
- Data are not human colorectal cancer Visium HD or an equivalent morphology-image-paired
  high-resolution spatial dataset, so CMS2 subtype predictions are not meaningful.
- The goal is cell reconstruction itself (destripe/segmentation/bin_to_cell) rather
  than downstream annotation comparison against 8 μm bins.
- Only a single cell representation is available with no 8 μm comparator, so the confidence/coverage/CMS2
  comparison cannot be made.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3170
tools:
- name: bin2cell
  role: Destripes 2 μm bins, groups bins into reconstructed cells (bin_to_cell with
    inserted/expanded StarDist labels), salvages filtered-out bins, and produces the
    cell object annotated downstream
  repo: https://github.com/Teichlab/bin2cell
- name: CellTypist
  role: Annotates both the reconstructed cell object and the 8 μm bin object; prediction
    confidence and CMS2 subtype calls are the comparison metrics
- name: Visium HD
  role: Source platform providing 2 μm bin transcriptomic data paired with a reference
    morphology image from FFPE human colorectal cancer
- name: StarDist
  role: Segments the morphology image (recommended mpp range 0.2–0.5, models trained
    around mpp 0.3) to supply labels for bin2cell.insert_labels and expand_labels
  repo: https://github.com/Teichlab/bin2cell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:b038a15a3c1944aa0739b8a096c2e1c6c7dcab3cb38d37e4d0559c13cc049d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - CellTypist
  - StarDist
  - Visium HD
  - bin2cell
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# crc-cms-subtype-classification

## Summary

Annotate Visium HD human colorectal cancer representations (bin2cell reconstructed cells vs default 8 μm bins) with CellTypist following the notebooks/destripe_celltypist.ipynb demo, then compare prediction confidence, per-cell gene coverage, and consensus molecular subtype (CMS2) calls to judge whether reconstructed cells outperform 8 μm bins downstream.

## When to use

Use when comparing downstream annotation quality of Visium HD cell representations (e.g. bin2cell reconstructed cells versus 8 μm SpaceRanger bins) on human colorectal cancer data, especially for CellTypist confidence, gene coverage, and CMS subtype prediction.

## When NOT to use

- Data are not human colorectal cancer Visium HD or an equivalent morphology-image-paired high-resolution spatial dataset, so CMS2 subtype predictions are not meaningful.
- The goal is cell reconstruction itself (destripe/segmentation/bin_to_cell) rather than downstream annotation comparison against 8 μm bins.
- Only a single cell representation is available with no 8 μm comparator, so the confidence/coverage/CMS2 comparison cannot be made.

## Inputs

- 10X demo Visium HD human colorectal cancer dataset (2 μm bin counts plus reference morphology image)
- Default 8 μm SpaceRanger bin object (comparator)
- StarDist-derived cell segmentation labels on the morphology image
- CellTypist models as used in notebooks/destripe_celltypist.ipynb

## Outputs

- celltypist_confidence_comparison.csv with per-representation CellTypist prediction confidence and per-cell gene coverage
- CMS2 subtype call per representation (correct for reconstructed cells, errant for 8 μm bins)
- Visual comparison of fine morphological structure of reconstructed cells versus 8 μm bin aggregation

## How to apply

Load the 10X demo Visium HD human colorectal cancer data (2 μm bin counts plus reference morphology image) and correct the technical effect from variable bin dimensions with bin2cell.destripe. Build the reconstructed cell object with bin2cell.bin_to_cell using StarDist-derived labels inserted via bin2cell.insert_labels and expanded via bin2cell.expand_labels, retaining the default 8 μm bin object as the comparator. Recover transcripts from bins filtered out during segmentation using bin2cell.salvage_secondary_labels on the reconstructed object. Annotate both objects with CellTypist following notebooks/destripe_celltypist.ipynb, record CellTypist prediction confidence and per-cell gene coverage per representation, and visually inspect fine morphological structure of reconstructed cells against the errant 8 μm bin aggregation. Write the per-representation comparison, including the CMS2 call (correct for reconstructed cells versus errant for 8 μm bins), to celltypist_confidence_comparison.csv.

## Related tools

- **bin2cell** (Destripes 2 μm bins, groups bins into reconstructed cells (bin_to_cell with inserted/expanded StarDist labels), salvages filtered-out bins, and produces the cell object annotated downstream) — https://github.com/Teichlab/bin2cell
- **CellTypist** (Annotates both the reconstructed cell object and the 8 μm bin object; prediction confidence and CMS2 subtype calls are the comparison metrics)
- **Visium HD** (Source platform providing 2 μm bin transcriptomic data paired with a reference morphology image from FFPE human colorectal cancer)
- **StarDist** (Segments the morphology image (recommended mpp range 0.2–0.5, models trained around mpp 0.3) to supply labels for bin2cell.insert_labels and expand_labels) — https://github.com/Teichlab/bin2cell

## Examples

```
import bin2cell as b2c; cdata = b2c.bin_to_cell(bdata, labels=stardist_labels); b2c.salvage_secondary_labels(cdata); # then annotate with CellTypist per notebooks/destripe_celltypist.ipynb and write celltypist_confidence_comparison.csv
```

## Evaluation signals

- Reconstructed cell object shows higher CellTypist prediction confidence than the 8 μm bins.
- Reconstructed cell object shows higher per-cell gene count coverage than the 8 μm bins.
- Reconstructed-cell bin grouping yields a correct CMS2 prediction where 8 μm bins give errant predictions.
- Visual inspection confirms reconstructed cells recapitulate fine morphological structure that 8 μm bin aggregation obscures.
- Per-representation results are recorded in celltypist_confidence_comparison.csv with both representations present.

## Limitations

- numpy is only loosely constrained (capped at 1.x) with no exact pin, a reproducibility gap for rerunning the comparison.
- StarDist is an optional dependency group (pip install bin2cell[stardist]) plus TensorFlow, so default installs may lack segmentation dependencies.
- Findings were demonstrated on 10X demo datasets (mouse brain and human colorectal cancer); no versioned release artifacts, DOIs, or dataset accessions are stated, only the source repository.
- Alternative expansion behavior depends on user-controlled parameters without default performance characterization given.

## Evidence

- [abstract] We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data: "We demonstrate improvements in downstream analysis when using the reconstructed cells over default 8 μm bins on mouse brain and human colorectal cancer data"
- [methods] Salvage step is part of the documented workflow: "bin2cell.salvage_secondary_labels"
- [readme] Notebook for destripe/CellTypist comparison is in the repo docs: [evidence span withheld — anchor status: external_doc]
- [methods] Recommended mpp range for StarDist segmentation on morphology images: [evidence span withheld — anchor status: not_found]
