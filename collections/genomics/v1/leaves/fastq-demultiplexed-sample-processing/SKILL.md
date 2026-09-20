---
name: fastq-demultiplexed-sample-processing
description: Use when MiniBar (-F binning) has produced per-sample FASTQ files of
  MinION rDNA amplicon reads and they must be cleaned of low-quality, short reads
  before consensus/barcode analysis.
when_to_use_negative:
- Input reads have not yet been demultiplexed — run minibar.py with the -F option
  first so records are grouped by identified sample
- Data are short Illumina reads or short amplicons, where the 3,000 bp length cutoff
  (sized to the multi-kilobase rDNA amplicon) would discard everything
- Input is already a filtered/consensus FASTA or a feature table rather than raw demultiplexed
  Nanopore FASTQ
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: NanoFilt
  role: Applies the -q 13 average-quality and -l 3000 length thresholds to each demultiplexed
    sample FASTQ
- name: MiniBar
  role: Upstream step that demultiplexes dual-indexed MinION reads and bins them into
    individual sample files via -F; README notes Python 2.7/3 and the Edlib module
    as requirements
  repo: https://github.com/calacademy-research/minibar
- name: Python
  role: Short script used to count FASTQ records in the filtered files and sum them
    across samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:920c2c9df9cfb484beeadfaaf43e2f604ff31a844b8a70581452c75e03c171de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - MiniBar
  - NanoFilt
  - Python
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fastq-demultiplexed-sample-processing

## Summary

Quality- and length-filters MiniBar-demultiplexed Nanopore FASTQ reads of long rDNA amplicons with NanoFilt (Phred Q>13, length >3 kb), producing per-sample cleaned FASTQ files whose combined read count for the main run should equal 245,433 reads.

## When to use

Use when MiniBar (-F binning) has produced per-sample FASTQ files of MinION rDNA amplicon reads and they must be cleaned of low-quality, short reads before consensus/barcode analysis.

## When NOT to use

- Input reads have not yet been demultiplexed — run minibar.py with the -F option first so records are grouped by identified sample
- Data are short Illumina reads or short amplicons, where the 3,000 bp length cutoff (sized to the multi-kilobase rDNA amplicon) would discard everything
- Input is already a filtered/consensus FASTA or a feature table rather than raw demultiplexed Nanopore FASTQ

## Inputs

- Per-sample demultiplexed FASTQ files produced by minibar.py with the -F binning option (e.g. sample_<SampleID>.fastq)
- Unassigned reads FASTQ file (*_unk) from MiniBar binning

## Outputs

- One quality-filtered FASTQ file per sample containing only reads with average Phred quality >13 and length >3,000 bp
- Per-sample and summed read counts (expected sum: 245,433 reads for the main run)

## How to apply

Start from the per-sample FASTQ files emitted by MiniBar's -F binning option (unassigned reads land in the *_unk file and are kept separate). For each sample file, pass reads through NanoFilt with -q 13, which discards reads whose average Phred quality falls below 13, and -l 3000, which discards reads shorter than 3,000 bp — a cutoff matched to the multi-kilobase rDNA amplicon sequenced on the MinION. Write each sample's retained reads to a quality-filtered FASTQ file. Count FASTQ records per file (e.g. a short Python script) and sum across samples; the total for the main Nanopore run must reproduce the reported 245,433 quality-filtered reads. Rationale: single-pass Nanopore reads of long rDNA amplicons carry errors and the pipeline relies on length to confirm full-length rDNA inserts, so aggressive Q and length cutoffs are the downstream gate before trimming-based consensus generation.

## Related tools

- **NanoFilt** (Applies the -q 13 average-quality and -l 3000 length thresholds to each demultiplexed sample FASTQ)
- **MiniBar** (Upstream step that demultiplexes dual-indexed MinION reads and bins them into individual sample files via -F; README notes Python 2.7/3 and the Edlib module as requirements) — https://github.com/calacademy-research/minibar
- **Python** (Short script used to count FASTQ records in the filtered files and sum them across samples)

## Examples

```
NanoFilt -q 13 -l 3000 < sample_Jun_40.fastq > sample_Jun_40.filtered.fastq
```

## Evaluation signals

- Every record in each output FASTQ has sequence length >= 3,000 bp
- Every retained record's average Phred quality score exceeds 13
- Sum of FASTQ record counts across all filtered sample files equals 245,433 for the main Nanopore run
- A filtered file exists for every sample FASTQ emitted by minibar.py -F, and the *_unk (unassigned) file is filtered separately rather than merged into sample files
- Output remains valid FASTQ (4-line records, headers preserved from MiniBar output)

## Limitations

- The 3,000 bp threshold is calibrated to the multi-kilobase rDNA amplicon; applying it to shorter amplicons would remove nearly all reads
- Filtering is quality/length based only — it does not correct Nanopore errors; consensus generation happens downstream in the pipeline
- Reads MiniBar could not assign (in the *_unk file) are excluded from per-sample filtering and counting

## Evidence

- [methods] Nanofilt quality and length thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Expected read yield after filtering: [evidence span withheld — anchor status: not_found]
- [readme] Per-sample FASTQ inputs from MiniBar binning: "Typically you'll want to group your records by identified sample. This can be done by using the -F option"
- [readme] Input format accepted downstream of demultiplexing: "The sequence file can be in Fasta or Fastq format, gzipped or plain text."
- [abstract] Position of filtering in the overall pipeline: "we assemble a pipeline for long rDNA barcode analysis"
