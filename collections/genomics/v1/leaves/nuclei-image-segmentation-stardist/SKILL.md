---
name: nuclei-image-segmentation-stardist
description: Use when reconstructing single cells from Visium HD 2 μm bins and a scaled
  morphology image (mpp 0.2–0.5) is available as the basis for nuclei detection.
when_to_use_negative:
- The morphology image has not been rescaled to a StarDist-appropriate mpp (models
  were trained at mpp ~0.3), e.g. running on the raw Space Ranger tissue image directly
- The data are standard 8 μm Space Ranger bins rather than subcellular 2 μm Visium
  HD bins, since there is no morphology-guided grouping to perform
- StarDist/TensorFlow is unavailable and the optional bin2cell[stardist] dependency
  group was not installed
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: bin2cell
  role: Provides stardist, insert_labels and expand_labels functions that run nuclei
    segmentation and map labels onto the 2 μm bin grid
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Pretrained H&E and fluorescence nucleus segmentation models whose performance
    depends strongly on input image mpp
  repo: https://github.com/stardist/stardist
- name: Visium HD
  role: Source platform supplying the 2 μm bins and paired reference morphology image
    being segmented
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:4ca87794073c728f8f7e0b754382664be7a6ed68cdd5a9d4d1522a21e9c6effe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - StarDist
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nuclei-image-segmentation-stardist

## Summary

Segments nuclei on a scaled Visium HD morphology image (H&E or IF) using pretrained StarDist models via bin2cell.stardist, producing nucleus labels that are subsequently expanded into cell labels for 2 μm bin grouping.

## When to use

Use when reconstructing single cells from Visium HD 2 μm bins and a scaled morphology image (mpp 0.2–0.5) is available as the basis for nuclei detection.

## When NOT to use

- The morphology image has not been rescaled to a StarDist-appropriate mpp (models were trained at mpp ~0.3), e.g. running on the raw Space Ranger tissue image directly
- The data are standard 8 μm Space Ranger bins rather than subcellular 2 μm Visium HD bins, since there is no morphology-guided grouping to perform
- StarDist/TensorFlow is unavailable and the optional bin2cell[stardist] dependency group was not installed

## Inputs

- Scaled morphology image (H&E or IF) at mpp 0.2–0.5 from bin2cell.scaled_he_image / scaled_if_image
- Visium HD 2 μm binned AnnData object (from b2c.read_visium)
- Pretrained StarDist model matched to the image modality

## Outputs

- StarDist nucleus label image aligned to the scaled morphology image
- Nucleus labels inserted into the 2 μm binned object (via bin2cell.insert_labels)
- Expanded cell labels (via bin2cell.expand_labels) used downstream by bin2cell.bin_to_cell

## How to apply

First generate a controlled-resolution reference image with bin2cell.scaled_he_image or bin2cell.scaled_if_image, choosing an mpp in the recommended 0.2–0.5 range since the StarDist models were trained at mpp ~0.3; e.g. mpp=0.35 with prob_thresh=0.1 drastically improved segmentation of the 10X lung IF demo. Run bin2cell.stardist with the pretrained StarDist model matched to the image modality (H&E or fluorescence), tuning prob_thresh between the overly stringent StarDist default and overly lenient values — prob_thresh=0.05 was the reported sweet spot at mpp 0.3 on the human colorectal cancer H&E data. Insert the resulting nucleus labels onto the 2 μm bin grid with bin2cell.insert_labels, then expand them into cell labels with bin2cell.expand_labels using the default fixed 4 μm expansion (2 bins) or label-specific volume-ratio expansion dL. Note StarDist (and TensorFlow) must be installed via the bin2cell[stardist] dependency group since StarDist is an optional dependency.

## Related tools

- **bin2cell** (Provides stardist, insert_labels and expand_labels functions that run nuclei segmentation and map labels onto the 2 μm bin grid) — https://github.com/Teichlab/bin2cell
- **StarDist** (Pretrained H&E and fluorescence nucleus segmentation models whose performance depends strongly on input image mpp) — https://github.com/stardist/stardist
- **Visium HD** (Source platform supplying the 2 μm bins and paired reference morphology image being segmented)

## Examples

```
import bin2cell as b2c; b2c.stardist(mpp=0.35, prob_thresh=0.1)
```

## Evaluation signals

- Inspect the segmentation overlay on the scaled image and adjust mpp/prob_thresh until detected objects are clearly nuclei (no quantitative accuracy metric is reported)
- Scaled image mpp falls within the recommended 0.2–0.5 range and shape matches inferred dimensions (bin2cell.actual_vs_inferred_image_shape), retaining 3 channels for H&E
- Inserted labels align nucleus boundaries to the 2 μm bin grid, and expanded cell labels cover equidistant bins assignable by bin_to_cell
- Sensitivity to prob_thresh behaves as documented: StarDist default is overly stringent, ~0.01 overly lenient, 0.05–0.1 the sweet spot at mpp 0.3–0.35

## Limitations

- Segmentation quality is highly dependent on input image resolution; mpp=0.5 performed poorly on the lung IF image until regenerated at mpp=0.35
- No quantitative segmentation accuracy metric is reported — validation is visual inspection and parameter tuning
- StarDist is an optional dependency group and needs TensorFlow installed separately; numpy is only loosely capped at 1.x

## Evidence

- [methods] The utility and performance of specific StarDist models (H&E or fluorescence) is highly dependent on the input image resolution.: "The utility and performance of specific StarDist models (H&E or fluorescence) is highly dependent on the input image resolution."
- [methods] we would recommend using the ``mpp`` (microns per pixel) range of 0.2-0.5: "we would recommend using the ``mpp`` (microns per pixel) range of 0.2-0.5"
- [methods] A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved.: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [readme] For `b2c.stardist()` to work, [StarDist](https://github.com/stardist/stardist) needs to be installed.: [evidence span withheld — anchor status: external_doc]
- [other] The nuclei labels are discretized into 2 μm bins and expanded into cells by label expansion of a fixed distance, defaulting to 2 bins (i.e. 4 μm).: [evidence span withheld — anchor status: not_found]
