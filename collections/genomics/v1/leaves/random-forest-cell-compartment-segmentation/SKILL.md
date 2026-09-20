---
name: random-forest-cell-compartment-segmentation
description: Use when segmenting single cells from multiplexed imaging data (e.g.,
  MIBI or multiplexed immunofluorescence tissue images such as colorectal adenoma
  or breast carcinoma TMA stacks) where membranes are incomplete and per-cell compartment
  intensities are needed.
when_to_use_negative:
- Input is already a per-cell feature/quantification table and no image segmentation
  is required
- Images lack multiple markers or compartmental structure (e.g., single-channel images
  without membrane staining), where the full-marker random-forest compartment classification
  cannot be trained
- A fully automated deep-learning-only segmentation is preferred and training annotations/labels
  for the pixel classifier cannot be provided
tools:
- name: MIRIAM
  role: Pipeline that consumes random-forest compartment classification, extends incomplete
    cell membranes, separates cells with internal membranes, and quantifies markers
    per compartment
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Required for some pixel-classification tasks in the MIRIAM pipeline per the
    README
  repo: https://www.ilastik.org/download.html
- name: Python
  role: Implementation of MIRIAM used to run the segmentation and quantification (largely
    similar results to Matlab)
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Implementation of MIRIAM used to run the segmentation and quantification
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:9f3a8319ccf843cf1061e640401d338933da058eb236d6fc6d6480df475901ca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - Ilastik
  - MIRIAM
  - Matlab
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# random-forest-cell-compartment-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a random-forest-based pixel classifier on all markers of multiplexed tissue images to define nuclei, membranes, and other intracellular compartments, then use those compartment masks to drive MIRIAM's internal-membrane cell separation and per-compartment marker quantification.

## When to use

Use when segmenting single cells from multiplexed imaging data (e.g., MIBI or multiplexed immunofluorescence tissue images such as colorectal adenoma or breast carcinoma TMA stacks) where membranes are incomplete and per-cell compartment intensities are needed.

## When NOT to use

- Input is already a per-cell feature/quantification table and no image segmentation is required
- Images lack multiple markers or compartmental structure (e.g., single-channel images without membrane staining), where the full-marker random-forest compartment classification cannot be trained
- A fully automated deep-learning-only segmentation is preferred and training annotations/labels for the pixel classifier cannot be provided

## Inputs

- Multiplexed image stack (e.g., MIBI breast carcinoma TMA, 41 patients, 36 markers, 2048x2048 pixels at 0.5 um/pixel)
- Multiplexed immunofluorescence images of colorectal adenoma TMA cores
- Trained random-forest pixel-classifier labels (nuclei, membranes, other intracellular structures) via Ilastik

## Outputs

- Single-cell segmentation masks resolving cells with internal membranes
- Per-cell marker intensity quantification table over the entire cell and in nucleus, membrane, and cytoplasm compartments (e.g., miriam_mibi_results.csv)

## How to apply

Train the random-forest pixel classifier using the entire marker set (e.g., all 36 markers in the MIBI breast carcinoma TMA) so that nuclei, membranes, and other intracellular structures are labeled — the article reports that using the full marker set yielded more complete membranes than subsets. Ilastik is required for this classification step per the repository README, and the pipeline itself is implemented in Matlab and Python, which produce largely similar results. Feed the compartment probability maps into MIRIAM's membrane-extension and internal-membrane cell-separation routines to resolve individual cells despite incomplete membranes. Finally, quantify each marker's intensity per cell over the entire cell as well as in the nucleus, membrane, and cytoplasm compartments.

## Related tools

- **MIRIAM** (Pipeline that consumes random-forest compartment classification, extends incomplete cell membranes, separates cells with internal membranes, and quantifies markers per compartment) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Required for some pixel-classification tasks in the MIRIAM pipeline per the README) — https://www.ilastik.org/download.html
- **Python** (Implementation of MIRIAM used to run the segmentation and quantification (largely similar results to Matlab)) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Implementation of MIRIAM used to run the segmentation and quantification) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Compare image-level segmentation to manual annotations: MIRIAM achieved DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08 on the annotated colorectal adenoma TMA cores (240 sub-images), outperforming Mesmer (DSC 0.59 ± 0.10) and Voronoi (DSC 0.32 ± 0.11)
- Segmentation masks show complete membrane boundaries separating cells with internal membranes (e.g., columnar epithelial cells), unlike Voronoi which undersegments cells towards the lumen
- Quantification table contains per-cell intensity for every marker in four compartments (entire cell, nucleus, membrane, cytoplasm) with no missing marker columns
- Matlab and Python implementations yield largely similar results (Supplemental Figure 6), so cross-implementation agreement is a sanity check

## Limitations

- The t-SNE cell-shape analysis using autoencoder latent vectors did not reveal discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5)
- Repository README provides no changelog or version numbers for the Matlab/Python implementations; example data omits the image stacks (too large), which must be regenerated or downloaded separately
- No accession ID or public availability statement is given for the breast cancer MIBI dataset or the manually annotated ground-truth data, complicating reproduction
- Performance depends on training the pixel classifier with the entire marker set; the README notes ilastik must be installed and a tutorial followed

## Evidence
