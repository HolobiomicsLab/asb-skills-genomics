---
name: spatial-transcriptomics-bin-to-cell-assignment
description: Use when working with 10X Visium HD 2 μm bin data plus a morphology image
  (H&E or IF) and the goal is to reconstruct per-cell transcriptomic objects instead
  of using the default 8 μm bins.
when_to_use_negative:
- Input is already aggregated at the default 8 μm Visium HD resolution or at single-cell
  resolution — reconstruction from 2 μm bins offers no benefit
- No morphology image is available for the capture area, removing the segmentation
  basis for grouping bins
- Task is custom probe design for Visium HD panels — use gene2probe instead
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: bin2cell
  role: 'Core package: scales images, runs StarDist, inserts labels, expands nuclei
    into cells, and assigns bins to cells'
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Nuclei detection on the scaled morphology image using modality-matched pretrained
    models (optional dependency group of bin2cell)
  repo: https://github.com/stardist/stardist
- name: Visium HD
  role: Data platform providing subcellular 2 μm bins with a paired reference morphology
    image
- name: spaceranger
  role: Produces the Visium HD spatial folder read by bin2cell (spaceranger_image_path
    in b2c.read_visium())
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:b8ffa4913d2a436fa16228cad15ac5d4a0efb07119f9c917e8df288a217e6612
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - StarDist
  - Visium HD
  - bin2cell
  - spaceranger
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spatial-transcriptomics-bin-to-cell-assignment

## Summary

Group 2 μm Visium HD bins into cells by combining StarDist nuclei segmentation of the morphology image with fixed-distance or volume-ratio label expansion and transcriptional bin-to-cell assignment, improving over default 8 μm bins.

## When to use

Use when working with 10X Visium HD 2 μm bin data plus a morphology image (H&E or IF) and the goal is to reconstruct per-cell transcriptomic objects instead of using the default 8 μm bins.

## When NOT to use

- Input is already aggregated at the default 8 μm Visium HD resolution or at single-cell resolution — reconstruction from 2 μm bins offers no benefit
- No morphology image is available for the capture area, removing the segmentation basis for grouping bins
- Task is custom probe design for Visium HD panels — use gene2probe instead

## Inputs

- Visium HD 2 μm binned count matrix (spaceranger spatial folder)
- High-resolution morphology image (H&E or immunofluorescence) from archived FFPE blocks
- Pretrained StarDist model appropriate for the image modality (H&E or fluorescence)

## Outputs

- Per-bin cell label assignments (bin_cell_labels.csv)
- AnnData object with reconstructed cells created from grouped 2 μm bins, carrying spatial information and sharper morphology images

## How to apply

First scale the morphology image to a target resolution of mpp 0.2–0.5 (StarDist models were trained around mpp 0.3; the demo used mpp=0.35 with prob_thresh=0.1) using bin2cell.scaled_he_image or bin2cell.scaled_if_image. Run nuclei segmentation with bin2cell.stardist using the pretrained StarDist model matched to the image modality (H&E vs fluorescence). Insert the nucleus labels into the 2 μm binned Visium HD object with bin2cell.insert_labels, then expand nuclei into full cell labels with bin2cell.expand_labels using the default fixed 4 μm (2-bin) expansion or the label-specific volume-ratio distance dL. Finally assign equidistant bins to cells by PCA transcriptional similarity with bin2cell.bin_to_cell, writing per-bin assignments (e.g. bin_cell_labels.csv). Visium HD also carries a variable-bin-dimension technical effect that bin2cell corrects, and bin2cell.destripe can address destriping before assignment.

## Related tools

- **bin2cell** (Core package: scales images, runs StarDist, inserts labels, expands nuclei into cells, and assigns bins to cells) — https://github.com/Teichlab/bin2cell
- **StarDist** (Nuclei detection on the scaled morphology image using modality-matched pretrained models (optional dependency group of bin2cell)) — https://github.com/stardist/stardist
- **Visium HD** (Data platform providing subcellular 2 μm bins with a paired reference morphology image)
- **spaceranger** (Produces the Visium HD spatial folder read by bin2cell (spaceranger_image_path in b2c.read_visium()))

## Examples

```
b2c.scaled_he_image(mpp=0.35); b2c.stardist(mpp=0.35, prob_thresh=0.1); b2c.insert_labels(); b2c.expand_labels(); b2c.bin_to_cell()
```

## Evaluation signals

- Visually inspect StarDist detections overlaid on the scaled image and adjust mpp/prob_thresh until detected objects are nuclei (no quantitative accuracy metric is reported)
- Confirm the working image resolution falls within the recommended mpp 0.2–0.5 range
- Verify expansion distance is the default 4 μm (2 bins of 2 μm) or a valid label-specific volume-ratio dL
- Check that bin_cell_labels.csv covers equidistant bins with PCA-similarity-based assignments and each bin maps to at most one cell
- Benchmark reconstructed cells against the standard 8 μm SpaceRanger output using the provided mouse brain and colorectal cancer comparison notebooks

## Limitations

- StarDist segmentation accuracy is highly dependent on input image resolution and must be visually validated; no quantitative accuracy metric is reported
- StarDist is an optional dependency and TensorFlow must be installed separately; numpy is loosely capped at 1.x, a reproducibility gap
- The alternative (volume-ratio) expansion depends on user-controlled parameters (k, max_call_distance) without default performance characterization
- No versioned releases, DOIs, or dataset accessions are stated beyond the GitHub repository

## Evidence

- [abstract] Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [methods] StarDist mpp recommendation: [evidence span withheld — anchor status: not_found]
- [methods] Demo tuning of segmentation: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [methods] Resolution dependency caveat: "The utility and performance of specific StarDist models (H&E or fluorescence) is highly dependent on the input image resolution."
- [readme] README on StarDist installation: [evidence span withheld — anchor status: external_doc]
- [readme] README on purpose: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
