---
name: transposable-element-sequence-alignment
description: Use when a BAM/CRAM of long reads aligned to a reference contains split
  reads or insertion signatures suspected to be non-reference TE insertions that must
  be classified against TE consensus FASTA sequences.
when_to_use_negative:
- The goal is detection of reference TE copies already present in the reference genome
  — sTELLeR targets non-reference insertions
- Input is short-read (Illumina) data — the method is designed and tested for long-read
  PacBio or ONT data
- Only generic insertions unrelated to any provided TE/insertion FASTA are sought
  — the caller classifies candidates against the supplied --TE_fasta sequences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: sTELLeR
  role: 'End-to-end caller: extracts candidates, runs the TE alignment step, refines
    consensus, applies filters, and writes the VCF'
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Aligns candidate insertion sequences to the TE consensus FASTA, producing
    ID_chr_repeats.sam intermediates
- name: pysam
  role: Parses the input BAM to extract split reads and insertion sequences spanning
    candidate TE insertions
- name: samtools
  role: Dependency used for BAM/SAM handling in the sTELLeR workflow
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:e23129d1ecf9e1bc2ab3e69130d5b2cefdeb00aec0a337424f6bcbde5c8a75e7
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# transposable-element-sequence-alignment

## Summary

Aligns candidate non-reference insertion sequences extracted from long-read (PacBio/ONT) alignments against a FASTA of known transposable element consensus sequences (e.g. L1, Alu, SVA, HERV) with minimap2, then classifies and filters insertions by supporting read count and mapping quality to call TE insertions in VCF.

## When to use

Use when a BAM/CRAM of long reads aligned to a reference contains split reads or insertion signatures suspected to be non-reference TE insertions that must be classified against TE consensus FASTA sequences.

## When NOT to use

- The goal is detection of reference TE copies already present in the reference genome — sTELLeR targets non-reference insertions
- Input is short-read (Illumina) data — the method is designed and tested for long-read PacBio or ONT data
- Only generic insertions unrelated to any provided TE/insertion FASTA are sought — the caller classifies candidates against the supplied --TE_fasta sequences

## Inputs

- BAM or CRAM of long reads (PacBio pb/clr, hifi, or ONT) aligned to a reference genome
- Reference genome FASTA used for the alignment (e.g. HG38_chr22.fa)
- FASTA of TE consensus/insertion sequences to detect (e.g. TEfastasequences.fa with L1 and Alu subfamilies, optionally SVA and HERV)
- Optional phased input for haplotype-aware output

## Outputs

- VCF file of TE insertion calls named INS:ME with INFO TE type plus start/end positions and FORMAT TE:TS:HT:GT fields
- Intermediate per-chromosome candidate-insertion FASTA files (ID_chr_reads.fasta)
- Intermediate SAM files of candidate insertions aligned to TEs (ID_chr_repeats.sam)

## How to apply

Extract split reads and insertion-spanning sequences from the input BAM (pysam), cluster candidate insertion signatures (DBSCAN), and write per-chromosome candidate FASTA intermediates (ID_chr_reads.fasta). Align these candidate sequences to the TE FASTA (e.g. TEfastasequences.fa containing L1 and Alu subfamilies, or TEsequences_SVA_HERV_ALU_L1.fasta adding SVA and HERV) with minimap2, writing SAM intermediates (ID_chr_repeats.sam). Refine calls to a consensus nucleotide position and filter with sTELLeR defaults: minimum 3 supporting split reads/insertions (--sr, default 3), mapping quality >= 20 to the specific TE sequence (--mq, default 20), and a maximum of 100 supporting reads (--maxreads, default 100) to avoid calls in regions exceeding given coverage. Emit haplotype-aware VCF calls named INS:ME with format fields TE:TS:HT:GT.

## Related tools

- **sTELLeR** (End-to-end caller: extracts candidates, runs the TE alignment step, refines consensus, applies filters, and writes the VCF) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Aligns candidate insertion sequences to the TE consensus FASTA, producing ID_chr_repeats.sam intermediates)
- **pysam** (Parses the input BAM to extract split reads and insertion sequences spanning candidate TE insertions)
- **samtools** (Dependency used for BAM/SAM handling in the sTELLeR workflow)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- On the sTELLeR_supplementary test data (HG38_chr22.fa, TEfastasequences.fa, testdata.bam with --sr 4 --style pb -mr 80), a total of 7 variants should be identified in testdata_res_repeats.vcf
- Calls match the provided truth positions in testbamTRUTH.txt when compared
- Every called variant has a mapping quality to the assigned TE sequence at or above the --mq threshold (default 20)
- Each call is supported by at least --sr (default 3) split reads/insertions and no more than --maxreads (default 100)
- Unphased output carries HT=0 with GT 1/0 or 1/1; phased input yields HT and GT 1|0, 0|1 or 1|1

## Limitations

- Supporting reads threshold (--sr) must be tuned to sample coverage and desired sensitivity; the default is 3
- Maximum supporting reads (--maxreads, default 100) guards against calls in regions exceeding given coverage; mis-tuning can miss or spuriously call high-coverage regions
- Intermediate files (ID_chr_reads.fasta, ID_chr_repeats.sam) are deleted unless --keep_intermediates is set
- Reference must match the alignment reference exactly; the supplementary test reference is a masked chr22 zip that must be unpacked (tar -xvzf) for correct reference bases

## Evidence

- [intro] a tool for detection of non-reference transposable element using long-read data such as PacBio or ONT: "a tool for detection of non-reference transposable element using long-read data such as PacBio or ONT"
- [methods] mq mapping quality default: "-m , --mq         Mapping quality (default 20)"
- [methods] sr minimum supporting reads: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
- [readme] TE fasta defines what can be detected: "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)."
- [readme] maxreads coverage rationale: "Maximum number of reads (--maxreads) can be adjusted to avoid calls in regions exceeding given coverage (default=100)."
- [readme] test data validation: "correct results can be verified by comparing to the provided file testbamTRUTH.txt - a total of 7 variants should be identified"
