---
name: srna-library-count-tabulation
description: Use when fasta sRNA libraries (e.g. test_dataset-1.fa, test_dataset-2.fa)
  have passed through the Wbench Filter step and you need per-library counts of retained
  sequences to validate filtering and populate the pipeline's count outputs.
when_to_use_negative:
- Input is still raw FASTQ needing adaptor trimming (fastx_clipper / fastq_to_fasta)
  rather than already-clipped fasta libraries
- The goal is genome-level filtering (mapping reads to the genome file with 0 mismatches
  using patman) rather than length/abundance/complexity/t-rRNA filtering counts
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: miRPursuit
  role: Pipeline whose Filtering stage runs the UEA sRNA workbench Filter and stores
    per-library read statistics in the count directory
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: Java
  role: Runtime required to execute the UEA sRNA workbench Filter tool used for the
    filtering being tabulated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:dd00bb51e34f1c1cdf2f4a972f5350580d31572d60d50fd12e325875185c48fb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/forestbiotech-lab/miRPursuit
  tools:
  - Java
  - miRPursuit
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# srna-library-count-tabulation

## Summary

Reconstructs the miRPursuit Filtering-stage step in which the number of sRNA sequences kept per library is tabulated after UEA sRNA workbench Filter processing (length 18-26 nt, abundance >=5, low-complexity and t/rRNA removal), producing a per-library kept-counts table.

## When to use

Use when fasta sRNA libraries (e.g. test_dataset-1.fa, test_dataset-2.fa) have passed through the Wbench Filter step and you need per-library counts of retained sequences to validate filtering and populate the pipeline's count outputs.

## When NOT to use

- Input is still raw FASTQ needing adaptor trimming (fastx_clipper / fastq_to_fasta) rather than already-clipped fasta libraries
- The goal is genome-level filtering (mapping reads to the genome file with 0 mismatches using patman) rather than length/abundance/complexity/t-rRNA filtering counts

## Inputs

- fasta sRNA libraries from INSERTS_DIR sharing a common name string with sequential numbering (e.g. test_dataset-1.fa, test_dataset-2.fa)
- wbench_filter.cfg / config/workbench_filter_*.cfg with FILTER_SUF=18_26_5 filter settings
- Filter tool discard log output

## Outputs

- Filtered fasta sRNA libraries (18-26 nt, abundance >=5, t/rRNA- and low-complexity-free) in the working directory
- filter_kept_counts.tsv table of sequences kept per library
- per-library read statistics stored in the pipeline count directory ([workdir]/counts)

## How to apply

Run the UEA sRNA workbench Filter tool on each input fasta sRNA library loaded from INSERTS_DIR, using the broad-range defaults in wbench_filter.cfg: min_length=18, max_length=26, min_abundance=5, filter_low_comp=true, filter_invalid=true, trrna=true with trrna_sense_only=false, and genome-hit/normalized-abundance/kill-list filtering disabled (filter_genome_hits=false, filter_norm_abund=false, filter_kill_list=false). Write the filtered fasta libraries to the working directory using the FILTER_SUF=18_26_5 suffix convention from config/workbench_filter_*.cfg. Then count the sequences retained per library (matching the shared library-name string with sequential numbering, e.g. Lib_1, Lib_2) and tabulate them, saving the table as filter_kept_counts.tsv. Rationale: miRPursuit runs libraries in parallel and stores read statistics in a count directory so filtering outcomes can be compared across libraries and validated against the filter discard log.

## Related tools

- **miRPursuit** (Pipeline whose Filtering stage runs the UEA sRNA workbench Filter and stores per-library read statistics in the count directory) — https://github.com/forestbiotech-lab/miRPursuit
- **Java** (Runtime required to execute the UEA sRNA workbench Filter tool used for the filtering being tabulated)

## Examples

```
bash pathToMiRPursuit/miRPursuit.sh -s 1 -f 1 -l 2 --fasta test_dataset-
```

## Evaluation signals

- Each filtered library contains only sequences within the 18-26 nt length range
- All retained sequences have abundance >=5 (min_abundance=5 from wbench_filter.cfg)
- t/rRNA reads and low-complexity/invalid reads are absent from the filtered libraries
- Per-library kept counts in filter_kept_counts.tsv are consistent with the Filter tool's discard log
- Filtered filenames follow the FILTER_SUF=18_26_5 suffix convention from config/workbench_filter_*.cfg

## Limitations

- The Filter step is applied per library in parallel; counts must be tracked per library, and the count files are space-separated rather than true TSV
- Filtering parameters must match the FILTER_SUF convention; using different settings requires changing the selected workbench_filter_*.cfg, otherwise downstream counts will not be comparable
- Stats/count collection and some report tables are documented as incomplete (TODO) in the pipeline documentation

## Evidence

- [readme] Read statistics stored in count directory: [evidence span withheld — anchor status: external_doc]
- [readme] Count file naming format: [evidence span withheld — anchor status: external_doc]
