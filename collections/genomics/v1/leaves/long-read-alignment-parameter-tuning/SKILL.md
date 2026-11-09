---
name: long-read-alignment-parameter-tuning
description: Use when running sTELLeR on PacBio/ONT long-read BAM or CRAM data to
  call non-reference TE insertions and you must choose sensitivity thresholds (supporting
  reads, mapping quality, max reads) appropriate to sample coverage and read style.
when_to_use_negative:
- Input is short-read sequencing data — sTELLeR is designed and tested for PacBio/ONT
  long-read detection of non-reference TE insertions.
- The insertion of interest is present in the reference genome — the caller targets
  non-reference insertions.
- You need de novo TE discovery without prior TE sequences — a TE fasta of element
  sequences to detect must be supplied.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0157
tools:
- name: sTELLeR
  role: Main TE insertion caller; parameters --sr, --mq, --maxreads, --style are tuned
    per dataset and platform
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Dependency for aligning long reads to the reference and TE sequences before
    parameterized calling
- name: samtools
  role: Dependency for BAM handling during calling
- name: pysam
  role: Python dependency used by sTELLeR to read/write alignment and VCF data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:36df92d9944fb5b645f51eb3de439638ad7a6a93dd14108c7fe04d8e146123d8
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
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# long-read-alignment-parameter-tuning

## Summary

Tuning sTELLeR's calling thresholds (--sr, --mq, --maxreads) and read-style setting (--style pb/ont) to detect non-reference transposable element insertions from long-read (PacBio/ONT) BAM alignments, benchmarked against truth sets for Alu, L1, HERV, and SVA insertions.

## When to use

Use when running sTELLeR on PacBio/ONT long-read BAM or CRAM data to call non-reference TE insertions and you must choose sensitivity thresholds (supporting reads, mapping quality, max reads) appropriate to sample coverage and read style.

## When NOT to use

- Input is short-read sequencing data — sTELLeR is designed and tested for PacBio/ONT long-read detection of non-reference TE insertions.
- The insertion of interest is present in the reference genome — the caller targets non-reference insertions.
- You need de novo TE discovery without prior TE sequences — a TE fasta of element sequences to detect must be supplied.

## Inputs

- BAM or CRAM file of long reads (PacBio or ONT) aligned to a reference genome
- Reference genome FASTA (e.g. HG38_chr22.fa; for the test data use the masked chr22 reference shipped in the supplementary repo)
- TE sequence FASTA (--TE_fasta), e.g. fasta/TEfastasequences.fa or fasta/TEsequences_SVA_HERV_ALU_L1.fasta
- Truth set file for validation (e.g. testbamTRUTH.txt) when benchmarking

## Outputs

- VCF file (e.g. testdata_res_repeats.vcf) with variants named INS:ME, TE type in INFO, and TE:TS:HT:GT in FORMAT
- Optional intermediate per-chromosome candidate-insertion FASTA (ID_chr_reads.fasta) and TE-alignment SAM (ID_chr_repeats.sam) if --keep_intermediates is set
- Per-TE-family sensitivity (TP/(TP+FN)) and precision (TP/(TP+FP)) metrics when compared to a truth set

## How to apply

Run steller.py with a reference genome, a TE fasta (--TE_fasta) listing element sequences to detect (e.g. TEfastasequences.fa with L1 and Alu, or TEsequences_SVA_HERV_ALU_L1.fasta adding SVA and HERV), and the aligned BAM. Set --sr (minimum supporting split reads/insertions, default 3; benchmark used 4) according to sample coverage and desired sensitivity, --mq as the mapping-quality threshold for TE-sequence alignment (default 20), and --mr/--maxreads (default 100; benchmark used 80) to avoid calls in regions exceeding the coverage cap. Choose --style 'pb' for PacBio CLR/HiFi or 'ont' for Oxford Nanopore since alignment characteristics differ between platforms. Parse the output VCF, whose variants are named INS:ME with the TE type in the INFO/FORMAT fields, then compute per-family sensitivity TP/(TP+FN) and precision TP/(TP+FP) against a truth set such as testbamTRUTH.txt; benchmark values on simulated data were sTELLeR Alu sensitivity 0.97 with precision 1.0, and 0.97/1.0 for SVA, 0.96/1.0 for L1, and 0.98/1.0 for HERV.

## Related tools

- **sTELLeR** (Main TE insertion caller; parameters --sr, --mq, --maxreads, --style are tuned per dataset and platform) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Dependency for aligning long reads to the reference and TE sequences before parameterized calling)
- **samtools** (Dependency for BAM handling during calling)
- **pysam** (Python dependency used by sTELLeR to read/write alignment and VCF data)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- On the bundled test data, exactly 7 variants should be identified and match testbamTRUTH.txt.
- Output VCF variants are named INS:ME with TE type, start/end positions in INFO, and TE:TS:HT:GT (genotype 1/0 or 1/1 unphased; 1|0, 0|1, or 1|1 phased) in FORMAT.
- Per-family sensitivity/precision on the simulated benchmark reach sTELLeR 0.97/1.0 (Alu), 0.96/1.0 (L1), 0.98/1.0 (HERV), 0.97/1.0 (SVA).
- Lowering --sr increases sensitivity at the risk of precision; raising --mr avoids spurious calls in regions exceeding the coverage cap (default 100).
- If --keep_intermediates is not set, no ID_chr_reads.fasta or ID_chr_repeats.sam files should remain after the run.

## Limitations

- Parameter defaults (sr=3, mq=20, maxreads=100) are starting points; supporting reads needed depend on sample coverage and desired sensitivity, so thresholds must be re-tuned per dataset.
- If input is not phased, haplotype output is 0 and genotype only indicates het (1/0) vs hom (1/1), losing haplotype-specific call information.
- The --maxreads cap may suppress true calls in very high-coverage regions if set too low.
- The supplied test reference is a masked chr22 with removed masked parts; using the wrong reference will give incorrect results.

## Evidence

- [readme] Supporting reads (--sr) of a insertion depends on the coverage of the sample and the desired sensitivity (dafault=3).: "Supporting reads (--sr) of a insertion depends on the coverage of the sample and the desired sensitivity (dafault=3)."
- [readme] Mapping quality threshold and max reads tuning: "Maximum number of reads (--maxreads) can be adjusted to avoid calls in regions exceeding given coverage (default=100)."
- [readme] Style parameter per platform: "-s , --style      ont or pb (clr) / hifi"
- [methods] Verification against truth set: "Results will be given in testdata_res_repeats.vcf and correct results can be verified by comparing to the provided file testbamTRUTH.txt - a total of 7 variants should be identified."
- [results] sTELLeR benchmark sensitivity/precision: [evidence span withheld — anchor status: not_found]
