---
name: alu-l1-insertion-genotyping
description: Use when you have a long-read BAM (PacBio or ONT) plus the aligned reference
  and an Alu/L1 TE fasta, and need to call and genotype non-reference TE insertions
  and compare caller agreement.
when_to_use_negative:
- Input is short-read data only and no long-read BAM is available for the sample —
  sTELLeR is designed for PacBio/ONT long reads; use the short-read callers directly
  instead.
- The goal is reference-resident TE annotation rather than non-reference insertion
  detection — sTELLeR detects non-reference TE insertions, not reference element cataloging.
- You need a quick install sanity check rather than a full four-caller overlap — use
  the provided testdata.bam truth-set comparison (7 expected variants on chr22) instead
  of the full HG004 workflow.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sTELLeR
  role: 'Primary long-read caller: detects non-reference TE insertions (Alu/L1) from
    the PacBio/ONT BAM and outputs a haplotype-aware VCF'
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: TLDR
  role: Independent long-read TE caller run on the same HG004 PacBio BAM for callset
    concordance
- name: RetroSeq
  role: Short-read TE caller run on the 30x HG004 BAM to provide a short-read comparison
    callset
- name: MELT2
  role: Short-read TE caller run on the 30x HG004 BAM to provide a second short-read
    comparison callset
- name: python3
  role: Runtime for sTELLeR (with pysam, samtools, minimap2 dependencies) and for
    computing callset intersections
- name: minimap2
  role: Dependency for sTELLeR, available through bioconda
- name: pysam
  role: Dependency for sTELLeR, available through bioconda
- name: samtools
  role: Dependency for sTELLeR, available through bioconda
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:be122a2e50630481836dd7327c20e28982c46c866685311fafb62442494f7175
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
yes_rate: 0.8
n_claims: 15
---

# alu-l1-insertion-genotyping

## Summary

Detect non-reference Alu/L1 transposable element insertions in long-read (PacBio/ONT) genomes with sTELLeR and cross-validate them against independent long-read (TLDR) and short-read (RetroSeq, MELT2) caller callsets by computing pairwise and four-way VCF overlaps, as done for HG004 (642 TEs shared by all four callers; 1302 TEI agreed by sTELLeR and TLDR).

## When to use

Use when you have a long-read BAM (PacBio or ONT) plus the aligned reference and an Alu/L1 TE fasta, and need to call and genotype non-reference TE insertions and compare caller agreement.

## When NOT to use

- Input is short-read data only and no long-read BAM is available for the sample — sTELLeR is designed for PacBio/ONT long reads; use the short-read callers directly instead.
- The goal is reference-resident TE annotation rather than non-reference insertion detection — sTELLeR detects non-reference TE insertions, not reference element cataloging.
- You need a quick install sanity check rather than a full four-caller overlap — use the provided testdata.bam truth-set comparison (7 expected variants on chr22) instead of the full HG004 workflow.

## Inputs

- Long-read BAM or CRAM (PacBio CLR/HiFi or ONT) aligned to the reference genome (e.g. HG004 PacBio BAM)
- Reference genome fasta used for alignment (e.g. HG38_chr22.fa)
- Fasta of TE insertion sequences to detect (e.g. TEfastasequences.fa: L1HS, L1preTa, L1PA2, L1PA6, L1, AluJb, AluY, AluJr, AluSp, AluSq, AluJo, AluYb8, AluYa5, AluSx; optionally TEsequences_SVA_HERV_ALU_L1.fasta with SVA and HERV)
- 30x short-read BAM (e.g. HG004 realigned with bwa-mem) for RetroSeq and MELT2 calls
- Optional phased BAM (phased input enables haplotype and genotype annotation in output VCF)

## Outputs

- sTELLeR VCF of TE insertions (variants named INS:ME; INFO holds TE type and transposon start/end; FORMAT TE:TS:HT:GT carries TE type, transposon start, haplotag, and genotype 1/0, 1/1, 1|0, 0|1)
- Independent TLDR long-read TE callset VCF
- RetroSeq and MELT2 short-read TE callset VCFs
- Merged, normalized four-caller VCF intersection table (overlap statistics: four-way shared TEs, sTELLeR-TLDR agreement, TEs missed by long-read callers)

## How to apply

Align the long reads to the reference genome (e.g. HG38) and run sTELLeR via `python steller/steller.py` with `--style pb` (or `ont`), `--sr 4` (minimum supporting split reads/insertions), and `-mr 80` (maximum supporting reads to avoid calls in high-coverage regions), providing the reference fasta, the TE fasta (e.g. TEfastasequences.fa with Alu and L1 subfamilies such as AluJb, AluY, L1HS, L1PA2), and the BAM; sTELLeR outputs a haplotype-aware VCF where variants are named INS:ME with TE type and start/end positions in the INFO field and TE:TS:HT:GT in the FORMAT. Run TLDR on the same PacBio BAM for an independent long-read callset, and run RetroSeq and MELT2 on the 30x short-read HG004 BAM (realigned with bwa-mem). Normalize all four callsets to a common VCF representation (INS:ME naming with TE type and start/end positions in INFO) so they can be merged, then compute pairwise and four-way intersections, tallying TEs shared by all four callers, TEI agreed by sTELLeR and TLDR, and TEs missed by the long-read callers.

## Related tools

- **sTELLeR** (Primary long-read caller: detects non-reference TE insertions (Alu/L1) from the PacBio/ONT BAM and outputs a haplotype-aware VCF) — https://github.com/kristinebilgrav/sTELLeR
- **TLDR** (Independent long-read TE caller run on the same HG004 PacBio BAM for callset concordance)
- **RetroSeq** (Short-read TE caller run on the 30x HG004 BAM to provide a short-read comparison callset)
- **MELT2** (Short-read TE caller run on the 30x HG004 BAM to provide a second short-read comparison callset)
- **python3** (Runtime for sTELLeR (with pysam, samtools, minimap2 dependencies) and for computing callset intersections)
- **minimap2** (Dependency for sTELLeR, available through bioconda)
- **pysam** (Dependency for sTELLeR, available through bioconda)
- **samtools** (Dependency for sTELLeR, available through bioconda)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- On the sTELLeR supplementary test data (testdata.bam, chr22), exactly 7 variants should be identified, verifiable against testbamTRUTH.txt.
- The HG004 four-caller overlap table should reproduce 642 TEs shared by sTELLeR, TLDR, RetroSeq, and MELT2.
- The HG004 overlap analysis should reproduce 1302 TE insertions agreed upon by sTELLeR and TLDR.
- The HG004 overlap analysis should reproduce 1280 TEs undetected by the long-read (lrGS) callers.
- Output VCF records must be named INS:ME with TE type and start/end positions in the INFO field, mergeable with VCFs from the other callers.

## Limitations

- Call sensitivity depends on coverage: --sr (default 3; 4 used here) is the minimum supporting split reads/insertions, and --maxreads (default 100; 80 used here) caps calls in regions exceeding a coverage threshold.
- The mapping quality threshold --mq (default 20) filters accepted TE calls against the specific TE sequence.
- If input BAM is not phased, haplotag is 0 and only genotype (1/0 or 1/1) is reported; haplotype-aware output requires phased input.
- Intermediate files (ID_chr_reads.fasta, ID_chr_repeats.sam) are deleted unless --keep_intermediates is set, which may complicate debugging.
- sTELLeR's TE detection scope is bounded by the TE fasta provided; only Alu and L1 (and optionally SVA/HERV) subfamilies in the supplied fastas are detectable.

## Evidence

- [readme] sTELLeR is a fast and CPU light tool designed and tested for detection of transposable element insertion in long-read data, however it supports identification of any type of insertions.: "sTELLeR is a fast and CPU light tool designed and tested for detection of transposable element insertion in long-read data, however it supports identification of any type of insertions."
- [readme] Example to run on PB data: "python steller/steller.py --ref < genome ref_file > --TE_fasta < TE sequence fasta file > --bam < bamfile > --sr 4 --style pb -o < output prefix > -mr 80"
- [readme] The variant is named INS:ME and in the info section the TE type along with start and end positions are given.: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
- [methods] a total of 7 variants should be identified: "correct results can be verified by comparing to the provided file testbamTRUTH.txt - a total of 7 variants should be identified."
- [methods] Ranging thresholds documented in the article index: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
- [results] Four-caller overlap statistics from the source card: [evidence span withheld — anchor status: not_found]
- [abstract] Haplotype-aware VCF output claim: "The caller is haplotype aware and output results in a variant call format (VCF) file, enabling compatibility with other variant callers and downstream analysis."
