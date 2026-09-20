---
name: spaceranger-spatial-output-handling
description: Use when starting from raw 10X Visium HD Space Ranger output (tissue
  H&E image + spatial folder with scale factors) and a controlled-resolution morphology
  image is needed for downstream bin2cell segmentation.
when_to_use_negative:
- The Space Ranger-supplied tissue image is not directly suitable for segmentation
  — do not feed it into StarDist without rescaling to a controlled mpp first
- Input is an IF rather than H&E morphology image — use bin2cell.scaled_if_image instead
  of scaled_he_image
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bin2cell
  role: Provides b2c.read_visium() with spaceranger_image_path and b2c.scaled_he_image()/scaled_if_image()
    to rescale and crop the Space Ranger morphology image to the target mpp
  repo: https://github.com/Teichlab/bin2cell
- name: spaceranger
  role: Source of the Visium HD spatial output (full-resolution tissue image and unified
    spatial folder with scale factors) consumed by this skill
- name: Visium HD
  role: 10X Genomics platform capturing full-scale transcriptomic data paired with
    the reference morphology image handled by this skill
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:b5aaf7ff97ae474592256eafbffce961c438e357a5fa09d1723d796d2c20abf6
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
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# spaceranger-spatial-output-handling

## Summary

Load and preprocess 10X Space Ranger Visium HD spatial outputs (full-resolution H&E morphology image plus the unified spatial folder with scale factors) via bin2cell's b2c.read_visium(), then rescale and crop to a controlled-resolution (e.g. mpp=0.5) capture-region reference image for segmentation and plotting.

## When to use

Use when starting from raw 10X Visium HD Space Ranger output (tissue H&E image + spatial folder with scale factors) and a controlled-resolution morphology image is needed for downstream bin2cell segmentation.

## When NOT to use

- The Space Ranger-supplied tissue image is not directly suitable for segmentation — do not feed it into StarDist without rescaling to a controlled mpp first
- Input is an IF rather than H&E morphology image — use bin2cell.scaled_if_image instead of scaled_he_image

## Inputs

- 10X Visium HD Space Ranger output: full-resolution tissue H&E (or IF) morphology image
- Space Ranger unified 'spatial' folder with scale factor files (spaceranger_image_path)

## Outputs

- Controlled-resolution capture-region morphology image, rescaled to the chosen mpp (e.g. scaled_he_image.png), retaining 3 color channels for H&E

## How to apply

Call b2c.read_visium() pointing spaceranger_image_path at the unified Space Ranger spatial folder (a parameter added in response to 10X creating this unified folder structure). The stock Space Ranger tissue image is unsuitable for segmentation, so run bin2cell.scaled_he_image with mpp set within the recommended 0.2-0.5 microns-per-pixel range (StarDist models were trained around mpp 0.3; 0.5 is the upper end) to rescale the raw tissue image. Crop the rescaled image to the tissue capture region so plotting and segmentation operate only on captured pixels, and save it (e.g. scaled_he_image.png). For IF (immunofluorescence) inputs use the analogous bin2cell.scaled_if_image stage instead.

## Related tools

- **bin2cell** (Provides b2c.read_visium() with spaceranger_image_path and b2c.scaled_he_image()/scaled_if_image() to rescale and crop the Space Ranger morphology image to the target mpp) — https://github.com/Teichlab/bin2cell
- **spaceranger** (Source of the Visium HD spatial output (full-resolution tissue image and unified spatial folder with scale factors) consumed by this skill)
- **Visium HD** (10X Genomics platform capturing full-scale transcriptomic data paired with the reference morphology image handled by this skill)

## Examples

```
import bin2cell as b2c; adata = b2c.read_visium('spaceranger_output/', spaceranger_image_path='spaceranger_output/spatial/'); b2c.scaled_he_image(adata, mpp=0.5, save_path='scaled_he_image.png')
```

## Evaluation signals

- Output image file (e.g. scaled_he_image.png) exists on disk
- Reported image shape matches dimensions inferred from the applied mpp scaling, e.g. verified via bin2cell.actual_vs_inferred_image_shape
- H&E output retains 3 color channels
- Image extent is limited to the tissue capture region (no off-capture pixels remain)
- Chosen mpp falls within the recommended 0.2-0.5 microns-per-pixel range

## Limitations

- StarDist model performance is highly dependent on input resolution; the wrong mpp degrades segmentation (e.g. moving from mpp 0.5 to 0.35 drastically improved segmentation of a Visium HD lung IF image)
- b2c.read_visium()'s spaceranger_image_path parameter exists specifically because 10X changed to a unified spaceranger spatial folder; older Space Ranger output layouts may need adjustment
- StarDist is an optional dependency group (pip install bin2cell[stardist]) plus TensorFlow, so a default bin2cell install may lack segmentation dependencies

## Evidence

- [discussion] add `spaceranger_image_path` to `b2c.read_visium()` in response to 10X creating a unified spaceranger spatial folder: "add `spaceranger_image_path` to `b2c.read_visium()` in response to 10X creating a unified spaceranger spatial folder"
- [methods] we would recommend using the ``mpp`` (microns per pixel) range of 0.2-0.5: "we would recommend using the ``mpp`` (microns per pixel) range of 0.2-0.5"
- [methods] The StarDist models were trained on images with an ``mpp`` around 0.3.: "The StarDist models were trained on images with an ``mpp`` around 0.3."
- [methods] bin2cell.scaled_he_image: "bin2cell.scaled_he_image"
- [abstract] Visium HD by 10X Genomics is the first commercially available platform capable of capturing full scale transcriptomic data paired with a reference morphology image: "Visium HD by 10X Genomics is the first commercially available platform capable of capturing full scale transcriptomic data paired with a reference morphology image"
