---
name: pca-transcriptional-similarity-bin-assignment
description: Use when a binned Visium HD AnnData object carries nucleus/cell labels
  (from bin2cell.insert_labels and bin2cell.expand_labels) and each 2 μm bin must
  be assigned to its final cell, with ambiguous equidistant bins resolved by gene-expression
  similarity.
when_to_use_negative:
- Input is already an 8 μm SpaceRanger cell-level object — this skill exists specifically
  to reconstruct cells from subcellular 2 μm bins, which SpaceRanger default output
  bypasses
- Nucleus labels have not yet been segmented (StarDist) and expanded (bin2cell.expand_labels);
  running bin_to_cell on raw bins has no segmentation basis to group them
- No gene expression matrix is available for the bins — the equidistant-bin resolution
  relies on PCA transcriptional similarity and cannot fall back to it
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: bin2cell
  role: Provides bin_to_cell, which assigns each 2 μm bin to an expanded cell label
    and resolves equidistant bins via PCA transcriptional similarity
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Upstream nucleus segmentation whose labels are inserted and expanded before
    bin assignment; requires the optional dependency group (pip install bin2cell[stardist])
  repo: https://github.com/stardist/stardist
- name: Visium HD
  role: Source platform providing subcellular 2 μm bin expression data paired with
    a reference morphology image
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:37c0a356568a914ce943c5689f06f65667eed34894a44b41bf5a82c18a023b4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - StarDist
  - Visium HD
  - bin2cell
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# PCA transcriptional similarity bin-to-cell assignment

## Summary

Final grouping step of the bin2cell pipeline in which 2 μm Visium HD bins are assigned to reconstructed cell labels produced by StarDist segmentation and label expansion, using PCA transcriptional similarity to resolve equidistant bins.

## When to use

Use when a binned Visium HD AnnData object carries nucleus/cell labels (from bin2cell.insert_labels and bin2cell.expand_labels) and each 2 μm bin must be assigned to its final cell, with ambiguous equidistant bins resolved by gene-expression similarity.

## When NOT to use

- Input is already an 8 μm SpaceRanger cell-level object — this skill exists specifically to reconstruct cells from subcellular 2 μm bins, which SpaceRanger default output bypasses
- Nucleus labels have not yet been segmented (StarDist) and expanded (bin2cell.expand_labels); running bin_to_cell on raw bins has no segmentation basis to group them
- No gene expression matrix is available for the bins — the equidistant-bin resolution relies on PCA transcriptional similarity and cannot fall back to it

## Inputs

- AnnData object of 2 μm Visium HD bins with StarDist nucleus labels inserted and expanded into cell labels (bin2cell.insert_labels + bin2cell.expand_labels output)
- Scaled morphology image used for segmentation (bin2cell.scaled_he_image or bin2cell.scaled_if_image output, mpp 0.2–0.5)

## Outputs

- Per-bin cell label assignments written to bin_cell_labels.csv
- AnnData object with cells created from grouped 2 μm bins carrying spatial information and per-cell gene expression

## How to apply

After running bin2cell.stardist on the scaled morphology image (mpp 0.2–0.5, e.g. mpp=0.35 with prob_thresh=0.1) and expanding nucleus labels with bin2cell.expand_labels (fixed 4 μm expansion or label-specific volume-ratio dL), run bin2cell.bin_to_cell on the 2 μm binned AnnData. Each bin is matched to the nearest expanded cell label; bins equidistant between two candidate cells — e.g. along cell borders where the 4 μm expansion from adjacent nuclei meets — are disambiguated by PCA transcriptional similarity, assigning the bin to the cell whose expression profile is most similar. Write the per-bin cell label assignments to bin_cell_labels.csv. Rationale: the morphology-based expansion alone cannot resolve shared-border bins, so expression information provides the tie-breaking signal, yielding an object where grouped bins share spatial information and per-cell expression.

## Related tools

- **bin2cell** (Provides bin_to_cell, which assigns each 2 μm bin to an expanded cell label and resolves equidistant bins via PCA transcriptional similarity) — https://github.com/Teichlab/bin2cell
- **StarDist** (Upstream nucleus segmentation whose labels are inserted and expanded before bin assignment; requires the optional dependency group (pip install bin2cell[stardist])) — https://github.com/stardist/stardist
- **Visium HD** (Source platform providing subcellular 2 μm bin expression data paired with a reference morphology image)

## Examples

```
import bin2cell as b2c; b2c.bin_to_cell(adata, labels='labels')
```

## Evaluation signals

- Every 2 μm bin that falls within exactly one expanded cell label region is assigned to that cell; only equidistant (shared-border) bins are assigned by transcriptional similarity
- bin_cell_labels.csv exists and contains one cell-label entry per input bin, with no bins from inside a single cell's expansion assigned to a different cell
- The resulting cell object carries spatial information and grouped-bin expression consistent with the counts of the underlying 2 μm bins
- Segmentation-derived output can be compared against default 8 μm SpaceRanger output (mouse brain and human colorectal cancer demo notebooks) to confirm downstream analysis improvements
- Inspect segmentation on the scaled image and adjust mpp/prob_thresh to confirm detected objects are nuclei, since no quantitative accuracy metric is reported

## Limitations

- No quantitative segmentation accuracy metric is reported; validation is limited to visual inspection of detected objects on the scaled image with manual mpp/prob_thresh tuning
- StarDist is an optional dependency group (bin2cell[stardist] plus TensorFlow), so a default pip install of bin2cell lacks the segmentation backend
- Alternative expansion/assignment behavior depends on user-controlled parameters (k, max_call_distance) without default performance characterization given
- The pipeline corrects for a novel technical effect stemming from variable bin dimensions; skipping destripe/bin correction propagates error into the bin-to-cell assignment

## Evidence

- [methods] PCA similarity resolves equidistant bins: [evidence span withheld — anchor status: not_found]
- [intro] Overall objective of grouping bins after segmentation: "The end result is an object with cells, created from grouped 2um bins assigned to the same object after segmentation, carrying spatial information"
- [abstract] Bin2cell leverages segmentation and expression on 2 μm bins: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [readme] README statement of bin-based reconstruction premise: "Visium HD captures gene expression data at a subcellular 2um resolution. It should be possible to use this data to reconstruct cells more accurately"
