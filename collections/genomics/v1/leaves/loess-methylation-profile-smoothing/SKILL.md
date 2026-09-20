---
name: loess-methylation-profile-smoothing
description: Use when comparing per-site methylation frequencies from nanopore tools
  (augmented BED per-site tables) against WGBS along targeted regions, to reveal local
  CpG island hypomethylation patterns and tool-specific local discrepancies.
when_to_use_negative:
- Input is raw fast5 signal data rather than per-site methylation frequency tables
  — methylation calling must be performed first via the Snakemake pipelines
- Per-read methylation scores are needed (e.g., for training METEORE combination models)
  rather than genome-anchored per-site frequencies
- The goal is global accuracy benchmarking against control mixtures of methylated/unmethylated
  reads rather than regional agreement with WGBS profiles
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: METEORE
  role: Provides Snakemake pipelines that produce the augmented BED per-site methylation
    frequency tables (and combined-strand tables) used as input for LOESS smoothing
  repo: https://github.com/comprna/METEORE
- name: Snakemake
  role: Runs the per-tool methylation calling workflows that generate the per-site
    output files
- name: Megalodon
  role: One of the best-performing methylation callers whose per-site calls are smoothed
    and compared against WGBS
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Methylation caller whose per-site calls are compared with WGBS via LOESS profiles;
    combined with Megalodon gave a good balance of accuracy and running time
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Guppy
  role: Methylation caller whose LOESS profile reveals underprediction at fully methylated
    sites and local discrepancies
- name: DeepMod
  role: Methylation caller showing underprediction tendency in LOESS profile comparisons
  repo: https://github.com/WGLab/DeepMod
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:893ebe7c51ce6a8fdc010df753e1cfecc9720d9f9f034e7632a2b62952aa5028
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
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# loess-methylation-profile-smoothing

## Summary

Fit LOESS smoothing curves of per-site CpG methylation frequency versus genomic position for each nanopore methylation-calling tool and matched WGBS data, to visualize CpG island hypomethylation and quantify local under- or overprediction of 5mC calls across tools such as Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod, and the METEORE consensus.

## When to use

Use when comparing per-site methylation frequencies from nanopore tools (augmented BED per-site tables) against WGBS along targeted regions, to reveal local CpG island hypomethylation patterns and tool-specific local discrepancies.

## When NOT to use

- Input is raw fast5 signal data rather than per-site methylation frequency tables — methylation calling must be performed first via the Snakemake pipelines
- Per-read methylation scores are needed (e.g., for training METEORE combination models) rather than genome-anchored per-site frequencies
- The goal is global accuracy benchmarking against control mixtures of methylated/unmethylated reads rather than regional agreement with WGBS profiles

## Inputs

- Per-site methylation frequency tables in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) for Nanopolish, DeepSignal, Tombo, Guppy, Megalodon and the METEORE consensus
- Combined-strand per-site tables merging both strands by averaging methylation frequencies and summing coverage
- Matched WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions

## Outputs

- LOESS smoothing curves of methylation frequency versus genomic position per tool and WGBS along each of the ten target regions
- Correlation-per-coverage table (TSV/CSV) of Pearson correlations between nanopore and WGBS methylation frequencies at 1x, 5x, 10x, 15x, 20x thresholds
- Exported LOESS profile figures highlighting local tool-WGBS discrepancies

## How to apply

Load each tool's per-site methylation frequency table in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand), using the combined-strand tables that merge both strands by averaging methylation frequencies and summing coverage. Intersect these calls with matched WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions on genomic coordinate using data.table/dplyr in R or pandas in Python. Fit LOESS smoothing curves of methylation frequency versus genomic position per tool and per WGBS along each target region to visualize CpG island hypomethylation. Quantify local under- or overprediction where a tool's LOESS profile deviates from the WGBS profile (e.g., Guppy/DeepMod underprediction at fully methylated loci such as IRF4, and Tombo/Guppy discrepancies at ACKR1/CADM3 and TPO). Alongside the LOESS profiles, subsample intersected sites at minimum coverage thresholds of 1x, 5x, 10x, 15x and 20x and compute Pearson correlations between nanopore and WGBS methylation frequencies to check that concordance is stable across coverage.

## Related tools

- **METEORE** (Provides Snakemake pipelines that produce the augmented BED per-site methylation frequency tables (and combined-strand tables) used as input for LOESS smoothing) — https://github.com/comprna/METEORE
- **Snakemake** (Runs the per-tool methylation calling workflows that generate the per-site output files)
- **Megalodon** (One of the best-performing methylation callers whose per-site calls are smoothed and compared against WGBS) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Methylation caller whose per-site calls are compared with WGBS via LOESS profiles; combined with Megalodon gave a good balance of accuracy and running time) — https://github.com/bioinfomaticsCSU/deepsignal
- **Guppy** (Methylation caller whose LOESS profile reveals underprediction at fully methylated sites and local discrepancies)
- **DeepMod** (Methylation caller showing underprediction tendency in LOESS profile comparisons) — https://github.com/WGLab/DeepMod

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(sub.loc[sub.cov>=10,'tool_meth_freq'], sub.loc[sub.cov>=10,'wgbs_meth_freq'])
```

## Evaluation signals

- Pearson correlations between nanopore and WGBS methylation frequencies for Megalodon, DeepSignal, and METEORE stay above 0.9 and are approximately constant across the 1x–20x coverage thresholds
- Tombo shows lower correlations with WGBS than the other tools at all coverage levels
- All tools' LOESS profiles recapitulate CpG island (CGI) hypomethylation consistent with the reported nCATS/WGBS comparison
- Combined-strand BED output has exactly five fields (chromosome, start, end, coverage, methylation frequency) with coverage summed and frequencies averaged across strands
- Guppy/DeepMod LOESS profiles underpredict at fully methylated loci (e.g., IRF4), and Tombo/Guppy deviate locally from WGBS at ACKR1/CADM3 and TPO

## Limitations

- No accession or repository identifiers for the WGBS comparison datasets are provided in the discussion
- Discrepancies with WGBS could stem from detection differences between nanopore and WGBS or from disparate model types and training regimes of the tools
- Combining multiple tools improves accuracy but may be impractical for routine analyses due to running times of methods without GPU support

## Evidence

- [discussion] The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses: "The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses"
- [discussion] DeepMod and Guppy tended to underpredict, and Tombo and Guppy showed local discrepancies with the other tools.: "DeepMod and Guppy tended to underpredict, and Tombo and Guppy showed local discrepancies with the other tools."
- [readme] In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage.: "we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage"
- [discussion] Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites: "Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites"
