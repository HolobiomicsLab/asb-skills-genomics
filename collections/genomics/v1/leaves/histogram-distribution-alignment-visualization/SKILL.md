---
name: histogram-distribution-alignment-visualization
description: Use when comparing normalization methods on multiplex tissue imaging
  (MTI) marker intensity distributions, especially to test whether peak alignment
  preserves distribution shape for unimodal (e.g., PD-L1) or bimodal (e.g., ECAD)
  markers.
when_to_use_negative:
- Markers with highly heterogeneous or irregular distributions (e.g., CD45 in PRAD-CyCIF)
  that violate the assumptions of the automatic workflow — these require UniFORM's
  guided fine-tuning mode instead
- mIHC (chromogenic) data, where linear normalization factors are unsuitable due to
  non-linear enzyme-catalyzed signal amplification
- Pixel-level normalization tasks where segmentation masks for non-cell region exclusion
  are unavailable
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: UniFORM
  role: Python pipeline performing automatic rigid-landmark functional registration
    to align marker intensity distributions and normalize raw feature data
  repo: https://github.com/kunlunW/UniFORM
- name: MxNorm
  role: R benchmark normalization method run with the registration option for comparison
    of distribution alignment
  repo: https://github.com/ColemanRHarris/mxnorm
- name: pyComBat
  role: Python benchmark batch-correction method (ComBat) applied to the same distributions
    for comparison
  repo: https://epigenelabs.github.io/pyComBat/
- name: plotly
  role: Interactive plotting library used for figure generation of intensity distributions
  repo: https://github.com/plotly/plotly.py
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:c38e32dfdf51cd53fb18eb37477241d7803cc54e143fee41c00db9eabbaff09e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - MxNorm
  - Python
  - UniFORM
  - plotly
  - pyComBat
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# histogram-distribution-alignment-visualization

## Summary

Reproduces UniFORM's marker intensity distribution alignment analysis: computes histograms of log-transformed cell intensities, aligns them via automatic rigid-landmark functional registration (maximum cross-correlation), and visualizes before/after normalization for benchmarks like MxNorm registration, Z score, ComBat, and mean division.

## When to use

Use when comparing normalization methods on multiplex tissue imaging (MTI) marker intensity distributions, especially to test whether peak alignment preserves distribution shape for unimodal (e.g., PD-L1) or bimodal (e.g., ECAD) markers.

## When NOT to use

- Markers with highly heterogeneous or irregular distributions (e.g., CD45 in PRAD-CyCIF) that violate the assumptions of the automatic workflow — these require UniFORM's guided fine-tuning mode instead
- mIHC (chromogenic) data, where linear normalization factors are unsuitable due to non-linear enzyme-catalyzed signal amplification
- Pixel-level normalization tasks where segmentation masks for non-cell region exclusion are unavailable

## Inputs

- PRAD-CyCIF single-cell feature data (AnnData .h5ad or pickle files) from Zenodo record 16383766
- CRC-ORION single-cell feature data from HTAN (participant ID HTA7_926)
- Marker intensity columns for ECAD and PD-L1 (log-transformed cell intensities)

## Outputs

- Aligned/normalized marker intensity distributions per method (UniFORM, MxNorm registration, Z score, ComBat, mean division)
- Figure 2A-style plots of intensity distributions before and after normalization, highlighting peak alignment and shape preservation

## How to apply

Load single-cell feature data — PRAD-CyCIF AnnData .h5ad files from Zenodo record 16383766 and CRC-ORION data from HTAN (participant HTA7_926) — into the UniFORM Python pipeline. Compute the histogram distribution of log-transformed cell intensities for each marker (e.g., bimodal ECAD in PRAD-CyCIF, unimodal PD-L1 in CRC-ORION), convert the histograms into functional data curves, and align them to a reference distribution using a maximum cross-correlation algorithm; apply the estimated normalization factors to the raw feature data. Run the benchmark methods on the same distributions — MxNorm with the registration option, Z score, ComBat (pyComBat), and mean division — then plot intensity distributions before and after each normalization to reproduce the Figure 2A layout. Correctness is judged by whether UniFORM aligns the negative population peaks while retaining the original bimodal/unimodal distribution shape of positive populations, consistent with the 'preserve biology' criterion, and whether the competing methods fail at peak alignment and/or shape preservation for the bimodal ECAD marker.

## Related tools

- **UniFORM** (Python pipeline performing automatic rigid-landmark functional registration to align marker intensity distributions and normalize raw feature data) — https://github.com/kunlunW/UniFORM
- **MxNorm** (R benchmark normalization method run with the registration option for comparison of distribution alignment) — https://github.com/ColemanRHarris/mxnorm
- **pyComBat** (Python benchmark batch-correction method (ComBat) applied to the same distributions for comparison) — https://epigenelabs.github.io/pyComBat/
- **plotly** (Interactive plotting library used for figure generation of intensity distributions) — https://github.com/plotly/plotly.py

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git && python -c "import uniform; uniform.align(hist_feature_data, markers=['ECAD','PD-L1'])"
```

## Evaluation signals

- UniFORM aligns the negative population peaks of the bimodal ECAD distribution while preserving the overall bimodal shape of positive populations
- The unimodal PD-L1 (CRC-ORION) distribution is aligned without distorting its unimodal shape
- Benchmark methods (MxNorm registration, Z score, ComBat, mean division) fail in peak alignment and/or shape preservation for the bimodal ECAD marker
- Output plot reproduces the Figure 2A layout of the paper
- Kullback-Leibler divergence-based flagging does not fire for well-behaved markers like ECAD and PD-L1

## Limitations

- Highly heterogeneous/irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the assumptions of the automatic normalization workflow and are flagged by a Kullback-Leibler divergence-based metric, requiring manual fine-tuning
- Effectiveness depends on the quality of the MTI dataset and its cell/nuclei segmentation masks; compromised signals cannot be recovered
- TMA-Lunaphore dataset used in the paper is not publicly available (only upon request from authors)

## Evidence
