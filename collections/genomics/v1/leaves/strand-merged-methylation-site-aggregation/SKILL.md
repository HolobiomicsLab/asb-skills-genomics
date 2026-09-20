---
name: strand-merged-methylation-site-aggregation
description: Use when nanopore per-site methylation outputs from multiple tools must
  be harmonized into a single strand-merged BED table per CpG site for downstream
  correlation, coverage-threshold filtering, or WGBS comparison.
when_to_use_negative:
- When per-strand methylation information must be preserved, e.g. strand-sensitive
  analyses of asymmetric methylation — the merged table averages both strands and
  drops the strand field
- When the methylation caller is DeepMod, whose per-site output is not yet in the
  METEORE augmented BED format and cannot be strand-merged with this procedure
- When single-read (per-read) methylation scores are the target, e.g. for training
  METEORE combined models, which use per-read score files rather than aggregated per-site
  tables
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: METEORE
  role: Produces the per-site augmented BED outputs and the combined-strand (strand-merged)
    result files for each tool via Snakemake pipelines
  repo: https://github.com/comprna/METEORE
- name: Nanopolish
  role: Methylation-calling tool whose per-site output is converted into the augmented
    BED / combined-strand format via its METEORE Snakefile
  repo: https://github.com/comprna/METEORE
- name: DeepSignal
  role: Methylation-calling tool whose default outputs are reformatted into the shared
    augmented BED format before strand merging
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified-base prediction tool whose per-site output is strand-merged in the
    METEORE pipeline
  repo: https://github.com/comprna/METEORE
- name: Guppy
  role: Modified basecaller (dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config)
    whose per-site calls are strand-merged via its Snakefile
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Modified base detection tool (Guppy/Remora backend) whose per-site outputs
    feed the strand-merged tables
  repo: https://github.com/nanoporetech/megalodon
- name: Snakemake
  role: Workflow engine running the per-tool Snakefiles that generate the per-site
    and combined-strand output files
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:bb0e99b24e91eba91b47223b6ae296d5b1cfbecf187fd4a0555296584b175fa2
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# strand-merged-methylation-site-aggregation

## Summary

Aggregates per-site CpG methylation calls from nanopore methylation-calling tools (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, METEORE) into combined-strand BED-format tables by averaging methylation frequencies across both strands and summing read coverage, enabling direct comparison with strand-agnostic WGBS methylation frequencies.

## When to use

Use when nanopore per-site methylation outputs from multiple tools must be harmonized into a single strand-merged BED table per CpG site for downstream correlation, coverage-threshold filtering, or WGBS comparison.

## When NOT to use

- When per-strand methylation information must be preserved, e.g. strand-sensitive analyses of asymmetric methylation — the merged table averages both strands and drops the strand field
- When the methylation caller is DeepMod, whose per-site output is not yet in the METEORE augmented BED format and cannot be strand-merged with this procedure
- When single-read (per-read) methylation scores are the target, e.g. for training METEORE combined models, which use per-read score files rather than aggregated per-site tables

## Inputs

- Per-site methylation frequency tables in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish, DeepSignal, Tombo, Guppy, Megalodon and the METEORE consensus
- Matched WGBS methylation frequencies at CpG sites (for downstream comparison)
- fast5 reads and a reference genome FASTA (e.g. data/ecoli_k12_mg1655.fasta) to run the upstream methylation-calling Snakemake pipelines

## Outputs

- Combined-strand per-site BED file per tool with fields: chromosome, start, end, coverage (summed across strands), methylation frequency (averaged across strands); strand column dropped
- Coverage-thresholded correlation table (TSV/CSV) of Pearson correlations between nanopore and WGBS methylation frequencies at 1x, 5x, 10x, 15x and 20x minimum coverage per tool
- LOESS methylation-frequency-versus-position profiles per tool and WGBS along target regions

## How to apply

Run the METEORE Snakemake pipelines (Nanopolish, Deepsignal1, Tombo, Guppy Snakefiles, or Megalodon directly) to generate per-site outputs in augmented BED format with fields chromosome, start, end, coverage, methylation frequency, strand. For each tool except DeepMod, produce the second METEORE output file in which methylation predictions from both strands at the same CpG site are combined by averaging the methylation frequencies and adding up the coverage, dropping the strand column. Use the combined-strand tables (not the stranded ones) when intersecting with matched WGBS methylation frequencies on genomic coordinate, e.g. at Cas9-targeted nCATS regions, and when subsampling at minimum coverage thresholds (1x, 5x, 10x, 15x, 20x) for Pearson correlation against WGBS. Rationale: the merged tables give a single comparable value per CpG across tools and platforms, which is required for the nCATS/WGBS concordance analysis where correlations above 0.9 for Megalodon, DeepSignal and METEORE were confirmed.

## Related tools

- **METEORE** (Produces the per-site augmented BED outputs and the combined-strand (strand-merged) result files for each tool via Snakemake pipelines) — https://github.com/comprna/METEORE
- **Nanopolish** (Methylation-calling tool whose per-site output is converted into the augmented BED / combined-strand format via its METEORE Snakefile) — https://github.com/comprna/METEORE
- **DeepSignal** (Methylation-calling tool whose default outputs are reformatted into the shared augmented BED format before strand merging) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified-base prediction tool whose per-site output is strand-merged in the METEORE pipeline) — https://github.com/comprna/METEORE
- **Guppy** (Modified basecaller (dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config) whose per-site calls are strand-merged via its Snakefile) — https://github.com/comprna/METEORE
- **Megalodon** (Modified base detection tool (Guppy/Remora backend) whose per-site outputs feed the strand-merged tables) — https://github.com/nanoporetech/megalodon
- **Snakemake** (Workflow engine running the per-tool Snakefiles that generate the per-site and combined-strand output files) — https://github.com/comprna/METEORE

## Examples

```
python meteore_combine.py -i input_file.txt -t RandomForest -o output.tsv; python meteore_combine.py per_site -i output_predictions.txt -o output.bed -f 0
```

## Evaluation signals

- Combined-strand BED has exactly 5 columns (chromosome, start, end, coverage, methylation frequency) with the strand column absent, while the stranded file has 6 columns
- Summed coverage in the merged file equals the sum of coverages from the two strand-specific rows at the same CpG coordinate
- Averaged methylation frequency in the merged file equals the mean of the two strand-specific methylation frequencies at that CpG
- One row per CpG coordinate per tool, with coordinates matching the stranded inputs' CpG positions
- Downstream Pearson correlations with WGBS computed on merged tables stay above 0.9 and approximately constant across 1x–20x coverage thresholds for Megalodon, DeepSignal and METEORE

## Limitations

- DeepMod per-site output is not yet available in the METEORE augmented BED format and cannot be strand-merged with this procedure
- Combining multiple tools (and thus producing consensus merged tables) can be impractical for routine analyses due to running times of methods without GPU support
- Low agreement with expected methylation frequency may reflect the M.SssI methylated control's ~95% methylation efficiency rather than aggregation errors

## Evidence

- [readme] combined-strand output README: "In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage."
- [readme] augmented BED format: "METEORE can now produce two per-site result files in an **augmented BED format** for each tool except for DeepMod (which will be updated very soon)."
- [results] WGBS comparison coverage stability: [evidence span withheld — anchor status: not_found]
- [discussion] combining tools impracticality: "Although combining multiple tools improved the accuracy even further, it might be impractical for routine analyses, due to the running times for some methods without GPU support"
