---
name: coverage-threshold-site-subsampling
description: Use when benchmarking nanopore methylation callers against WGBS and you
  need to know if per-site correlation with bisulfite methylation frequencies holds
  as minimum read coverage increases.
when_to_use_negative:
- You only have per-read methylation calls and no aggregated per-site coverage/frequency
  BED tables — the threshold subsampling requires site-level coverage.
- Your comparison regions are not matched to WGBS measurements (e.g., no bisulfite
  data at the target loci), so no correlation can be computed at any threshold.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: METEORE
  role: Produces the augmented BED per-site methylation frequency tables (strand-split
    and combined-strand) for each tool and the consensus used as subsampling input
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Methylation caller whose per-site output is evaluated for coverage-stability
    against WGBS (correlation above 0.9)
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Methylation caller whose per-site output is evaluated for coverage-stability
    against WGBS (correlation above 0.9)
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Guppy
  role: Methylation caller whose per-site output is subsampled at coverage thresholds;
    shows lower agreement at fully methylated sites
- name: Tombo
  role: Methylation caller whose per-site output shows lower correlations with WGBS
    at all coverage levels
- name: Nanopolish
  role: Methylation caller whose per-site output is subsampled at coverage thresholds
    against WGBS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:d7d0545084d5c3fa74850c8033a7cc40369020b1033cb4bfcc9d7e078029e1dc
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
  - Tombo
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# coverage-threshold-site-subsampling

## Summary

Subsamples intersected nanopore-vs-WGBS CpG methylation frequency tables at per-site coverage thresholds (1x, 5x, 10x, 15x, 20x) and computes Pearson correlations per tool to test whether nanopore-WGBS concordance is stable across coverage levels.

## When to use

Use when benchmarking nanopore methylation callers against WGBS and you need to know if per-site correlation with bisulfite methylation frequencies holds as minimum read coverage increases.

## When NOT to use

- You only have per-read methylation calls and no aggregated per-site coverage/frequency BED tables — the threshold subsampling requires site-level coverage.
- Your comparison regions are not matched to WGBS measurements (e.g., no bisulfite data at the target loci), so no correlation can be computed at any threshold.

## Inputs

- Per-site methylation frequency tables in augmented BED format for Nanopolish, DeepSignal, Tombo, Guppy, Megalodon and METEORE consensus (both strand-split and combined-strand versions)
- WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions
- Coordinate-intersected nanopore/WGBS site table with per-site coverage and methylation frequency columns

## Outputs

- Table (TSV/CSV) of Pearson correlation between nanopore and WGBS methylation frequencies per tool at each minimum coverage threshold (1x, 5x, 10x, 15x, 20x)
- Coverage-stability summary indicating whether correlations remain approximately constant across thresholds per tool

## How to apply

Load per-site methylation frequency tables in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) for Nanopolish, DeepSignal, Tombo, Guppy, Megalodon and the METEORE consensus, using the combined-strand tables that average methylation frequencies and sum coverage across both strands. Intersect these with matched WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions on genomic coordinate. In R (data.table/dplyr) or Python (pandas/scipy.stats.pearsonr), iteratively filter the intersected sites to minimum coverage thresholds of 1x, 5x, 10x, 15x and 20x and compute the Pearson correlation between nanopore and WGBS methylation frequencies at each threshold per tool. The rationale is that the article found correlations were approximately constant across coverage levels, so any strong threshold dependence signals a tool-specific artifact rather than a sampling-depth effect.

## Related tools

- **METEORE** (Produces the augmented BED per-site methylation frequency tables (strand-split and combined-strand) for each tool and the consensus used as subsampling input) — https://github.com/comprna/METEORE
- **Megalodon** (Methylation caller whose per-site output is evaluated for coverage-stability against WGBS (correlation above 0.9)) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Methylation caller whose per-site output is evaluated for coverage-stability against WGBS (correlation above 0.9)) — https://github.com/bioinfomaticsCSU/deepsignal
- **Guppy** (Methylation caller whose per-site output is subsampled at coverage thresholds; shows lower agreement at fully methylated sites)
- **Tombo** (Methylation caller whose per-site output shows lower correlations with WGBS at all coverage levels)
- **Nanopolish** (Methylation caller whose per-site output is subsampled at coverage thresholds against WGBS)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(sub['meth_nanopore'], sub['meth_wgbs'])  # sub = sites filtered to coverage >= threshold
```

## Evaluation signals

- Megalodon, DeepSignal and METEORE Pearson correlations with WGBS stay above 0.9 and are approximately constant across the 1x–20x coverage thresholds.
- Tombo shows lower correlations than the other tools at all coverage levels.
- Combined-strand BED tables have coverage values equal to the sum of the two strand-specific coverages, and methylation frequencies equal to their average.
- The correlation-per-coverage table contains one row per tool per threshold (5 thresholds) with no missing entries for intersected CpG sites in the ten nCATS regions.

## Limitations

- Discrepancies with WGBS may reflect detection differences between nanopore and bisulfite or the disparate model types and training regimes of the tools, not coverage effects alone.
- The M.SssI methylated control has only 95% methylation efficiency, which can depress agreement with expected methylation frequency.
- No accession or repository identifiers for the WGBS comparison datasets are provided in the discussion, complicating reuse.
- Combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support.

## Evidence

- [results] Correlations between nanopore and WGBS methylation were approximately constant across coverage levels; Megalodon, DeepSignal, and METEORE had correlations above 0.9: [evidence span withheld — anchor status: not_found]
- [readme] augmented BED format: "METEORE can now produce two per-site result files in an **augmented BED format** for each tool except for DeepMod"
- [readme] combined strands: "we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage"
- [discussion] tools underprediction: "DeepMod and Guppy tended to underpredict, and Tombo and Guppy showed local discrepancies with the other tools."
- [discussion] M.SssI efficiency: "A possible reason for this low agreement is that the M.Sssl treatment used to generate the CpG-methylated control only has 95% efficiency"
