---
name: morphology-image-mpp-scaling
description: Use when a Visium HD 2 μm bin object is paired with a high-resolution
  H&E or IF morphology image whose native pixel resolution does not match the mpp
  (~0.3) at which StarDist models were trained.
when_to_use_negative:
- You are working only with default 8 μm bin output and have no subcellular 2 μm data
  plus paired morphology image to reconstruct cells from.
- The morphology image is already at the target mpp (≈0.3, within 0.2–0.5) matched
  to the StarDist model — rescaling again is unnecessary.
tools:
- name: bin2cell
  role: Provides scaled_he_image and scaled_if_image to rescale the morphology image
    to the target mpp before segmentation.
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Downstream consumer of the scaled image; its pretrained H&E/IF models dictate
    the required mpp because they were trained at mpp around 0.3.
  repo: https://github.com/Teichlab/bin2cell
- name: Visium HD
  role: Source platform whose morphology images are paired with the 2 μm binned transcript
    data being rescaled.
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:588146334d0ce4950eed2f7b31839b417205518fff99826aea2b66ea0dc5ab59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - Python
  - StarDist
  - Visium HD
  - bin2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# morphology-image-mpp-scaling

## Summary

Rescales the Visium HD morphology image (H&E or immunofluorescence) to a target microns-per-pixel (mpp) resolution with bin2cell.scaled_he_image / bin2cell.scaled_if_image so that StarDist nuclei segmentation operates at the mpp the pretrained models were trained on (~0.3).

## When to use

Use when a Visium HD 2 μm bin object is paired with a high-resolution H&E or IF morphology image whose native pixel resolution does not match the mpp (~0.3) at which StarDist models were trained.

## When NOT to use

- You are working only with default 8 μm bin output and have no subcellular 2 μm data plus paired morphology image to reconstruct cells from.
- The morphology image is already at the target mpp (≈0.3, within 0.2–0.5) matched to the StarDist model — rescaling again is unnecessary.

## Inputs

- Visium HD 2 μm binned AnnData object with spatial coordinates
- High-resolution H&E morphology image (for scaled_he_image)
- Immunofluorescence (IF) morphology image (for scaled_if_image)
- Target mpp value within the 0.2–0.5 range (≈0.3 recommended)

## Outputs

- Morphology image rescaled to the chosen mpp, suitable as direct input to bin2cell.stardist nuclei segmentation

## How to apply

Generate a scaled image with bin2cell.scaled_he_image for H&E or bin2cell.scaled_if_image for immunofluorescence, staying within the recommended mpp range of 0.2–0.5. Because the StarDist models were trained on images at mpp around 0.3, target ~0.3 (e.g. mpp=0.35 in the 10X lung IF demo) rather than the overly coarse 0.5. Treat mpp and StarDist prob_thresh as coupled decision points: in the lung IF demo, moving from mpp=0.5 to mpp=0.35 while resetting prob_thresh to 0.1 drastically improved segmentation. Validate by inspecting the scaled image and confirming the detected objects are nuclei, adjusting mpp/prob_thresh if they are not.

## Related tools

- **bin2cell** (Provides scaled_he_image and scaled_if_image to rescale the morphology image to the target mpp before segmentation.) — https://github.com/Teichlab/bin2cell
- **StarDist** (Downstream consumer of the scaled image; its pretrained H&E/IF models dictate the required mpp because they were trained at mpp around 0.3.) — https://github.com/Teichlab/bin2cell
- **Visium HD** (Source platform whose morphology images are paired with the 2 μm binned transcript data being rescaled.)

## Examples

```
import bin2cell as b2c; scaled_img = b2c.scaled_he_image(binned, mpp=0.3, save_path='scaled_he.png')
```

## Evaluation signals

- The chosen mpp falls within the recommended 0.2–0.5 range, ideally ~0.3 to match StarDist training resolution.
- Visual inspection of segmentation on the scaled image shows the detected objects are nuclei (no quantitative accuracy metric is reported).
- Segmentation quality improves relative to a coarser mpp: in the demo, switching from mpp=0.5 to mpp=0.35 with prob_thresh=0.1 drastically improved the lung IF segmentation.
- prob_thresh is co-tuned with mpp (e.g. prob_thresh=0.1 at mpp=0.35 for the IF image; 0.05 reported as a sweet spot for H&E at mpp 0.3).

## Limitations

- The default coarse resolution (mpp 0.5) drastically degraded StarDist segmentation on the Visium HD lung IF image, so mpp must be actively chosen, not defaulted.
- StarDist performance is highly dependent on input image resolution because the pretrained models were trained at mpp ~0.3.
- No quantitative accuracy metric is available for validating segmentation; validation is by visual inspection and manual tuning of mpp/prob_thresh.

## Evidence

- [methods] Recommended mpp range and StarDist training resolution: [evidence span withheld — anchor status: not_found]
- [methods] Effect of mpp choice on IF segmentation: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
