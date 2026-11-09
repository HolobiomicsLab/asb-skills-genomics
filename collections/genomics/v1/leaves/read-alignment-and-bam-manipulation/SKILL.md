---
name: read-alignment-and-bam-manipulation
description: Use when you have long-read sequencing data (e.g. PBSIM3-simulated PacBio
  RS II reads or real ONT/PacBio reads) plus a reference genome FASTA and need a sorted
  BAM as input for insertion/variant calling.
when_to_use_negative:
- Input is already an aligned, sorted, indexed BAM or CRAM — proceed directly to sTELLeR
  instead of re-aligning.
- Input is short Illumina reads, since sTELLeR and this workflow are designed and
  tested for long-read (PacBio/ONT) data.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns long reads to the reference genome, producing the SAM alignments that
    become the caller input BAM
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: samtools
  role: Sorts and indexes the alignment file into the BAM format required by sTELLeR
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: pysam
  role: Python dependency sTELLeR uses to read the BAM and reference during TE calling
  repo: https://github.com/kristinebilgrav/sTELLeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:f9941f20dd69c08f14264ff228cfec856d1ae74cd217973843a0cdb02ac95238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - PBSIM3
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# read-alignment-and-bam-manipulation

## Summary

Aligns long reads (PacBio or ONT) to a reference genome with minimap2 and produces a sorted, indexed BAM with samtools, yielding the caller-ready alignment file required by sTELLeR for TE insertion detection.

## When to use

Use when you have long-read sequencing data (e.g. PBSIM3-simulated PacBio RS II reads or real ONT/PacBio reads) plus a reference genome FASTA and need a sorted BAM as input for insertion/variant calling.

## When NOT to use

- Input is already an aligned, sorted, indexed BAM or CRAM — proceed directly to sTELLeR instead of re-aligning.
- Input is short Illumina reads, since sTELLeR and this workflow are designed and tested for long-read (PacBio/ONT) data.

## Inputs

- Long-read sequencing reads (PacBio CLR/HiFi or ONT; FASTA/FASTQ, e.g. PBSIM3-simulated reads at 20x coverage)
- Reference genome FASTA (e.g. GRCh38, GCF_000001405.26 or HG38_chr22.fa)

## Outputs

- Sorted, indexed BAM file of reads aligned to the reference (e.g. testdata.bam)
- BAI index for random-access queries by sTELLeR/pysam

## How to apply

Align the long reads against the unmodified reference genome (e.g. GRCh38/GCF_000001405.26) using minimap2, which handles the high error rates typical of PacBio RS II or ONT reads. Pipe or convert the resulting SAM alignments into BAM format, then sort and index with samtools so the file supports random access by downstream tools. sTELLeR requires this BAM together with the same reference used for alignment and a TE FASTA (--TE_fasta); the --style flag should match the read technology (pb/clr/hifi or ont). Verify the BAM is valid and sorted (samtools quickcheck/index succeeds) before passing it to the caller, as sTELLeR's split-read (--sr, default 3, example 4) and mapping-quality (--mq, default 20) filters depend on correct alignment records.

## Related tools

- **minimap2** (Aligns long reads to the reference genome, producing the SAM alignments that become the caller input BAM) — https://github.com/kristinebilgrav/sTELLeR
- **samtools** (Sorts and indexes the alignment file into the BAM format required by sTELLeR) — https://github.com/kristinebilgrav/sTELLeR
- **pysam** (Python dependency sTELLeR uses to read the BAM and reference during TE calling) — https://github.com/kristinebilgrav/sTELLeR

## Examples

```
minimap2 -ax map-pb GRCh38.fa simulated_reads.fq | samtools sort -o testdata.bam && samtools index testdata.bam
```

## Evaluation signals

- samtools index completes without error and the sorted BAM passes validation (usable as sTELLeR input).
- For the benchmark workflow, the simulated reads align back to the unmodified reference at the expected ~20x coverage.
- Running sTELLeR on the resulting BAM with the test data identifies exactly 7 variants, matching testbamTRUTH.txt.
- Header/read-group metadata and --style (pb vs ont) match the technology used to generate the reads.
- Truth-table validation for the simulated benchmark: 886 Alu, 888 L1, 444 SVA, and 452 HERV insertions recorded before simulation.

## Limitations

- sTELLeR/BAM pipeline assumes long reads; error model and --style must match the platform (ont vs pb/clr/hifi) or calls will be degraded.
- The masked-reference trick used in the simulated benchmark (chr22 zip with masked parts removed) means the reference base must match the BAM or coordinates will be wrong.
- Maximum supporting reads (--maxreads, default 100) caps calls in high-coverage regions, so very deep BAMs may suppress true insertions.

## Evidence

- [readme] minimap2 role in dependencies: "Install dependencies: 
- minimap2 
- pysam
- samtools"
- [results] simulated BAM construction: [evidence span withheld — anchor status: not_found]
- [methods] alignment and sort/index step: [evidence span withheld — anchor status: not_found]
- [readme] BAM as caller input requirement: "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)"
