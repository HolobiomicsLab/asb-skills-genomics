---
name: pacbio-bam-variant-calling-benchmarking
description: Use when comparing TE insertion callers on PacBio long-read BAMs and
  you need per-caller true-positive counts scored against an assembly-based callset,
  e.g. reproducing Table 3 on HG002/HG01071.
when_to_use_negative:
- Input reads are short-read (Illumina) rather than PacBio/ONT long reads — sTELLeR
  is designed and tested for long-read data
- The goal is calling arbitrary non-TE variants rather than transposable element insertions,
  and no TE sequence FASTA is available
- No assembly-based truth callset exists to score true positives against
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: sTELLeR
  role: Primary TE insertion caller run with --style pb, --sr 4, --mr 80 on PacBio
    BAMs; emits haplotype-aware VCF output
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: TLDR
  role: Comparison TE insertion caller run on the same HG002/HG01071 BAMs for TP-count
    benchmarking
- name: xTEA
  role: Comparison TE insertion caller run on the same BAMs for TP-count benchmarking
- name: minimap2
  role: Dependency for sTELLeR; used for read alignment of long reads to the reference
- name: samtools
  role: Dependency for BAM handling in the sTELLeR workflow
- name: pysam
  role: Python dependency used by sTELLeR to parse BAM files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:62739317d3bd1a0174c03a937a67911a1a7fa647e8581bf6930abb07f3c76b93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - TLDR
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  - xTEA
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pacbio-bam-variant-calling-benchmarking

## Summary

Benchmark transposable element insertion (TEI) callers (sTELLeR, TLDR, xTEA) on GRCh38-realigned PacBio BAMs by counting true positives against an assembly-based truth callset, as done for HG002 and HG01071 in the sTELLeR paper (Table 3).

## When to use

Use when comparing TE insertion callers on PacBio long-read BAMs and you need per-caller true-positive counts scored against an assembly-based callset, e.g. reproducing Table 3 on HG002/HG01071.

## When NOT to use

- Input reads are short-read (Illumina) rather than PacBio/ONT long reads — sTELLeR is designed and tested for long-read data
- The goal is calling arbitrary non-TE variants rather than transposable element insertions, and no TE sequence FASTA is available
- No assembly-based truth callset exists to score true positives against

## Inputs

- GRCh38-realigned PacBio BAM files for samples HG002 and HG01071
- Reference genome FASTA (GRCh38/HG38) used for alignment
- TE sequence FASTA (e.g. TEfastasequences.fa with L1 and Alu sequences; TEsequences_SVA_HERV_ALU_L1.fasta to also include SVA/HERV)
- Assembly-based truth TE insertion callset (VCF/BED) for TP scoring

## Outputs

- Per-sample, per-caller TE insertion VCFs with INS:ME records (sTELLeR: *_repeats.vcf)
- caller_tp_counts.tsv: Table 3-style per-caller true-positive counts (sTELLeR 1885/1914, TLDR 1741/1808, xTEA 1543/1645 for HG002/HG01071)

## How to apply

Run sTELLeR (steller.py) on each GRCh38-realigned PacBio BAM with --style pb, --sr 4, the default mapping-quality threshold --mq 20, and --mr 80 to emit per-sample TEI VCFs. Run TLDR and xTEA on the identical BAM files with their PacBio-appropriate settings to produce comparable VCFs. Intersect each caller's INS:ME VCF records with the assembly-based truth callset to count true positives per caller per sample. Aggregate the TP counts into a per-caller table (sTELLeR 1885/1914, TLDR 1741/1808, xTEA 1543/1645 for HG002/HG01071) and write it to caller_tp_counts.tsv. sTELLeR needs, besides the BAM, the reference used for alignment and a --TE_fasta of insertion sequences to detect, e.g. TEfastasequences.fa containing L1 and Alu sequences from the supplementary repo.

## Related tools

- **sTELLeR** (Primary TE insertion caller run with --style pb, --sr 4, --mr 80 on PacBio BAMs; emits haplotype-aware VCF output) — https://github.com/kristinebilgrav/sTELLeR
- **TLDR** (Comparison TE insertion caller run on the same HG002/HG01071 BAMs for TP-count benchmarking)
- **xTEA** (Comparison TE insertion caller run on the same BAMs for TP-count benchmarking)
- **minimap2** (Dependency for sTELLeR; used for read alignment of long reads to the reference)
- **samtools** (Dependency for BAM handling in the sTELLeR workflow)
- **pysam** (Python dependency used by sTELLeR to parse BAM files)

## Examples

```
python steller/steller.py --ref GRCh38.fa --TE_fasta fasta/TEfastasequences.fa --bam HG002.pb.bam --sr 4 --style pb -o HG002_res -mr 80
```

## Evaluation signals

- sTELLeR TP count on HG002 reproduces the reported 1885 true positives within tolerance (and 1914 on HG01071)
- caller_tp_counts.tsv contains exactly one TP-count row per caller (sTELLeR, TLDR, xTEA)
- TP counts rank as sTELLeR (1885/1914) > TLDR (1741/1808) > xTEA (1543/1645) against the assembly-based callset
- sTELLeR output VCF records are named INS:ME with TE type, transposon start/end in INFO, and TE:TS:HT:GT in FORMAT
- On the supplementary test dataset (testdata.bam), sTELLeR identifies a total of 7 variants matching testbamTRUTH.txt

## Limitations

- Minimum supporting reads (--sr) must be tuned to sample coverage and desired sensitivity; default is 3, benchmark used 4
- --maxreads (default 100) caps calls in regions exceeding the given coverage; benchmark used 80
- If input is not phased, haplotag is 0 and genotype is unphased (1/0 or 1/1), so haplotype-aware evaluation is not possible
- sTELLeR benchmarking here covers Alu/L1 (and optionally SVA/HERV) elements; sensitivity claims are particularly for Alu elements

## Evidence

- [results] Against the assembly-based callset, sTELLeR called 1885 true positive TEI in HG002 and 1914 in HG01071, exceeding TLDR (1741/1808) and xTEA (1543/1645).: [evidence span withheld — anchor status: not_found]
- [readme] sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta): "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)"
- [readme] The output is given in a VCF file, which is compatible to merge with VCF files from other variant callers.: "The output is given in a [VCF file](https://samtools.github.io/hts-specs/VCFv4.2.pdf), which is compatible to merge with VCF files from other variant callers."
- [abstract] Particularly, sTELLeR exhibit higher precision and sensitivity for calling of Alu elements than similar tools.: "Particularly, sTELLeR exhibit higher precision and sensitivity for calling of Alu elements than similar tools."
- [methods] -r , --sr Minimum number of supporting split reads/insertions to call a variant (default 3): "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
