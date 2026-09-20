---
name: microns-per-pixel-image-scale-conversion
description: Use when a Visium HD full-resolution tissue image from Space Ranger must
  be converted to a fixed microns-per-pixel scale (e.g. mpp=0.5) before StarDist segmentation
  or downstream bin2cell plotting.
when_to_use_negative:
- The morphology image is already at the desired mpp or has been rescaled by a prior
  bin2cell.scaled_he_image/scaled_if_image call — rescaling again would degrade or
  distort it
- The goal is expression-level processing (destriping, bin-to-cell grouping) rather
  than preparing a segmentation input — use bin2cell.destripe or bin2cell.bin_to_cell
  instead
- Working with non-Visium-HD imagery whose physical pixel size is unknown, since mpp
  cannot be set correctly without Space Ranger scale factors
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bin2cell
  role: Provides b2c.read_visium(), bin2cell.scaled_he_image() and bin2cell.scaled_if_image()
    to rescale and crop the Space Ranger morphology image to a target mpp
  repo: https://github.com/Teichlab/bin2cell
- name: spaceranger
  role: Source of the full-resolution tissue image, unified spatial folder, and scale
    factors consumed by b2c.read_visium()
- name: StarDist
  role: Downstream segmentation consumer whose models are resolution-dependent, motivating
    the mpp normalization
  repo: https://github.com/stardist/stardist
- name: numpy
  role: Array backend for the rescaled image arrays produced by the scaling step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:2b623616f23450ffe465011cc4a6e8e2294343064b5b99b368ebc941bd7d3e40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - StarDist
  - bin2cell
  - numpy
  - spaceranger
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# microns-per-pixel-image-scale-conversion

## Summary

Rescales a Visium HD Space Ranger morphology image (H&E or IF) to a controlled microns-per-pixel (mpp) resolution using bin2cell's scaled_he_image/scaled_if_image, producing a consistent-resolution reference image for segmentation. This is needed because StarDist segmentation performance is highly dependent on input image resolution.

## When to use

Use when a Visium HD full-resolution tissue image from Space Ranger must be converted to a fixed microns-per-pixel scale (e.g. mpp=0.5) before StarDist segmentation or downstream bin2cell plotting.

## When NOT to use

- The morphology image is already at the desired mpp or has been rescaled by a prior bin2cell.scaled_he_image/scaled_if_image call — rescaling again would degrade or distort it
- The goal is expression-level processing (destriping, bin-to-cell grouping) rather than preparing a segmentation input — use bin2cell.destripe or bin2cell.bin_to_cell instead
- Working with non-Visium-HD imagery whose physical pixel size is unknown, since mpp cannot be set correctly without Space Ranger scale factors

## Inputs

- Full-resolution Visium HD tissue H&E (or IF) morphology image from Space Ranger output
- Space Ranger unified spatial folder with scale factors (passed as spaceranger_image_path to b2c.read_visium())
- Target microns-per-pixel value within the recommended 0.2-0.5 range

## Outputs

- Rescaled morphology image at the chosen mpp (e.g. scaled_he_image.png), cropped to the tissue capture region with 3 color channels for H&E
- Verified image dimensions matching the inferred shape from the applied mpp scaling (checkable via bin2cell.actual_vs_inferred_image_shape)

## How to apply

Load the Space Ranger output via b2c.read_visium(), pointing spaceranger_image_path at the unified spatial folder containing the full-resolution tissue H&E image and scale factors. Run bin2cell.scaled_he_image (or bin2cell.scaled_if_image for immunofluorescence) with an mpp in the recommended 0.2-0.5 microns-per-pixel range, since the StarDist models were trained on images with an mpp around 0.3; e.g. for a 10X Visium HD mouse brain demo, mpp=0.5 is used, and for a lung IF image changing mpp from 0.5 to 0.35 (with prob_thresh=0.1) drastically improved segmentation. Crop the rescaled image to the tissue capture region so downstream segmentation and plotting operate only on captured pixels, and save it (e.g. as scaled_he_image.png). Rationale: the raw Space Ranger tissue image is uncontrolled in resolution, so converting to a defined mpp provides consistent input for StarDist and for bin2cell plotting.

## Related tools

- **bin2cell** (Provides b2c.read_visium(), bin2cell.scaled_he_image() and bin2cell.scaled_if_image() to rescale and crop the Space Ranger morphology image to a target mpp) — https://github.com/Teichlab/bin2cell
- **spaceranger** (Source of the full-resolution tissue image, unified spatial folder, and scale factors consumed by b2c.read_visium())
- **StarDist** (Downstream segmentation consumer whose models are resolution-dependent, motivating the mpp normalization) — https://github.com/stardist/stardist
- **numpy** (Array backend for the rescaled image arrays produced by the scaling step)

## Examples

```
b2c.read_visium(spaceranger_image_path='spatial/'); b2c.scaled_he_image(mpp=0.5, save_path='scaled_he_image.png')
```

## Evaluation signals

- The output file (e.g. scaled_he_image.png) exists and retains 3 color channels for H&E
- Reported image shape matches dimensions inferred from the applied mpp scaling, e.g. via bin2cell.actual_vs_inferred_image_shape
- Chosen mpp falls within the recommended 0.2-0.5 microns-per-pixel range (models trained at ~0.3)
- The image covers only the tissue capture region — no pixels outside the captured area remain for downstream segmentation/plotting
- For IF images, changing mpp (e.g. 0.5 → 0.35) should visibly change StarDist segmentation quality; a drastic improvement indicates the resolution was previously mismatched

## Limitations

- StarDist segmentation quality is highly dependent on input resolution; a wrong mpp (e.g. 0.5 instead of 0.35 for one lung IF image) can drastically worsen segmentation unless prob_thresh is also retuned
- StarDist is an optional dependency of bin2cell and TensorFlow must be installed separately for segmentation to run
- No exact version pins are given (numpy only capped at 1.x), so exact reproducibility of the rescaling pipeline is not guaranteed

## Evidence

- [methods] StarDist models trained around mpp 0.3 with recommended range: [evidence span withheld — anchor status: not_found]
- [methods] Changing mpp improved IF segmentation: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [methods] Resolution-dependent segmentation rationale: "The utility and performance of specific StarDist models (H&E or fluorescence) is highly dependent on the input image resolution."
- [methods] Scaled image workflow functions: "bin2cell.scaled_he_image"
- [readme] README pointing to demo notebook and package purpose: "Bin2cell proposes 2um bin to cell groupings based on segmentation, which can be done on the morphology image and/or a visualisation of the gene expression."
