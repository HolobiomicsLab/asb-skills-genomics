---
name: augmented-bed-methylation-table-parsing
description: Use when input is a per-site CpG methylation file in augmented BED format
  (chromosome, start, end, coverage, methylation frequency, strand) from nanopore
  methylation tools and it must be loaded, merged across strands, or intersected with
  WGBS calls.
when_to_use_negative:
- Input is per-read methylation score data (e.g. METEORE per-read score files for
  combined-model training) rather than per-site augmented BED tables
- DeepMod output needs to be parsed into the augmented BED format, since METEORE did
  not yet produce per-site BED files for DeepMod
- Raw fast5 files or basecall fastq are the input — conversion (multi_to_single_fast5,
  fastq annotation) must happen before per-site tables exist
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: METEORE
  role: Source of the augmented BED per-site output files for each methylation-calling
    tool; its Snakemake pipelines generate the tables parsed here
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Produces per-site modified base detection output converted to the augmented
    BED format
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Its default output is reformatted into the same augmented BED format generated
    by other tools before parsing
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Snakemake
  role: Runs the per-tool pipelines (Nanopolish, DeepSignal, Tombo, Guppy Snakefiles)
    that generate the per-site BED outputs
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:ce166f37cdc5103048e0d5d2794b10b244fc471185b88691e5b5ca944013d7e6
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
  - R data.table/dplyr
  - Snakemake
  - Tombo
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# augmented-bed-methylation-table-parsing

## Summary

Parses the augmented-BED per-site methylation frequency tables produced by METEORE's Snakemake pipelines (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, METEORE consensus), including the combined-strand variant that averages methylation frequencies and sums coverage across strands, for downstream comparison with WGBS.

## When to use

Use when input is a per-site CpG methylation file in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) from nanopore methylation tools and it must be loaded, merged across strands, or intersected with WGBS calls.

## When NOT to use

- Input is per-read methylation score data (e.g. METEORE per-read score files for combined-model training) rather than per-site augmented BED tables
- DeepMod output needs to be parsed into the augmented BED format, since METEORE did not yet produce per-site BED files for DeepMod
- Raw fast5 files or basecall fastq are the input — conversion (multi_to_single_fast5, fastq annotation) must happen before per-site tables exist

## Inputs

- Augmented BED per-site methylation files (chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, and METEORE consensus
- Combined-strand augmented BED files (5 columns, no strand) with averaged methylation frequencies and summed coverage
- WGBS methylation frequency tables at CpG sites for coordinate-based intersection

## Outputs

- Loaded and typed per-site methylation tables (data.table or pandas DataFrame) with coverage and methylation frequency as numeric columns
- Coordinate-intersected nanopore/WGBS per-site tables used for per-coverage Pearson correlation analysis
- Correlation-per-coverage-threshold tables exported as TSV/CSV

## How to apply

Read each tool's per-site output with explicit column names since the augmented BED has 6 columns (reference chromosome, start, end, read coverage, methylation frequency, strandedness) and the combined-strand variant has 5 (strand dropped). For combined-strand tables, verify that methylation frequencies from both strands were averaged and coverage values summed at each CpG. Convert to a data.table (R) or pandas DataFrame (Python) so sites can be intersected with matched WGBS methylation frequencies at CpGs within the ten Cas9-targeted nCATS regions on genomic coordinate. Preserve coverage as a numeric column because downstream subsampling at 1x/5x/10x/15x/20x minimum-coverage thresholds depends on it. Check for strand-specific duplicates before collapsing, and treat DeepMod outputs separately since METEORE's per-site BED export did not yet cover DeepMod.

## Related tools

- **METEORE** (Source of the augmented BED per-site output files for each methylation-calling tool; its Snakemake pipelines generate the tables parsed here) — https://github.com/comprna/METEORE
- **Megalodon** (Produces per-site modified base detection output converted to the augmented BED format) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Its default output is reformatted into the same augmented BED format generated by other tools before parsing) — https://github.com/bioinfomaticsCSU/deepsignal
- **Snakemake** (Runs the per-tool pipelines (Nanopolish, DeepSignal, Tombo, Guppy Snakefiles) that generate the per-site BED outputs) — https://github.com/comprna/METEORE

## Examples

```
import pandas as pd; df = pd.read_csv('example.bed', sep='\t', names=['chrom','start','end','coverage','methylation','strand'])
```

## Evaluation signals

- Schema check: single-strand tables parse into exactly 6 fields and combined-strand tables into exactly 5 fields as defined by METEORE's augmented BED specification
- Combined-strand table CpG positions each appear once, with coverage equal to the sum of the two strand-level coverages
- Methylation frequency column values fall within [0, 1] and coverage values are positive integers
- Intersection with WGBS recovers sites only within the ten Cas9-targeted nCATS regions
- Subsampled tables at the 1x–20x minimum-coverage thresholds yield monotonically shrinking site counts and Pearson correlations that remain approximately constant across thresholds for Megalodon, DeepSignal, and METEORE (above 0.9)

## Limitations

- DeepMod per-site results were not yet available in the augmented BED format produced by METEORE's pipelines
- Coverage-dependent parsing is sensitive to low-coverage sites; correlation analyses subsample at 1x–20x thresholds to control this
- Discrepancies with WGBS at intersected CpGs may stem from detection differences or disparate model types and training regimes of the tools, not parsing errors

## Evidence

- [readme] Augmented BED format fields: "Reference chromosome
2. Start position in chromosome
3. End position in chromosome
4. Read coverage
5. Methylation (i.e. methylation frequency)
6. Strandedness"
- [readme] Combined-strand output definition: "we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage."
- [discussion] WGBS comparison validation: "The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses"
- [other] Coverage-stability finding: [evidence span withheld — anchor status: not_found]
- [readme] DeepMod output gap: "METEORE can now produce two per-site result files in an **augmented BED format** for each tool except for DeepMod (which will be updated very soon)."
