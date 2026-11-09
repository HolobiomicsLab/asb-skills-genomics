---
name: long-read-transposable-element-calling
description: Use when you have a long-read BAM/CRAM aligned to a reference genome
  and need to detect non-reference TE insertions, or when benchmarking sTELLeR against
  TLDR/xTEA on PacBio data.
when_to_use_negative:
- Input is short-read sequencing data — sTELLeR is designed and tested for PacBio/ONT
  long reads; use short-read TE callers such as RetroSeq or MELT2 instead
- No reference-aligned long-read BAM/CRAM is available or the reference used for alignment
  cannot be provided — sTELLeR requires both the read file and the exact reference
  it was aligned to
- Only reference-resident TE annotation is needed — sTELLeR detects non-reference
  insertions, not reference TE genotyping against a reference callset alone
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sTELLeR
  role: Primary TE insertion caller run via steller.py on long-read BAMs
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Dependency used to align candidate insertion sequences to the TE FASTA
- name: pysam
  role: Python dependency for BAM/reading support within sTELLeR
- name: samtools
  role: Dependency for BAM handling during candidate extraction
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:4b4c2e93de740f2ee51cd9f71214a1dbf4855b96f577ff0bea23d2f524eb9b7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - MELT2
  - RetroSeq
  - TLDR
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# long-read-transposable-element-calling

## Summary

Detect non-reference transposable element insertions (Alu, L1, HERV, SVA) in long-read (PacBio/ONT) genomes using sTELLeR, a fast, haplotype-aware caller that outputs VCF records (INS:ME) compatible with merging against other TE callers such as TLDR and xTEA.

## When to use

Use when you have a long-read BAM/CRAM aligned to a reference genome and need to detect non-reference TE insertions, or when benchmarking sTELLeR against TLDR/xTEA on PacBio data.

## When NOT to use

- Input is short-read sequencing data — sTELLeR is designed and tested for PacBio/ONT long reads; use short-read TE callers such as RetroSeq or MELT2 instead
- No reference-aligned long-read BAM/CRAM is available or the reference used for alignment cannot be provided — sTELLeR requires both the read file and the exact reference it was aligned to
- Only reference-resident TE annotation is needed — sTELLeR detects non-reference insertions, not reference TE genotyping against a reference callset alone

## Inputs

- BAM or CRAM file of long reads aligned to the reference genome (e.g. PacBio BAM from HG002/HG004/HG01071 or simulated chr22 testdata.bam)
- Reference genome FASTA used for alignment (e.g. HG38_chr22.fa; note the masked chr22 reference in sTELLeR_supplementary must be unzipped with tar -xvzf)
- TE sequence FASTA listing insertions to detect (e.g. fasta/TEfastasequences.fa with AluJb, AluY, AluSx, L1HS, L1preTa, etc., or TEsequences_SVA_HERV_ALU_L1.fasta adding SVA and HERV)

## Outputs

- VCF file of TE insertions (outputprefix_repeats.vcf) with INS:ME variant names, TE type and start/end in INFO, and TE:TS:HT:GT in FORMAT
- Optionally, intermediate files per chromosome: candidate-insertion FASTA (ID_chr_reads.fasta) and TE-aligned SAM (ID_chr_repeats.sam) if --keep_intermediates is set

## How to apply

Run sTELLeR by supplying a reference genome (--ref), a TE sequence FASTA (e.g. TEfastasequences.fa with Alu and L1 subfamilies, or TEsequences_SVA_HERV_ALU_L1.fasta for SVA/HERV inclusion), and the aligned BAM, using --style pb for PacBio CLR or 'ont' for Oxford Nanopore, with --sr 4 as the minimum supporting split reads and -mr 80 as the maximum supporting reads in the paper's benchmark settings (mapping quality default 20). sTELLeR extracts candidate insertions as intermediate per-chromosome FASTA files, aligns them to the TE FASTA (via minimap2/pysam/samtools), and emits a haplotype-aware VCF named outputprefix_repeats.vcf where variants are named INS:ME and the FORMAT fields TE:TS:HT:GT carry TE type, transposon start, haplotag, and genotype. Compare or intersect the VCF with truth sets or other callers' callsets (TLDR, xTEA) after normalizing all to the INS:ME VCF representation. Installation requires python3, pysam, samtools, and minimap2 (all available through bioconda), or the Docker container at hub.docker.com/r/kristinebilgrav/steller.

## Related tools

- **sTELLeR** (Primary TE insertion caller run via steller.py on long-read BAMs) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Dependency used to align candidate insertion sequences to the TE FASTA)
- **pysam** (Python dependency for BAM/reading support within sTELLeR)
- **samtools** (Dependency for BAM handling during candidate extraction)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- On the provided test data, the run should identify exactly 7 variants, verifiable by comparing testdata_res_repeats.vcf to testbamTRUTH.txt
- Output VCF variants are named INS:ME with TE type and start/end positions in the INFO field and TE:TS:HT:GT (haplotag, genotype 1/0 or 1/1) in FORMAT
- Reproduced benchmark numbers match Table values: sTELLeR 1885/1914 true positive TEI on HG002/HG01071 vs. TLDR 1741/1808 and xTEA 1543/1645; simulated Alu sensitivity/precision 0.97/1.0
- Intermediate files ID_chr_reads.fasta and ID_chr_repeats.sam are removed by default unless --keep_intermediates is specified
- Phased input yields haplotype and phased genotypes (1|0, 0|1, 1|1); unphased input yields HT=0 and unphased genotypes

## Limitations

- xTEA comparison context shows not all callers detect all families — e.g. xTEA calls no L1 or HERV on the simulated set; caller choice affects per-family sensitivity
- Supporting reads (--sr) must be tuned to sample coverage and desired sensitivity (default 3; paper used 4), and --maxreads (default 100, paper used 80) must be adjusted to avoid calls in regions exceeding the given coverage
- Haplotype information is only present in the output VCF if the input is phased; otherwise haplotag is 0 and genotype alone indicates het/hom
- The masked chr22 test reference is shipped zipped and must be extracted with tar -xvzf for correct results

## Evidence

- [readme] sTELLeR is a fast and CPU light tool designed and tested for detection of transposable element insertion in long-read data: "sTELLeR is a fast and CPU light tool designed and tested for detection of transposable element insertion in long-read data, however it supports identification of any type of insertions."
- [readme] Example PB run command with parameters: "python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80"
- [methods] Validation against truth file with 7 variants: "correct results can be verified by comparing to the provided file testbamTRUTH.txt - a total of 7 variants should be identified."
- [readme] VCF output naming and FORMAT semantics: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
- [abstract] Benchmark performance claim: "The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers."
- [readme] Supplementary TE FASTA contents: [evidence span withheld — anchor status: external_doc]
