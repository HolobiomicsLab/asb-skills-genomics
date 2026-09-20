---
name: he-image-cropping-to-capture-region
description: Use when preparing a Visium HD H&E morphology image for bin2cell segmentation
  or plotting and the raw Space Ranger tissue image needs rescaling to a target mpp
  and cropping to captured pixels only.
when_to_use_negative:
- The morphology image is immunofluorescence (IF) rather than H&E — use bin2cell.scaled_if_image
  instead
- The image is already at the target mpp and limited to the capture region, e.g. an
  existing bin2cell-scaled output
- The default Space Ranger tissue image is sufficient for non-segmentation visualisation
  and no controlled resolution is required
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bin2cell
  role: Provides b2c.read_visium() to load Space Ranger output, scaled_he_image to
    rescale/crop the H&E image, and actual_vs_inferred_image_shape for validation
  repo: https://github.com/Teichlab/bin2cell
- name: spaceranger
  role: Produces the unified spatial folder and full-resolution tissue H&E image consumed
    by b2c.read_visium()
- name: StarDist
  role: Downstream segmentation consumer whose model performance depends on the input
    image resolution set at this stage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:e1ef98a9fd67aaf9ad3772328a9f40e96f9902ab284a430b938aaa58bc7f8884
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# H&E image cropping to capture region

## Summary

Rescales the raw Space Ranger full-resolution tissue H&E image to a controlled microns-per-pixel resolution and crops it to the Visium HD tissue capture region, providing consistent morphology input for bin2cell segmentation and plotting.

## When to use

Use when preparing a Visium HD H&E morphology image for bin2cell segmentation or plotting and the raw Space Ranger tissue image needs rescaling to a target mpp and cropping to captured pixels only.

## When NOT to use

- The morphology image is immunofluorescence (IF) rather than H&E — use bin2cell.scaled_if_image instead
- The image is already at the target mpp and limited to the capture region, e.g. an existing bin2cell-scaled output
- The default Space Ranger tissue image is sufficient for non-segmentation visualisation and no controlled resolution is required

## Inputs

- Space Ranger output for a 10X Visium HD dataset: full-resolution tissue H&E image
- Unified Space Ranger spatial folder with scale factors
- Target resolution in microns per pixel (recommended mpp range 0.2-0.5)

## Outputs

- scaled_he_image.png — H&E image rescaled to the target mpp and cropped to the capture region, retaining 3 color channels

## How to apply

Load the Space Ranger output (full-resolution tissue H&E image plus the unified spatial folder with scale factors) via b2c.read_visium(), pointing spaceranger_image_path at the spatial folder. Run bin2cell.scaled_he_image with mpp=0.5 — the upper end of the recommended 0.2-0.5 microns-per-pixel range, chosen because the StarDist models were trained on images with an mpp around 0.3 — to rescale the raw tissue image. Crop the rescaled image to the tissue capture region so downstream segmentation and plotting operate only on captured pixels. Save the result as scaled_he_image.png and validate via bin2cell.actual_vs_inferred_image_shape.

## Related tools

- **bin2cell** (Provides b2c.read_visium() to load Space Ranger output, scaled_he_image to rescale/crop the H&E image, and actual_vs_inferred_image_shape for validation) — https://github.com/Teichlab/bin2cell
- **spaceranger** (Produces the unified spatial folder and full-resolution tissue H&E image consumed by b2c.read_visium())
- **StarDist** (Downstream segmentation consumer whose model performance depends on the input image resolution set at this stage)

## Examples

```
import bin2cell as b2c; adata = b2c.read_visium(path='spaceranger_output', spaceranger_image_path='spaceranger_output/spatial'); b2c.scaled_he_image(adata, mpp=0.5, save_path='scaled_he_image.png')
```

## Evaluation signals

- The output file scaled_he_image.png exists on disk
- Reported shape matches the inferred dimensions from the applied mpp scaling, checked via bin2cell.actual_vs_inferred_image_shape
- The output image retains 3 color channels for H&E
- The chosen mpp lies within the recommended 0.2-0.5 range (e.g. mpp=0.5)
- Image extent corresponds to the tissue capture region only, not the full slide

## Limitations

- StarDist segmentation performance is highly dependent on input image resolution; an unsuitable mpp degrades segmentation (e.g. changing mpp from 0.5 to 0.35 drastically improved segmentation of a Visium HD lung IF image)
- StarDist is an optional dependency group for bin2cell, so default installs may lack the segmentation model
- Raw Space Ranger tissue images are unsuitable for direct segmentation, motivating the controlled-resolution custom reference image

## Evidence

- [methods] StarDist models trained at ~0.3 mpp: [evidence span withheld — anchor status: not_found]
- [methods] scaled_he_image workflow step: "bin2cell.scaled_he_image"
- [discussion] unified spatial folder handling: "add `spaceranger_image_path` to `b2c.read_visium()` in response to 10X creating a unified spaceranger spatial folder"
- [methods] StarDist resolution dependence: "The utility and performance of specific StarDist models (H&E or fluorescence) is highly dependent on the input image resolution."
- [readme] demo notebook reference: [evidence span withheld — anchor status: not_found]
