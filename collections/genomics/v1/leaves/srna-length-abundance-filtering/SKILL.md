---
name: srna-length-abundance-filtering
description: Use when raw adaptor-trimmed sRNA fasta/fastq libraries must be reduced
  to a clean 18-26 nt, abundance-filtered read set prior to genome mapping or miRNA/tasiRNA
  prediction.
when_to_use_negative:
- Input is still raw FASTQ requiring adaptor clipping first — adaptor trimming with
  fastx_clipper is a separate upstream step (Step 1 with --trim)
- You intend to re-run only later stages (annotation/reporting) with different parameters
  — miRPursuit's staged design lets you restart from a later step rather than re-filtering
- The libraries have already been genome-mapped with patman (Step 2); genome-hit filtering
  should then be handled there, not re-applied here
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: miRPursuit
  role: 'Pipeline that runs the Filtering stage (Step 1: Wbench Filter) via sRNAworkFlow.sh,
    connects inputs/outputs between stages, and stores read counts in the count directory'
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: Java
  role: Runtime required to execute the UEA sRNA workbench Filter tool invoked by
    the pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:36609f0c7efa8a4e881174aa0e7f65eb1d3693e227212942b0e77af9221134e5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/forestbiotech-lab/miRPursuit
  tools:
  - Java
  - miRPursuit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# srna-length-abundance-filtering

## Summary

The miRPursuit Filtering stage uses the UEA sRNA workbench Filter tool to retain fasta sRNA reads of 18-26 nt with abundance >=5, removing low-complexity, invalid and t/rRNA reads before genome mapping, with parameters set in wbench_filter.cfg.

## When to use

Use when raw adaptor-trimmed sRNA fasta/fastq libraries must be reduced to a clean 18-26 nt, abundance-filtered read set prior to genome mapping or miRNA/tasiRNA prediction.

## When NOT to use

- Input is still raw FASTQ requiring adaptor clipping first — adaptor trimming with fastx_clipper is a separate upstream step (Step 1 with --trim)
- You intend to re-run only later stages (annotation/reporting) with different parameters — miRPursuit's staged design lets you restart from a later step rather than re-filtering
- The libraries have already been genome-mapped with patman (Step 2); genome-hit filtering should then be handled there, not re-applied here

## Inputs

- fasta sRNA library files (e.g. test_dataset-1.fa, test_dataset-2.fa) named with a common stem and sequential numbering, from INSERTS_DIR
- config/workbench_filter_*.cfg (wbench_filter.cfg) with filtering parameters
- config/workdirs.cfg with FILTER_SUF filter-suffix selection

## Outputs

- Filtered fasta sRNA libraries within 18-26 nt, abundance >=5, t/rRNA and low-complexity removed
- Per-library kept-sequence counts stored in the workdir count directory (space-separated .tsv-style files)

## How to apply

Load input fasta sRNA libraries sharing a common name stem with sequential numbering (e.g. test_dataset-1.fa, test_dataset-2.fa) from INSERTS_DIR, as configured in workdirs.cfg. Apply the UEA sRNA workbench Filter tool to each library using the broad-range defaults from the wbench_filter.cfg configuration file: min_length=18, max_length=26, min_abundance=5, with filter_low_comp=true and filter_invalid=true enabled. Remove t/rRNA reads by keeping trrna=true and trrna_sense_only=false, while leaving genome-hit, normalized-abundance and kill-list filtering disabled (filter_genome_hits=false, filter_norm_abund=false, filter_kill_list=false) since genome filtering is a separate later step done with patman. Write the filtered fasta libraries to the working directory, matching the FILTER_SUF=18_26_5 filter-suffix convention chosen in workdirs.cfg, and tabulate per-library kept counts from the count directory. Filtered libraries feed Step 2 (Filter Genome & miRBase) where reads are mapped to the genome with 0 mismatches using patman.

## Related tools

- **miRPursuit** (Pipeline that runs the Filtering stage (Step 1: Wbench Filter) via sRNAworkFlow.sh, connects inputs/outputs between stages, and stores read counts in the count directory) — https://github.com/forestbiotech-lab/miRPursuit
- **Java** (Runtime required to execute the UEA sRNA workbench Filter tool invoked by the pipeline)

## Examples

```
bash pathToMirPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset- -s 1
```

## Evaluation signals

- Every sequence in each filtered library is within the 18-26 nt length range and has abundance >=5
- t/rRNA reads are removed (trrna=true, trrna_sense_only=false) and low-complexity/invalid reads are absent
- Per-library kept counts in the count directory are consistent with the filter discard log
- Output file names follow the FILTER_SUF=18_26_5 filter-suffix convention
- Filter step is the first pipeline step (-s 1: Wbench Filter), and its output precedes patman genome mapping with 0 mismatches

## Limitations

- Filtering parameters are fixed by the selected config file (FILTER_SUF picks the predefined filter settings); changing them requires editing wbench_filter.cfg and re-running from this stage
- Quality control (fastqc) is not yet implemented in the pre-preprocessing stage, so read-quality issues are not caught before filtering
- Counts files are space-separated despite the .tsv-style naming, which may complicate downstream parsing

## Evidence

- [readme] FILTER_SUF selects the predefined filter settings in workdirs.cfg: [evidence span withheld — anchor status: external_doc]
