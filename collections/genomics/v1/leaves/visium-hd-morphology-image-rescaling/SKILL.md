---
name: visium-hd-morphology-image-rescaling
description: Use when preparing Visium HD Space Ranger output (full-resolution H&E
  or IF tissue image plus spatial folder with scale factors) for bin2cell segmentation,
  because the default tissue image resolution is unsuitable for StarDist segmentation.
when_to_use_negative:
- The input is not Visium HD Space Ranger output with a raw full-resolution morphology
  image and scale factors
- The image is already at a controlled mpp within the 0.2–0.5 range suitable for StarDist
  segmentation
- Working with a platform lacking a reference morphology image paired to transcriptomic
  data
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bin2cell
  role: Provides b2c.read_visium(), bin2cell.scaled_he_image() and bin2cell.scaled_if_image()
    to load Space Ranger output and rescale/crop the morphology image to the target
    mpp
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: 'Source platform: full-scale transcriptomic data paired with a reference morphology
    image from FFPE blocks at sub-cellular resolution'
- name: spaceranger
  role: Produces the unified spatial folder with the raw tissue image and scale factors
    consumed by b2c.read_visium()
- name: numpy
  role: Array handling for the image used in the rescaling workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:35e3e7709cb99209036b7054a2c1447fae1eccf6eb178e674573f34665cee311
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - Visium HD
  - bin2cell
  - numpy
  - spaceranger
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# visium-hd-morphology-image-rescaling

## Summary

Generates a controlled-resolution morphology reference image for bin2cell by rescaling the raw Space Ranger Visium HD tissue image to a specified microns-per-pixel (mpp) value and cropping it to the tissue capture region, providing consistent input for downstream cell segmentation and plotting.

## When to use

Use when preparing Visium HD Space Ranger output (full-resolution H&E or IF tissue image plus spatial folder with scale factors) for bin2cell segmentation, because the default tissue image resolution is unsuitable for StarDist segmentation.

## When NOT to use

- The input is not Visium HD Space Ranger output with a raw full-resolution morphology image and scale factors
- The image is already at a controlled mpp within the 0.2–0.5 range suitable for StarDist segmentation
- Working with a platform lacking a reference morphology image paired to transcriptomic data

## Inputs

- Full-resolution tissue H&E (or IF) morphology image from Space Ranger output
- Unified Space Ranger spatial folder containing scale factors (spaceranger_image_path)

## Outputs

- Rescaled, capture-region-cropped morphology image (e.g. scaled_he_image.png) at the chosen mpp with 3 color channels for H&E

## How to apply

Load the Space Ranger output via b2c.read_visium(), pointing spaceranger_image_path at the unified spatial folder (as introduced in response to 10X creating a unified spaceranger spatial folder). Run bin2cell.scaled_he_image (or bin2cell.scaled_if_image for immunofluorescence) with mpp set within the recommended 0.2–0.5 microns-per-pixel range — the StarDist models were trained on images with an mpp around 0.3, and a value of 0.5 is the upper end for H&E. Crop the rescaled image to the tissue capture region so downstream segmentation and plotting operate only on captured pixels, and save it (e.g. as scaled_he_image.png). Correct mpp choice is consequential: changing mpp from 0.5 to 0.35 drastically improved StarDist segmentation of a Visium HD lung IF image.

## Related tools

- **bin2cell** (Provides b2c.read_visium(), bin2cell.scaled_he_image() and bin2cell.scaled_if_image() to load Space Ranger output and rescale/crop the morphology image to the target mpp) — https://github.com/Teichlab/bin2cell
- **Visium HD** (Source platform: full-scale transcriptomic data paired with a reference morphology image from FFPE blocks at sub-cellular resolution)
- **spaceranger** (Produces the unified spatial folder with the raw tissue image and scale factors consumed by b2c.read_visium())
- **numpy** (Array handling for the image used in the rescaling workflow)

## Examples

```
b2c.read_visium(spaceranger_image_path='spatial'); b2c.scaled_he_image(adata, mpp=0.5)
```

## Evaluation signals

- The output file (e.g. scaled_he_image.png) exists and retains 3 color channels for H&E
- Reported image shape matches the inferred dimensions from the applied mpp scaling (e.g. checked via bin2cell.actual_vs_inferred_image_shape)
- The chosen mpp falls within the recommended 0.2–0.5 range (StarDist models trained around mpp 0.3)
- The image is limited to the tissue capture region so downstream segmentation/plotting operate only on captured pixels
- For IF images, segmentation quality improves at the tuned mpp (e.g. prob_thresh returned to 0.1 after rescaling to mpp=0.35)

## Limitations

- Resolution choice materially affects segmentation: an mpp of 0.5 drastically underperformed 0.35 on a Visium HD lung IF image until it was regenerated
- The rescaled image is limited to the capture region, so it is not a full-slide reference
- StarDist model performance is highly dependent on input image resolution

## Evidence

- [methods] StarDist models trained around 0.3 mpp: [evidence span withheld — anchor status: not_found]
- [methods] IF rescaling improved segmentation: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [methods] scaled_he_image workflow step: "bin2cell.scaled_he_image"
- [discussion] spaceranger_image_path rationale: "add `spaceranger_image_path` to `b2c.read_visium()` in response to 10X creating a unified spaceranger spatial folder"
- [abstract] Visium HD pairs morphology image: "Visium HD by 10X Genomics is the first commercially available platform capable of capturing full scale transcriptomic data paired with a reference morphology image"
