---
name: label-expansion-cell-reconstruction
description: Use when Visium HD 2 μm bins must be grouped into cell objects after
  StarDist nuclei detection, via fixed-distance (4 μm default) or volume-ratio label
  expansion on a H&E or IF morphology image.
when_to_use_negative:
- Input is already standard 8 μm SpaceRanger Visium HD output with no 2 μm binning
  — reconstruction from 2 μm bins is the premise of this skill
- No morphology image (H&E or IF) is available for StarDist nuclei detection, so there
  are no nucleus labels to expand
- Goal is probe design for Visium HD panels — that is handled by the separate gene2probe
  tool, not label expansion
tools:
- name: bin2cell
  role: Provides insert_labels, expand_labels (fixed 4 μm default or volume-ratio
    dL), salvage_secondary_labels, and bin_to_cell to group 2 μm bins into reconstructed
    cells
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Pretrained H&E or fluorescence models detect nuclei on the scaled morphology
    image; the resulting labels are the seed objects that label expansion grows into
    cells
  repo: https://github.com/stardist/stardist
- name: Visium HD
  role: Supplies the subcellular 2 μm bins and paired reference morphology image that
    the reconstructed cells are built from
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:8c4cdde2a0a75b9942f46dd51628abf7de9f63bd9bc399eb64371e3619b3589f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - StarDist
  - Visium HD
  - bin2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# label-expansion-cell-reconstruction

## Summary

Expands StarDet-detected nucleus labels inserted into a 2 μm binned Visium HD object into full cell labels using bin2cell (fixed-distance expansion, default 4 μm i.e. 2 bins, or label-specific volume-ratio dL), then assigns equidistant bins to cells by transcriptional similarity.

## When to use

Use when Visium HD 2 μm bins must be grouped into cell objects after StarDist nuclei detection, via fixed-distance (4 μm default) or volume-ratio label expansion on a H&E or IF morphology image.

## When NOT to use

- Input is already standard 8 μm SpaceRanger Visium HD output with no 2 μm binning — reconstruction from 2 μm bins is the premise of this skill
- No morphology image (H&E or IF) is available for StarDist nuclei detection, so there are no nucleus labels to expand
- Goal is probe design for Visium HD panels — that is handled by the separate gene2probe tool, not label expansion

## Inputs

- Visium HD 2 μm binned count matrix (AnnData/SDuke-style object)
- Visium HD H&E or IF morphology image (e.g., from 10X demo Visium HD lung IF dataset)
- StarDist nucleus label masks from bin2cell.stardist

## Outputs

- Expanded cell label masks (beyond nucleus boundaries)
- Per-bin cell label assignments written to bin_cell_labels.csv
- Final Visium HD object with cells created from grouped 2 μm bins, carrying spatial information and sharper morphology images

## How to apply

First scale the morphology image to the target resolution with bin2cell.scaled_he_image or bin2cell.scaled_if_image, staying within the recommended mpp range of 0.2–0.5 because the StarDist models were trained on images with an mpp around 0.3. Run nuclei segmentation with bin2cell.stardist using the pretrained StarDist model matched to the image modality (H&E or fluorescence); in the 10X demo Visium HD lung IF dataset, mpp=0.35 with prob_thresh=0.1 drastically improved segmentation versus mpp=0.5. Insert the StarDist nucleus labels into the 2 μm binned object with bin2cell.insert_labels, then expand them into cell labels with bin2cell.expand_labels using either the default fixed distance of 2 bins (4 μm) or the label-specific volume-ratio distance dL; optionally use bin2cell.salvage_secondary_labels for split nuclei. Finally run bin2cell.bin_to_cell, which assigns equidistant bins to adjacent cells by PCA transcriptional similarity, producing per-bin cell label assignments written to bin_cell_labels.csv.

## Related tools

- **bin2cell** (Provides insert_labels, expand_labels (fixed 4 μm default or volume-ratio dL), salvage_secondary_labels, and bin_to_cell to group 2 μm bins into reconstructed cells) — https://github.com/Teichlab/bin2cell
- **StarDist** (Pretrained H&E or fluorescence models detect nuclei on the scaled morphology image; the resulting labels are the seed objects that label expansion grows into cells) — https://github.com/stardist/stardist
- **Visium HD** (Supplies the subcellular 2 μm bins and paired reference morphology image that the reconstructed cells are built from)

## Examples

```
import bin2cell as b2c; b2c.expand_labels(adata)
```

## Evaluation signals

- Inspect the StarDist segmentation on the scaled image and confirm the detected objects are nuclei, adjusting mpp (target ~0.3–0.35, within 0.2–0.5) and prob_thresh (e.g. 0.1) since no quantitative accuracy metric is reported
- Confirm the morphology image was rescaled to an mpp within the recommended 0.2–0.5 range before segmentation
- Verify that every expanded cell label encompasses ≥1 of the equidistant 2 μm bins and that bins equidistant from multiple nuclei are resolved by PCA transcriptional similarity in bin_to_cell
- Check bin_cell_labels.csv is produced with per-bin cell assignments covering the binned object
- Compare downstream analysis of reconstructed cells against default 8 μm bins (mouse brain / human colorectal cancer comparisons) for expected improvement

## Limitations

- No quantitative accuracy metric is reported for the nuclei segmentation; validation is visual inspection and parameter tuning
- The alternative volume-ratio expansion depends on user-controlled parameters (k, max_call_distance) without default performance characterization given
- StarDist is an optional dependency group (pip install bin2cell[stardist]) and TensorFlow must be installed separately, so default installs may lack the segmentation dependency
- numpy is loosely capped at 1.x with no exact pin, a reproducibility gap

## Evidence

- [methods] StarDist models require mpp range: [evidence span withheld — anchor status: not_found]
- [methods] mpp=0.35 improved segmentation: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [methods] StarDist model choice depends on resolution: "The utility and performance of specific StarDist models (H&E or fluorescence) is highly dependent on the input image resolution."
- [abstract] Bin2cell reconstructs cells from 2 μm bins: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [intro] End result description: "The end result is an object with cells, created from grouped 2um bins assigned to the same object after segmentation, carrying spatial information and sharper morphology images for visualisation."
- [readme] StarDist dependency note: [evidence span withheld — anchor status: external_doc]
