---
name: per-cell-compartment-marker-quantification
description: Use when a multiplexed imaging segmentation mask (e.g., from MIRIAM,
  MIBI or immunofluorescence TMA data) is available and per-cell, per-compartment
  marker intensity values are needed as a single-cell feature table.
when_to_use_negative:
- No single-cell segmentation masks exist yet — run MIRIAM segmentation first; this
  skill only quantifies within existing cell/compartment boundaries
- Input is already a per-cell feature/intensity table — quantification would be redundant
- Images lack multi-channel marker content or compartment labels (nucleus/membrane/cytoplasm)
  cannot be defined by pixel classification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Pipeline whose membrane-extension and internal-membrane cell-separation routines
    produce the segmentation masks and compartment definitions over which markers
    are quantified
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random-forest pixel classification (required dependency per README) that labels
    nuclei, membranes, and other intracellular structures used as quantification compartments
  repo: https://www.ilastik.org/download.html
- name: Matlab
  role: Primary implementation in which MIRIAM cell segmentation and per-cell marker
    quantification are executed
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation producing largely similar quantification
    results (Supplemental Figure 6)
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:8c7fd3288402aba6baf94d75380c8f4da33419c29871d8d239bd830193a1bff1
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-cell-compartment-marker-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify every marker's intensity in each segmented cell, both over the entire cell and separately in nucleus, membrane, and cytoplasm compartments, following MIRIAM's single-cell quantification scheme for multiplexed tissue images.

## When to use

Use when a multiplexed imaging segmentation mask (e.g., from MIRIAM, MIBI or immunofluorescence TMA data) is available and per-cell, per-compartment marker intensity values are needed as a single-cell feature table.

## When NOT to use

- No single-cell segmentation masks exist yet — run MIRIAM segmentation first; this skill only quantifies within existing cell/compartment boundaries
- Input is already a per-cell feature/intensity table — quantification would be redundant
- Images lack multi-channel marker content or compartment labels (nucleus/membrane/cytoplasm) cannot be defined by pixel classification

## Inputs

- Multiplexed image stack (e.g., MIBI breast carcinoma TMA, 2048x2048 pixels at 0.5 um/pixel, 36 markers)
- MIRIAM single-cell segmentation masks (Matlab or Python implementation output)
- Pixel-classification labels defining nucleus, membrane, and other intracellular structures (ilastik/random-forest output)

## Outputs

- Per-cell marker quantification table with intensity per marker over the entire cell and in nucleus, membrane, and cytoplasm compartments (e.g., miriam_mibi_results.csv)
- Compartment-level feature matrix usable for downstream t-SNE or clustering analyses

## How to apply

After running MIRIAM's random-forest pixel classification (trained with ilastik on the full marker set to define nuclei, membranes, and other intracellular structures) and its membrane-extension/internal-membrane cell-separation routines, assign each pixel to a cell and to a compartment class (nucleus, membrane, cytoplasm, entire cell). For each marker channel and each segmented cell, compute mean image intensity restricted to each compartment, integrating signal over incomplete membranes resolved by MIRIAM's extension step. Apply this identically across platforms — the article demonstrates it on both multiplexed immunofluorescence colorectal adenoma images and a MIBI breast carcinoma TMA (41 patients, 36 markers, 2048x2048 pixels at 0.5 um/pixel) — producing a per-cell quantification table suitable for downstream analyses such as t-SNE on marker intensities. Judge correctness by checking that every segmented cell carries values for all markers in all four measurement regions and that compartment sums are consistent with whole-cell intensities.

## Related tools

- **MIRIAM** (Pipeline whose membrane-extension and internal-membrane cell-separation routines produce the segmentation masks and compartment definitions over which markers are quantified) — https://github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random-forest pixel classification (required dependency per README) that labels nuclei, membranes, and other intracellular structures used as quantification compartments) — https://www.ilastik.org/download.html
- **Matlab** (Primary implementation in which MIRIAM cell segmentation and per-cell marker quantification are executed) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation producing largely similar quantification results (Supplemental Figure 6)) — https://github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Every segmented cell has an intensity value for each marker in each of the four regions: entire cell, nucleus, membrane, and cytoplasm
- Compartment-restricted intensities are mutually consistent with the whole-cell intensity (compartments partition the cell area)
- Downstream t-SNE on the resulting marker intensities should reveal discernible cellular population patterns (as in the article's Figure 3A)
- Quantification runs reproducibly with either the Matlab or Python MIRIAM implementations, which the article reports give largely similar results
- Output table (e.g., miriam_mibi_results.csv) contains one row per cell with marker-by-compartment intensity columns and no missing segmentation-to-marker joins

## Limitations

- Membranes may be incomplete in lower-quality data (e.g., the MIBI breast carcinoma TMA with a different marker set), so whole-cell and compartment intensities depend on MIRIAM's membrane-extension step rather than directly observed boundaries
- Accuracy of compartment assignment is bounded by the random-forest/ilastik pixel classifier; using the entire marker set for classification yielded more complete membranes than marker subsets
- The article notes quantitative segmentation similarity scores are referenced via figures rather than fully reported in text, so expected numeric ranges for compartment quantification fidelity are not explicitly stated

## Evidence
