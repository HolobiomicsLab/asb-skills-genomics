---
name: read-count-validation-against-reported-metrics
description: Use when you have MiniBar-demultiplexed per-sample Nanopore fastq files
  from the long rDNA barcoding pipeline and need to reproduce or verify the quality/length
  filtering step and its reported read yield.
when_to_use_negative:
- Input is raw MinION pore-runner output that has not yet been demultiplexed by MiniBar
  — run minibar.py with -T -F first to obtain per-sample files
- Input consists of short Illumina reads from the paper's mock-community comparison,
  where a 3 kb minimum-length threshold is inappropriate
- Input is already consensus-called or collapsed rDNA sequences, for which read-level
  quality/length filtering has no meaning
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0196
tools:
- name: NanoFilt
  role: Applies the -q 13 quality and -l 3000 length thresholds to each demultiplexed
    sample fastq
- name: MiniBar
  role: Produces the per-sample demultiplexed fastq inputs (via -F binning with -T
    trimming) that this skill filters
  repo: https://github.com/calacademy-research/minibar
- name: Python
  role: Used to count fastq records in the filtered files and sum counts across samples
    for validation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:eb2bfadc6e8cb44bbf98fa3398321febbd6c772666861815e26df4f89c8f0dfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - MiniBar
  - NanoFilt
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Read count validation against reported metrics

## Summary

Quality- and length-filter demultiplexed Nanopore reads with NanoFilt (minimum average Phred quality 13, minimum length 3000 bp) and validate the pipeline by checking that summed per-sample read counts match the paper's reported yield of 245,433 filtered reads for the main run.

## When to use

Use when you have MiniBar-demultiplexed per-sample Nanopore fastq files from the long rDNA barcoding pipeline and need to reproduce or verify the quality/length filtering step and its reported read yield.

## When NOT to use

- Input is raw MinION pore-runner output that has not yet been demultiplexed by MiniBar — run minibar.py with -T -F first to obtain per-sample files
- Input consists of short Illumina reads from the paper's mock-community comparison, where a 3 kb minimum-length threshold is inappropriate
- Input is already consensus-called or collapsed rDNA sequences, for which read-level quality/length filtering has no meaning

## Inputs

- Per-sample demultiplexed fastq files produced by minibar.py -F binning (default prefix sample_)
- Unassigned-read fastq file (*_unk) from MiniBar demultiplexing
- Barcode/index combination file used for demultiplexing (e.g. Demultiplex.txt)

## Outputs

- One quality-filtered fastq file per sample (reads with Phred >= 13 and length >= 3000 bp)
- Per-sample read counts and a summed total across samples (expected ~245,433 for the main run)

## How to apply

Start from the per-sample fastq files produced by running minibar.py with the -F binning option (unassigned reads land in the *_unk file and should be tracked separately). Pipe each sample file through NanoFilt with -q 13 (discard reads with average Phred quality below 13) and -l 3000 (discard reads shorter than 3000 bp, matching the multi-kilobase rDNA amplicon), writing each sample's retained reads to a quality-filtered fastq file. Because the target amplicon is a multi-kilobase rDNA fragment, the 3000 bp cutoff is deliberately lenient enough to keep full-length inserts while removing short Nondopore byproducts. Count fastq records in each filtered file (e.g. with a short Python script counting records) and sum across samples. Correct application is judged by whether the summed count reproduces the reported 245,433 quality-filtered reads for the main Nanopore run.

## Related tools

- **NanoFilt** (Applies the -q 13 quality and -l 3000 length thresholds to each demultiplexed sample fastq)
- **MiniBar** (Produces the per-sample demultiplexed fastq inputs (via -F binning with -T trimming) that this skill filters) — https://github.com/calacademy-research/minibar
- **Python** (Used to count fastq records in the filtered files and sum counts across samples for validation)

## Examples

```
cat sample_Jun_40.fastq | NanoFilt -q 13 -l 3000 > Jun_40.filtered.fastq; python -c "import sys;print(sum(1 for _ in open('Jun_40.filtered.fastq'))//4)"
```

## Evaluation signals

- Sum of filtered read counts across all samples equals the reported 245,433 reads for the main Nanopore run
- Every retained record in the filtered fastq has length >= 3000 bp and average Phred quality >= 13
- Each filtered file contains well-formed fastq records (4 lines per record, matching headers)
- Reads assigned to no sample remain in the *_unk file and are accounted for separately from per-sample totals
- Filtered file sizes are substantially smaller than the pre-filter demultiplexed files, consistent with removal of short/low-quality reads

## Limitations

- The 245,433 figure is specific to the paper's main Nanopore run; other runs or subsets will yield different totals, so the invariant is exact reproduction only for that dataset
- Filtering is applied to MiniBar-assigned reads only; unassigned (*_unk) reads are not part of the per-sample count validation
- The paper notes further research is needed before long rDNA amplicon sequencing is reliable for bulk-sample analysis, so read-yield validation does not by itself confirm taxonomic recovery

## Evidence

- [readme] Typically you'll want to group your records by identified sample. This can be done by using the -F option: "Typically you'll want to group your records by identified sample. This can be done by using the -F option, optionally with -P to use a file prefix other than the "sample_" default."
- [methods] Identify MinION sequence by dual barcode indexes and primers: "Identify MinION sequence by dual barcode indexes and primers."
- [discussion] MinION dual indexed rDNA amplicon sequencing summary: "Sequencing long dual indexed rDNA amplicons on ONT's MinION is a simple, cost-effective, accurate, and universal approach for eukaryote DNA barcoding."
