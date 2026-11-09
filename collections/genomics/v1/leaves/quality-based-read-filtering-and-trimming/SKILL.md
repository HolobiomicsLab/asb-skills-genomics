---
name: quality-based-read-filtering-and-trimming
description: Use when starting from base-called curlcake/direct RNA FASTQ reads for
  paired modified vs. unmodified conditions and you need cleaned reads aligned to
  synthetic reference transcripts so per-site mismatch/deletion/quality features can
  later be extracted.
when_to_use_negative:
- Input is raw FAST5 signal/squiggle data — EpiNano works on base-called features
  without squiggle realignment or manipulation of raw current intensity data
- Only a single sample is available with no matched modified/unmodified pair — error-based
  modification calling is designed for pairwise (WT vs KO/KD or modified vs unmodified)
  comparison
- Reads are already aligned and a per-site feature table exists — skip directly to
  error-feature extraction and SVM/Error prediction
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: NanoFilt
  role: Quality-score filtering plus headcrop/tailcrop trimming of base-called direct
    RNA FASTQ reads
- name: minimap2
  role: Aligns filtered ONT reads to the four synthetic reference transcripts using
    the '-ax map-ont' preset
- name: samtools
  role: Converts SAM alignments to sorted BAM for downstream per-site error-feature
    extraction
- name: EpiNano
  role: Downstream toolkit that consumes per-k-mer base-calling error features (mismatch,
    insertion, deletion, quality) from the mapped reads
  repo: https://github.com/novoalab/EpiNano
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:263d75ef83b0d0dd5919b9bba22bb6c72a8d7a6aff5fe0aab0a3a71a357feda7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - NanoFilt
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Quality-based read filtering and trimming

## Summary

Filter and trim base-called ONT direct RNA FASTQ reads (NanoFilt with quality-score filtering plus headcrop/tailcrop trimming) before aligning them to reference transcripts with minimap2, producing sorted BAMs that enable per-site comparison of base-called error features between m6A-modified and unmodified conditions.

## When to use

Use when starting from base-called curlcake/direct RNA FASTQ reads for paired modified vs. unmodified conditions and you need cleaned reads aligned to synthetic reference transcripts so per-site mismatch/deletion/quality features can later be extracted.

## When NOT to use

- Input is raw FAST5 signal/squiggle data — EpiNano works on base-called features without squiggle realignment or manipulation of raw current intensity data
- Only a single sample is available with no matched modified/unmodified pair — error-based modification calling is designed for pairwise (WT vs KO/KD or modified vs unmodified) comparison
- Reads are already aligned and a per-site feature table exists — skip directly to error-feature extraction and SVM/Error prediction

## Inputs

- Base-called ONT direct RNA FASTQ reads for the m6A-modified IVT synthetic construct
- Base-called ONT direct RNA FASTQ reads for the unmodified IVT synthetic construct
- FASTA file of the four synthetic reference transcripts

## Outputs

- Quality-filtered and trimmed FASTQ files per condition
- SAM alignments from minimap2 against the four synthetic references
- Sorted BAM files (one per condition) containing reads mapped to the synthetic references

## How to apply

Load the deposited base-called FASTQ reads for both the m6A-modified and unmodified IVT synthetic constructs, then apply NanoFilt with quality-score filtering plus headcrop and tailcrop to trim adapter/leader and trailing regions. Align the filtered reads against the four synthetic reference sequences using minimap2 with the '-ax map-ont' preset, which is the correct preset for Oxford Nanopore direct RNA reads. Convert the resulting SAM to BAM and sort with samtools so downstream per-site error-feature extraction (mismatch, deletion, insertion frequencies and per-base quality) is positionally addressable. The rationale is that EpiNano detects m6A as systematic base-calling 'errors' in base-called features, so reads must be cleaned and mapped per condition before modified/unmodified sites can be compared pairwise. Keep both conditions symmetric: the same filter/trim parameters must be applied to each sample so error-pattern differences reflect modification, not preprocessing.

## Related tools

- **NanoFilt** (Quality-score filtering plus headcrop/tailcrop trimming of base-called direct RNA FASTQ reads)
- **minimap2** (Aligns filtered ONT reads to the four synthetic reference transcripts using the '-ax map-ont' preset)
- **samtools** (Converts SAM alignments to sorted BAM for downstream per-site error-feature extraction)
- **EpiNano** (Downstream toolkit that consumes per-k-mer base-calling error features (mismatch, insertion, deletion, quality) from the mapped reads) — https://github.com/novoalab/EpiNano

## Examples

```
minimap2 -ax map-ont synthetic_refs.fa filtered_reads.fastq | samtools sort -o mapped.sorted.bam
```

## Evaluation signals

- A sorted BAM file exists for each condition (modified and unmodified) after samtools sort
- samtools flagstat/log shows a substantial fraction of reads successfully mapped to the synthetic references in every condition
- Identical NanoFilt quality/headcrop/tailcrop parameters were recorded and applied to both conditions
- Per-site mismatch/deletion frequencies can be computed at reference positions in both conditions, enabling pairwise per-site comparison
- Alignment was performed with the '-ax map-ont' preset rather than an Illumina-oriented preset (checkable from the command or @PG line)

## Limitations

- Direct RNA sequencing base-calling produces a significant amount of 'errors' even in unmodified sequences, so a matched control dataset is highly recommended for confident calls
- EpiNano has no per-read resolution; predictions aggregate information from all reads mapping to a site, so mapping quality directly conditions site-level accuracy
- Only ~32% of known modification sites are recovered by the method, so failure to see a signal at a site does not prove the site is unmodified
- EpiNano does not distinguish between different types of RNA modifications (e.g., m1A vs m6A)

## Evidence

- [readme] EpiNano relies on the use of base-calling 'errors' to detect RNA modifications; however, direct RNA sequencing base-calling produces a significant amount of 'errors' in unmodified sequences.: [evidence span withheld — anchor status: external_doc]
- [abstract] m6A RNA modifications can be detected via direct RNA sequencing as systematic errors and decreased base-calling qualities.: "N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
- [discussion] RNA modifications can be identified as systematic and reproducible base-calling 'errors' in direct RNA sequencing data.: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [discussion] EpiNano works from base-called features rather than raw signal.: "here we provide a novel strategy to identify RNA modifications from base-called features, without the need of squiggling realignments or manipulation of raw current intensity data sets."
- [discussion] The algorithm aggregates evidence across reads mapped to a site, making the mapping step load-bearing.: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
