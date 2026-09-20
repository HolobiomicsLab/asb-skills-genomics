---
name: nanopore-wgbs-methylation-correlation-analysis
description: Use when nanopore per-site methylation frequency tables (augmented BED
  format) must be compared to matched WGBS methylation frequencies across coverage
  thresholds (1x–20x) to verify concordance stability.
when_to_use_negative:
- Input is raw fast5 signal data rather than per-site methylation frequency tables
  — methylation must first be called with the individual tools
- The goal is per-read methylation accuracy against controlled methylated/unmethylated
  mixtures rather than per-site agreement with WGBS reference frequencies
- No matched WGBS data covering the nanopore-targeted CpG sites exists, so coordinate-wise
  intersection is impossible
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: METEORE
  role: Provides Snakemake pipelines producing per-site augmented-BED methylation
    tables and a consensus prediction; its combined-strand output is intersected with
    WGBS
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Methylation-calling tool whose per-site output is one of the nanopore methylation
    tables compared to WGBS; best overall performer requiring GPUs
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Methylation-calling tool whose per-site output is compared to WGBS across
    coverage thresholds
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: DeepMod
  role: Methylation-calling tool whose local discrepancies (underprediction) are quantified
    against WGBS LOESS profiles
  repo: https://github.com/WGLab/DeepMod
- name: Snakemake
  role: Workflow manager for the tool pipelines generating per-site methylation tables
    reproducibly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:d247aa446469b86d8ff85efbff05356134de70d5980272fafce4e5516ab9ea4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepMod
  - DeepSignal
  - Guppy
  - METEORE
  - Megalodon
  - Nanopolish
  - Snakemake
  - Tombo
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# nanopore-wgbs-methylation-correlation-analysis

## Summary

Assess the stability of nanopore CpG methylation calls against WGBS by computing Pearson correlations between methylation frequencies at matched CpG sites across per-site coverage thresholds, and visualize regional agreement with LOESS profiles along Cas9-targeted nCATS regions.

## When to use

Use when nanopore per-site methylation frequency tables (augmented BED format) must be compared to matched WGBS methylation frequencies across coverage thresholds (1x–20x) to verify concordance stability.

## When NOT to use

- Input is raw fast5 signal data rather than per-site methylation frequency tables — methylation must first be called with the individual tools
- The goal is per-read methylation accuracy against controlled methylated/unmethylated mixtures rather than per-site agreement with WGBS reference frequencies
- No matched WGBS data covering the nanopore-targeted CpG sites exists, so coordinate-wise intersection is impossible

## Inputs

- Per-site methylation frequency tables in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) for Nanopolish, DeepSignal, Tombo, Guppy, Megalodon and METEORE consensus
- Combined-strand methylation tables merging both strands by averaging methylation frequencies and summing coverage
- Matched WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions

## Outputs

- Correlation-per-coverage table (TSV/CSV) of Pearson correlations between nanopore and WGBS methylation frequencies at 1x, 5x, 10x, 15x and 20x thresholds per tool
- LOESS profile figures of methylation frequency versus genomic position per tool and WGBS along each of the ten target regions

## How to apply

Load the combined-strand per-site methylation tables (chromosome, start, end, coverage, methylation frequency, strand) for Nanopolish, DeepSignal, Tombo, Guppy, Megalodon and the METEORE consensus, which merge both strands by averaging methylation frequencies and summing coverage. Intersect these with matched WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions on genomic coordinate. Subsample the intersected sites at minimum coverage thresholds of 1x, 5x, 10x, 15x and 20x and compute the Pearson correlation between nanopore and WGBS methylation frequencies at each threshold per tool (R data.table/dplyr or Python pandas/scipy.stats.pearsonr). Fit LOESS smoothing curves of methylation frequency versus genomic position per tool and WGBS along each target region to visualize CpG island hypomethylation and local discrepancies, and quantify local under- or overprediction from LOESS deviations.

## Related tools

- **METEORE** (Provides Snakemake pipelines producing per-site augmented-BED methylation tables and a consensus prediction; its combined-strand output is intersected with WGBS) — https://github.com/comprna/METEORE
- **Megalodon** (Methylation-calling tool whose per-site output is one of the nanopore methylation tables compared to WGBS; best overall performer requiring GPUs) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Methylation-calling tool whose per-site output is compared to WGBS across coverage thresholds) — https://github.com/bioinfomaticsCSU/deepsignal
- **DeepMod** (Methylation-calling tool whose local discrepancies (underprediction) are quantified against WGBS LOESS profiles) — https://github.com/WGLab/DeepMod
- **Snakemake** (Workflow manager for the tool pipelines generating per-site methylation tables reproducibly)

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; sites = pd.read_csv('megalodon.bed', sep='\t'); wgbs = pd.read_csv('wgbs.bed', sep='\t'); m = sites.merge(wgbs, on=['chr','start']); r, p = pearsonr(m['methylation_nanopore'], m['methylation_wgbs'])
```

## Evaluation signals

- Megalodon, DeepSignal and METEORE Pearson correlations with WGBS remain above 0.9 and approximately constant across all coverage thresholds (1x–20x)
- Tombo shows lower correlations than the other tools at every coverage level
- All tools recapitulate CpG island hypomethylation in the LOESS profiles consistent with the reported nCATS/WGBS comparison
- Combined-strand tables correctly average methylation frequencies and sum coverage from both strands
- Guppy/DeepMod underprediction at fully methylated loci (e.g., IRF4) and Tombo/Guppy local discrepancies at ACKR1/CADM3 and TPO are detectable as LOESS deviations from WGBS

## Limitations

- Discrepancies with WGBS could stem from detection differences or disparate model types and training regimes of the tools
- No accession or repository identifiers for the WGBS comparison datasets are provided in the discussion
- Combining multiple tools (e.g., METEORE consensus) may be impractical for routine analyses due to running times for methods without GPU support

## Evidence

- [readme] augmented BED format: "METEORE can now produce two per-site result files in an **augmented BED format** for each tool except for DeepMod (which will be updated very soon)."
- [readme] combined strand fields: "we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage."
- [discussion] WGBS comparison consistency: "The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses"
- [discussion] discrepancy causes: "Although these discrepancies could be due to the detection differences between nanopore and WGBS, they could also point to the disparate types of models and training regimes used by the tools tested."
- [discussion] local underprediction: "Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites"
- [intro] benchmarking objective: "We have performed a systematic benchmarking of six tools for 5mC detection from nanopore sequencing fast5 files using individual reads, controlled methylation mixtures, Cas9-targeted sequencing, and"
