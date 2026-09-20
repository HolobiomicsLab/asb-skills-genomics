---
name: cpg-island-hypomethylation-visualization
description: Use when comparing nanopore per-site methylation calls (augmented BED
  files from Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, METEORE) against WGBS
  methylation frequencies at CpG sites in targeted regions to check regional agreement
  and coverage stability.
when_to_use_negative:
- Input is per-read fast5 signal data rather than per-site aggregated methylation
  frequency tables — this skill operates on the augmented BED per-site outputs produced
  downstream
- Analysis goal is single-read methylation status or controlled mixture benchmarking
  rather than regional agreement with WGBS at CpG sites
- No matched WGBS methylation frequencies are available for the target regions, so
  no reference profile exists to compare against
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: METEORE
  role: Provides Snakemake pipelines generating the per-site augmented BED methylation
    frequency tables (including combined-strand files) used as input; its consensus
    prediction is one of the profiles visualized
  repo: https://github.com/comprna/METEORE
- name: Snakemake
  role: Reproducible pipeline framework for the methylation-calling tools that produce
    the per-site outputs
- name: Megalodon
  role: Best-performing tool whose per-site methylation frequencies are intersected
    with WGBS and profiled via LOESS
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Tool whose per-site calls are compared with WGBS; maintains correlation >0.9
    across coverage levels
  repo: https://github.com/bioinfomaticsCSU/deepsignal
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:650e33e9f72fbfd79b4d8d2a4e654be28a16fffc25e04697422d61b79ea31d7c
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
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 22
---

# cpg-island-hypomethylation-visualization

## Summary

Visualize and validate CpG island hypomethylation by fitting LOESS smoothing curves of per-site methylation frequency versus genomic position for each nanopore methylation tool and WGBS across Cas9-targeted nCATS regions, and quantify where tool profiles deviate from the WGBS reference.

## When to use

Use when comparing nanopore per-site methylation calls (augmented BED files from Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, METEORE) against WGBS methylation frequencies at CpG sites in targeted regions to check regional agreement and coverage stability.

## When NOT to use

- Input is per-read fast5 signal data rather than per-site aggregated methylation frequency tables — this skill operates on the augmented BED per-site outputs produced downstream
- Analysis goal is single-read methylation status or controlled mixture benchmarking rather than regional agreement with WGBS at CpG sites
- No matched WGBS methylation frequencies are available for the target regions, so no reference profile exists to compare against

## Inputs

- Per-site methylation frequency tables in augmented BED format for Nanopolish, DeepSignal, Tombo, Guppy, Megalodon and METEORE consensus
- Combined-strand methylation tables (averaged frequencies, summed coverage) for each tool
- WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions

## Outputs

- Correlation-per-coverage table (TSV/CSV) of Pearson correlations between nanopore and WGBS methylation frequencies at 1x, 5x, 10x, 15x and 20x thresholds per tool
- LOESS profile figures of methylation frequency versus genomic position per tool and WGBS along each of the ten target regions

## How to apply

Load the per-site methylation frequency tables in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) for each tool, using the combined-strand tables that merge both strands by averaging methylation frequencies and summing coverage. Intersect tool calls with matched WGBS methylation frequencies at CpG sites within the ten Cas9-targeted nCATS regions on genomic coordinate. Subsample intersected sites at minimum coverage thresholds of 1x, 5x, 10x, 15x and 20x and compute Pearson correlations between nanopore and WGBS methylation frequencies per tool (R data.table/dplyr or Python pandas/scipy.stats.pearsonr). Fit LOESS curves of methylation frequency versus genomic position per tool and WGBS along each target region to visualize CGI hypomethylation and local discrepancies (e.g., Guppy/DeepMod underprediction at fully methylated loci such as IRF4, and Tombo/Guppy discrepancies at ACKR1/CADM3 and TPO). Export the correlation-per-coverage table as TSV/CSV and the LOESS profile figures.

## Related tools

- **METEORE** (Provides Snakemake pipelines generating the per-site augmented BED methylation frequency tables (including combined-strand files) used as input; its consensus prediction is one of the profiles visualized) — https://github.com/comprna/METEORE
- **Snakemake** (Reproducible pipeline framework for the methylation-calling tools that produce the per-site outputs)
- **Megalodon** (Best-performing tool whose per-site methylation frequencies are intersected with WGBS and profiled via LOESS) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Tool whose per-site calls are compared with WGBS; maintains correlation >0.9 across coverage levels) — https://github.com/bioinfomaticsCSU/deepsignal

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(sub_sites['meth_freq_nanopore'], sub_sites['meth_freq_wgbs'])  # per tool at 1x/5x/10x/15x/20x min-coverage subsamples
```

## Evaluation signals

- Pearson correlations between nanopore and WGBS methylation frequencies remain above 0.9 for Megalodon, DeepSignal and METEORE and are approximately constant across the 1x–20x coverage thresholds
- Combined-strand tables correctly average methylation frequencies and sum coverage across the two strands
- LOESS profiles from all tools recapitulate CpG island hypomethylation consistent with the reported nCATS/WGBS comparison
- Local deviations from the WGBS profile are identifiable and interpretable, e.g., Guppy/DeepMod underprediction at fully methylated loci such as IRF4
- Intersected per-site tables align on genomic coordinate with matched WGBS CpG sites within each of the ten target regions

## Limitations

- Discrepancies with WGBS could stem from detection differences between nanopore and WGBS or from the disparate types of models and training regimes used by the tools
- Tombo had lower correlations with WGBS at all coverage levels, and Tombo/Guppy showed local discrepancies with the other tools (e.g., at ACKR1/CADM3 and TPO)
- Combining multiple tools improved accuracy but may be impractical for routine analyses due to running times for methods without GPU support (Megalodon relies on GPUs to be time efficient)

## Evidence

- [results] combined-strand tables merge both strands by averaging methylation frequencies and summing coverage: "In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage."
- [results] Megalodon, DeepSignal and METEORE correlations above 0.9 across coverage levels: [evidence span withheld — anchor status: not_found]
- [discussion] local tool discrepancies vs WGBS profiles: "DeepMod and Guppy tended to underpredict, and Tombo and Guppy showed local discrepancies with the other tools."
- [discussion] Guppy extreme-case behavior at fully methylated sites: "Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites"
- [discussion] WGBS comparison consistency: "The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses"
- [readme] augmented BED format fields: "The first output file contains the following fields: 1. Reference chromosome 2. Start position in chromosome 3. End position in chromosome 4. Read coverage 5. Methylation (i.e. methylation frequency)"
