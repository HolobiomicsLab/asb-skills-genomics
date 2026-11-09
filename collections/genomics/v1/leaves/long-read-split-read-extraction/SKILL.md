---
name: long-read-split-read-extraction
description: Use when you have a PacBio/ONT long-read bam or cram aligned to a reference
  and need to recover split-read signatures and insertion sequences for candidate
  non-reference TE (or any insertion) calls.
when_to_use_negative:
- The input is short-read data — sTELLeR is designed and tested for long-read (PacBio/ONT)
  alignments, so split-read TE signatures will not be recoverable.
- You need reference-present TE genotyping only — this skill targets non-reference
  insertions absent from the reference.
- Phasing/haplotype assignment of TE insertions is not needed and VCF-compatible output
  is not required, e.g. quick exploratory screens.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: sTELLeR
  role: End-to-end caller implementing split-read extraction, DBSCAN clustering, minimap2
    TE alignment, consensus refinement, and VCF output
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: pysam
  role: Parses the input bam to extract split reads and insertion sequences spanning
    candidate TE insertions into per-chromosome fasta intermediates
- name: minimap2
  role: Aligns clustered candidate insertion sequences to the TE fasta (produces ID_chr_repeats.sam)
- name: samtools
  role: Dependency used by sTELLeR for bam/alignment handling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:d8b50626579dd62063c3a0e969a178abc1a4150de9ebbc2b14d539c2e75c80eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# long-read-split-read-extraction

## Summary

Extract split reads and insertion sequences spanning candidate non-reference transposable element insertions from a long-read alignment (bam/cram) using pysam, producing per-chromosome candidate fasta intermediates that downstream TE clustering and alignment steps consume.

## When to use

Use when you have a PacBio/ONT long-read bam or cram aligned to a reference and need to recover split-read signatures and insertion sequences for candidate non-reference TE (or any insertion) calls.

## When NOT to use

- The input is short-read data — sTELLeR is designed and tested for long-read (PacBio/ONT) alignments, so split-read TE signatures will not be recoverable.
- You need reference-present TE genotyping only — this skill targets non-reference insertions absent from the reference.
- Phasing/haplotype assignment of TE insertions is not needed and VCF-compatible output is not required, e.g. quick exploratory screens.

## Inputs

- bam or cram file of long reads aligned to the reference (e.g. testdata.bam, --style pb or ont)
- reference genome fasta used for the alignment (e.g. HG38_chr22.fa)
- fasta file with insertion/TE sequences to detect (--TE_fasta, e.g. fasta/TEfastasequences.fa)

## Outputs

- per-chromosome candidate-insertion fasta intermediates (ID_chr_reads.fasta)
- candidate insertions aligned to TEs (ID_chr_repeats.sam)
- final VCF of TE calls (INS:ME variants; format TE:TS:HT:GT), e.g. testdata_res_repeats.vcf

## How to apply

Parse the input bam with pysam to extract split reads and insertion sequences spanning candidate TE insertions, writing per-chromosome candidate fasta intermediates (ID_chr_reads.fasta). Retain reads whose mapping quality meets the sTELLeR threshold (--mq, default 20) so only confidently aligned split reads seed candidates. These extracted candidate sequences are then clustered (DBSCAN) and aligned against the TE fasta (e.g. TEfastasequences.fa) with minimap2, producing ID_chr_repeats.sam intermediates. Keep intermediates only with --keep_intermediates; otherwise they are removed. Calls require a minimum of --sr supporting split reads/insertions (default 3), are capped at --maxreads (default 100) to avoid high-coverage regions, and are written to a VCF with variants named INS:ME and format fields TE:TS:HT:GT.

## Related tools

- **sTELLeR** (End-to-end caller implementing split-read extraction, DBSCAN clustering, minimap2 TE alignment, consensus refinement, and VCF output) — https://github.com/kristinebilgrav/sTELLeR
- **pysam** (Parses the input bam to extract split reads and insertion sequences spanning candidate TE insertions into per-chromosome fasta intermediates)
- **minimap2** (Aligns clustered candidate insertion sequences to the TE fasta (produces ID_chr_repeats.sam))
- **samtools** (Dependency used by sTELLeR for bam/alignment handling)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- Run on the sTELLeR_supplementary test data (HG38_chr22.fa, TEfastasequences.fa, testdata.bam, --style pb) and confirm exactly 7 variants are identified in the output VCF.
- Diff the produced testdata_res_repeats.vcf against the provided testbamTRUTH.txt truth set; TE positions should match.
- Every reported variant must carry the minimum supporting split reads (--sr, default 3; example runs use 4) and TE-alignment mapping quality >= 20 (--mq).
- Check VCF structure: variant names INS:ME and format fields TE:TS:HT:GT, with HT=0 and GT 1/0 or 1/1 for unphased input.
- Verify that --maxreads filtering excludes calls in regions exceeding the coverage cap (default 100; example uses 80).

## Limitations

- The --sr threshold trades sensitivity against coverage: supporting reads depend on sample coverage and desired sensitivity (default 3).
- High-coverage regions can generate spurious calls unless --maxreads is lowered (default 100) to cap supporting reads.
- If input is unphased, haplotag is 0 and genotype is coarser (1/0 vs 1/1) than phased output (1|0, 0|1, 1|1).
- Intermediate files (ID_chr_reads.fasta, ID_chr_repeats.sam) are deleted unless --keep_intermediates is set, which can hinder debugging.

## Evidence

- [methods] pysam-based split-read extraction: [evidence span withheld — anchor status: not_found]
- [methods] filtering defaults: [evidence span withheld — anchor status: not_found]
- [methods] validation against truth set: "a total of 7 variants should be identified"
- [readme] output format: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
- [intro] input requirements: "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)"
