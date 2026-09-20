---
name: per-caller-sensitivity-counting
description: Use when comparing sensitivity of TE insertion callers (sTELLeR, TLDR,
  xTEA) on long-read BAMs by counting true positives against an assembly-based callset,
  as in the sTELLeR paper's Table 3.
when_to_use_negative:
- Input reads are short-read (Illumina) data — sTELLeR is designed and tested for
  long-read PacBio or ONT data and is run with --style pb or --style ont
- The goal is de novo TE discovery rather than benchmarking caller sensitivity against
  an existing assembly-based truth set
- Truth is a small test truth file (testbamTRUTH.txt with 7 expected variants) intended
  only for installation verification, not a per-caller sensitivity benchmark
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sTELLeR
  role: Primary TE insertion caller run with --style pb, --sr 4, --mq 20 (default),
    --mr 80 to produce INS:ME VCFs
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: TLDR
  role: Comparison TE caller run on the same BAM files with PacBio-appropriate settings
- name: xTEA
  role: Comparison TE caller run on the same BAM files
- name: pysam
  role: Python dependency of sTELLeR; useful for parsing/intersecting VCF records
    against the truth callset
- name: samtools
  role: Dependency for BAM handling and realignment validation
- name: minimap2
  role: Dependency used for aligning long reads to GRCh38
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:b5d4ee7ae8d58c9356f0d92cbdc33ce5682003192e64c4a5b040af1c11e34e13
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
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# per-caller-sensitivity-counting

## Summary

Benchmark transposable element insertion (TEI) callers by intersecting each caller's VCF records with an assembly-based truth callset and counting true positives per caller per sample. Reproduces Table 3-style comparisons showing sTELLeR outperforming TLDR and xTEA on PacBio data.

## When to use

Use when comparing sensitivity of TE insertion callers (sTELLeR, TLDR, xTEA) on long-read BAMs by counting true positives against an assembly-based callset, as in the sTELLeR paper's Table 3.

## When NOT to use

- Input reads are short-read (Illumina) data — sTELLeR is designed and tested for long-read PacBio or ONT data and is run with --style pb or --style ont
- The goal is de novo TE discovery rather than benchmarking caller sensitivity against an existing assembly-based truth set
- Truth is a small test truth file (testbamTRUTH.txt with 7 expected variants) intended only for installation verification, not a per-caller sensitivity benchmark

## Inputs

- GRCh38-realigned PacBio BAM files for samples HG002 and HG01071
- Reference genome FASTA (GRCh38)
- TE sequence FASTA (e.g. TEfastasequences.fa with Alu and L1 sequences, or TEsequences_SVA_HERV_ALU_L1.fasta)
- Assembly-based truth callset of TE insertions

## Outputs

- Per-sample TE insertion VCFs from each caller (INS:ME records)
- Per-caller true-positive count table (e.g. caller_tp_counts.tsv: sTELLeR 1885/1914, TLDR 1741/1808, xTEA 1543/1645)

## How to apply

Run each caller on the same GRCh38-realigned PacBio BAM files: sTELLeR via steller.py with --style pb, --sr 4, --mr 80 and default mapping-quality threshold 20; TLDR and xTEA with their PacBio-appropriate settings. Collect each caller's VCF, filtering for INS:ME records. Intersect each caller's INS:ME entries with the assembly-based truth callset to count true positives per caller per sample. Aggregate counts into a per-caller table; the reference result is sTELLeR 1885/1914 TP, TLDR 1741/1808, xTEA 1543/1645 for HG002/HG01071. sTELLeR's output VCF names variants INS:ME with TE type in the INFO field and TE:TS:HT:GT in FORMAT, which is compatible for merging with other callers' VCFs.

## Related tools

- **sTELLeR** (Primary TE insertion caller run with --style pb, --sr 4, --mq 20 (default), --mr 80 to produce INS:ME VCFs) — https://github.com/kristinebilgrav/sTELLeR
- **TLDR** (Comparison TE caller run on the same BAM files with PacBio-appropriate settings)
- **xTEA** (Comparison TE caller run on the same BAM files)
- **pysam** (Python dependency of sTELLeR; useful for parsing/intersecting VCF records against the truth callset)
- **samtools** (Dependency for BAM handling and realignment validation)
- **minimap2** (Dependency used for aligning long reads to GRCh38)

## Examples

```
python steller/steller.py --ref GRCh38.fa --TE_fasta TEfastasequences.fa --bam HG002.pb.bam --sr 4 --style pb -o HG002_steller -mr 80
```

## Evaluation signals

- Reproduced sTELLeR true-positive count on HG002 matches the reported 1885 (and 1914 on HG01071) within tolerance
- Output table contains exactly one true-positive-count row per caller (sTELLeR, TLDR, xTEA) per sample
- Each caller VCF record counted as a TP is an INS:ME entry overlapping the assembly-based truth callset
- Relative ordering holds: sTELLeR > TLDR > xTEA in true positives on both HG002 and HG01071 PacBio data
- On the supplementary test dataset, sTELLeR identifies the expected 7 variants vs testbamTRUTH.txt, confirming a correct install before benchmarking

## Limitations

- True-positive counts depend on the assembly-based truth callset; callers may differ in which TE types they detect (sTELLeR exhibits higher precision and sensitivity particularly for Alu elements)
- Maximum supporting reads (--mr, default 100) caps calls in high-coverage regions; --sr 4 was used here rather than the default 3, affecting sensitivity
- Haplotag is 0 and genotype is unphased (1/0 or 1/1) when input BAMs are not phased, limiting haplotype-aware comparisons

## Evidence

- [other] Run sTELLeR (steller.py) on each GRCh38-realigned PacBio BAM with --style pb, --sr 4, default mapping-quality threshold 20, and --mr 80: [evidence span withheld — anchor status: not_found]
- [other] sTELLeR called 1885 true positive TEI in HG002 and 1914 in HG01071, exceeding TLDR (1741/1808) and xTEA (1543/1645).: [evidence span withheld — anchor status: not_found]
- [methods] python steller/steller.py --ref < genome ref_file > --TE_fasta < TE sequence fasta file > --bam < bamfile > --sr 4 --style pb -o < output prefix > -mr 80: "python steller/steller.py --ref < genome ref_file > --TE_fasta < TE sequence fasta file > --bam < bamfile > --sr 4 --style pb -o < output prefix > -mr 80"
- [readme] The output is given in a VCF file, which is compatible to merge with VCF files from other variant callers.: "The output is given in a [VCF file](https://samtools.github.io/hts-specs/VCFv4.2.pdf), which is compatible to merge with VCF files from other variant callers."
