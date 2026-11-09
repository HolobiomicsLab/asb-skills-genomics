---
name: incomplete-membrane-extension-cell-separation
description: Use when segmenting single cells in multiplexed imaging (e.g., IMC/MIBI/immunofluorescence)
  where membrane marker predictions are incomplete and nuclear-dilation or Voronoi
  approaches undersegment elongated epithelial cells.
when_to_use_negative:
- Input is already a per-cell feature/quantification table with no image data to segment
- Cells are small and round with complete membrane staining where simpler methods
  (nuclear dilation, Voronoi) would suffice
- The image modality or tissue type falls outside the multiplexed imaging scope MIRIAM
  was designed for and no membrane/nuclear markers exist to train the pixel classifier
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Pipeline providing the novel membrane-extension and internal-membrane cell-separation
    routines; implemented in Matlab and Python
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Python implementation of MIRIAM used for the MIBI breast carcinoma generalization
    run
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Matlab implementation of MIRIAM; README instructs installing Ilastik for some
    tasks and following the bundled tutorial
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random-forest pixel classification of nuclei, membranes, and intracellular
    structures feeding the separation routine
  repo: https://www.ilastik.org/download.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:0fcd5956bcd554657282875ca276a1ff4de758f7ed9c092a6cd5de738d219a66
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  repo_url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - Ilastik
  - MIRIAM
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# incomplete-membrane-extension-cell-separation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM's membrane-extension and internal-membrane cell-separation routine extends incomplete cell membrane predictions to delineate single cells, including columnar epithelial cells with internal membranes, in multiplexed tissue imaging data.

## When to use

Use when segmenting single cells in multiplexed imaging (e.g., IMC/MIBI/immunofluorescence) where membrane marker predictions are incomplete and nuclear-dilation or Voronoi approaches undersegment elongated epithelial cells.

## When NOT to use

- Input is already a per-cell feature/quantification table with no image data to segment
- Cells are small and round with complete membrane staining where simpler methods (nuclear dilation, Voronoi) would suffice
- The image modality or tissue type falls outside the multiplexed imaging scope MIRIAM was designed for and no membrane/nuclear markers exist to train the pixel classifier

## Inputs

- Multiplexed image stack (e.g., MIBI breast carcinoma TMA, 41 patients, 36 markers, 2048x2048 pixels at 0.5 um/pixel; or multiplexed immunofluorescence colorectal adenoma TMA)
- Random-forest/Ilastik pixel-classification probabilities for nuclei, membrane, and other intracellular structures
- Optionally, manually annotated ground-truth segmentation masks for validation

## Outputs

- Single-cell segmentation masks with cells separated despite incomplete membranes
- Per-cell marker intensity table quantified over entire cell and nucleus, membrane, and cytoplasm compartments (e.g., miriam_mibi_results.csv)

## How to apply

First run random forest-based pixel classification (Ilastik per the README) over the entire marker stack — using all markers, e.g. the 36 markers of the MIBI breast carcinoma TMA — to define nuclei, membranes, and other intracellular compartments, since using all markers yields more complete membranes. Then run the MIRIAM membrane-extension routine to fill gaps in the predicted membrane mask, followed by the internal-membrane cell-separation step that identifies and separates cells containing internal membranes (e.g., columnar epithelial cells oriented toward the lumen). Output single-cell masks and quantify each marker's intensity per cell over the whole cell and in the nucleus, membrane, and cytoplasm compartments. Validate by comparing against manual annotations using image-level Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC); on the manually annotated colorectal adenoma TMA cores, MIRIAM achieved DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08, outperforming Voronoi (DSC 0.32) and Mesmer (DSC 0.59).

## Related tools

- **MIRIAM** (Pipeline providing the novel membrane-extension and internal-membrane cell-separation routines; implemented in Matlab and Python) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Python implementation of MIRIAM used for the MIBI breast carcinoma generalization run) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Matlab implementation of MIRIAM; README instructs installing Ilastik for some tasks and following the bundled tutorial) — https://github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random-forest pixel classification of nuclei, membranes, and intracellular structures feeding the separation routine) — https://www.ilastik.org/download.html

## Examples

```
git clone https://github.com/Coffey-Lab/MIRIAM && python MIRIAM.py --input mibi_breast_tma_stack --markers 36 --compartments nucleus,membrane,cytoplasm --output miriam_mibi_results.csv
```

## Evaluation signals

- Image-level DSC against manual annotations falls near 0.65 ± 0.08 and JSC near 0.48 ± 0.08 (colorectal adenoma benchmark); should exceed Voronoi (DSC 0.32 ± 0.11) and Mesmer (DSC 0.59 ± 0.10)
- Segmentation masks show complete cell boundaries for columnar epithelial cells, avoiding the lumen-ward undersegmentation observed with Voronoi
- Per-cell quantification table contains intensity values per marker for each compartment (nucleus, membrane, cytoplasm) with one row per segmented cell
- Matlab and Python implementations produce largely similar segmentation results (Supplemental Figure 6)
- Example-data files regenerate all intermediate outputs except the large image stacks, allowing diff checks against the Dropbox example data bundle

## Limitations

- Requires Ilastik for some tasks; image stacks are not shipped with example data (too large) and must be regenerated/downloaded separately
- Segmentation quality can be limited by incomplete membranes and lower image quality (e.g., MIBI breast carcinoma data relative to colorectal adenoma data)
- Quantitative performance on the MIBI breast carcinoma set is demonstrated only via Supplemental Figure 7 with no accession ID or public availability statement for that dataset

## Evidence
