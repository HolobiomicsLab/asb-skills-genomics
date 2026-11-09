---
name: ome-tiff-whole-slide-image-io
description: Use when performing pixel-level normalization on whole-slide MTI data
  stored as OME-TIFF stacks, where cell/nucleus segmentation masks must restrict analysis
  to in-cell pixels and output must remain uint16-compatible.
when_to_use_negative:
- Input is already a single-cell feature table (cell x marker matrix); UniFORM's feature-level
  workflow does not need OME-TIFF stack IO.
- Data are mIHC chromogenic images; linear normalization factors are unsuitable because
  enzyme-catalyzed colorimetric amplification produces non-linear signal intensities.
- Normalization targets extracellular (non-cell) regions; UniFORM is currently optimized
  only for within-cell normalization via segmentation masks.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: UniFORM
  role: Python pipeline that reads the OME-TIFF whole-slide stack and segmentation
    masks with Dask and writes the normalized pixel stack as normalized_pixel_stack.ome.tiff
  repo: https://github.com/kunlunW/UniFORM
- name: Python
  role: Implementation language for the Dask-backed OME-TIFF image reading/writing
    environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:c34d3a7b6ba8f3d5a498826268d5956de64966dd1662e1eb3d412d15df543e15
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - Python
  - UniFORM
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ome-tiff-whole-slide-image-io

## Summary

Dask-backed reading of OME-TIFF whole-slide multiplex immunofluorescence stacks with nucleus/cell segmentation masks, and writing of the normalized intensity stack back as an OME-TIFF file, as used in UniFORM's pixel-level normalization pipeline.

## When to use

Use when performing pixel-level normalization on whole-slide MTI data stored as OME-TIFF stacks, where cell/nucleus segmentation masks must restrict analysis to in-cell pixels and output must remain uint16-compatible.

## When NOT to use

- Input is already a single-cell feature table (cell x marker matrix); UniFORM's feature-level workflow does not need OME-TIFF stack IO.
- Data are mIHC chromogenic images; linear normalization factors are unsuitable because enzyme-catalyzed colorimetric amplification produces non-linear signal intensities.
- Normalization targets extracellular (non-cell) regions; UniFORM is currently optimized only for within-cell normalization via segmentation masks.

## Inputs

- OME-TIFF whole-slide multiplex tissue imaging (MTI) image stack (uint16)
- Nucleus/cell segmentation masks corresponding to the image stack

## Outputs

- normalized_pixel_stack.ome.tiff (normalized image stack written as OME-TIFF)

## How to apply

Clone the UniFORM repository and use Dask-backed image reading to load the OME-TIFF whole-slide image stack together with the nucleus/cell segmentation masks. Restrict pixel-level histogram calculation exclusively to pixels inside nuclear or cellular masks so non-cell background does not distort the log-transformed per-marker intensity distributions. After normalization factors are estimated via maximum cross-correlation functional registration (scikit-fda), apply the stretching factor derived from the most-compressed sample before rescaling to 16 bits, preventing uint16 range-compression artifacts when a normalization factor <1 shifts intensities left. Write the result as an OME-TIFF file named normalized_pixel_stack.ome.tiff.

## Related tools

- **UniFORM** (Python pipeline that reads the OME-TIFF whole-slide stack and segmentation masks with Dask and writes the normalized pixel stack as normalized_pixel_stack.ome.tiff) — https://github.com/kunlunW/UniFORM
- **Python** (Implementation language for the Dask-backed OME-TIFF image reading/writing environment)

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git
```

## Evaluation signals

- The normalized OME-TIFF stack (normalized_pixel_stack.ome.tiff) exists and is readable as an OME-TIFF file.
- All pixel values in the normalized stack stay within the uint16 dynamic range.
- No rounding-spike artifacts appear at low intensities after rescaling to 16-bit (verified by inspecting the low-intensity end of per-marker histograms).
- Histogram statistics computed on the normalized stack are restricted to masked (nuclear/cellular) pixels, excluding background regions.
- Per-marker intensity distribution shapes are preserved relative to the pre-normalization distributions (stretching restores intensity ranges without reshaping distributions).

## Limitations

- Effectiveness depends on the quality of the MTI dataset and the cell/nuclei segmentation masks; compromised signals cannot be recovered.
- The Pixel-Level (OME-TIFF) example dataset and pixel-level normalization tutorial are not yet available (being assembled).
- A linear stretching factor is required to avoid uint16 range-compression artifacts when normalization factors <1 shift intensity distributions left; the pipeline is linear and thus unsuitable for mIHC.

## Evidence
