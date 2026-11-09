---
name: kmer-fingerprint-haplotype-assignment
description: Use when a phased long-read assembly pipeline has partitioned most reads
  into haplotype blocks but leftover unphased reads (in a long-read BAM) must still
  be assigned to the correct haplotype.
when_to_use_negative:
- The sample is homozygous and unsuitable for haplotype phasing, e.g., CHM13 from
  a single complete hydatidiform mole (VolcanoSV-asm was not used for its diploid
  assembly)
- Input reads are short reads without a phased haplotype block structure to fingerprint
- Phasing is needed on sex chromosomes — VolcanoSV is designed only for autosome chromosomes
  since phasing there is very different
edam_operation: http://edamontology.org/operation_0346
edam_topics: []
tools:
- name: VolcanoSV
  role: Host pipeline; its VolcanoSV-asm module performs read partitioning, unique
    k-mer similarity assignment of unphased reads, and haplotype-aware local assembly
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:1977011b2c578deecc8b8a63ddfab3ad4dba95e7b08c816e82d7048595da3e5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_0346
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kmer-fingerprint-haplotype-assignment

## Summary

Assigns unphased long reads to haplotypes by computing k-mer similarity against haplotype-specific fingerprint k-mer sets, as implemented in VolcanoSV-asm to complete its phased diploid assembly prior to haplotype-aware local assembly.

## When to use

Use when a phased long-read assembly pipeline has partitioned most reads into haplotype blocks but leftover unphased reads (in a long-read BAM) must still be assigned to the correct haplotype.

## When NOT to use

- The sample is homozygous and unsuitable for haplotype phasing, e.g., CHM13 from a single complete hydatidiform mole (VolcanoSV-asm was not used for its diploid assembly)
- Input reads are short reads without a phased haplotype block structure to fingerprint
- Phasing is needed on sex chromosomes — VolcanoSV is designed only for autosome chromosomes since phasing there is very different

## Inputs

- Phased read sets (PS_HP) per haplotype from VolcanoSV-asm phasing output
- Long-read BAM file containing unphased reads (Hifi, CLR, or ONT)
- k-mer size (k=12) and step (1) parameters

## Outputs

- read_haplotype_assignments.tsv — read-to-haplotype assignment table
- Haplotype-specific fingerprint (unique) k-mer sets
- Phased read sets enriched with assigned unphased reads for haplotype-aware local assembly

## How to apply

Load the phased reads per haplotype block (PS_HP) from the VolcanoSV-asm phasing output and build a fingerprint k-mer set per haplotype using k=12 with step=1, retaining k-mers that uniquely mark each haplotype. Extract k-mers from each unphased read in the long-read BAM and compute a normalized similarity vector against every haplotype's fingerprint set. Apply an empirical-distribution quantile significance test with the ratio threshold set to the default r=10%; a read whose similarity to a haplotype passes the test is assigned to its best-matching haplotype. Write assignments to read_haplotype_assignments.tsv, which feed the downstream haplotype-aware local assembly module. The rationale is that unique haplotype-specific k-mers provide a robust, reference-independent signature for classifying reads that phased-SNP-based methods left ambiguous.

## Related tools

- **VolcanoSV** (Host pipeline; its VolcanoSV-asm module performs read partitioning, unique k-mer similarity assignment of unphased reads, and haplotype-aware local assembly) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam sample.chr10.bam -ref genome.fa -o outdir -chr chr10
```

## Evaluation signals

- Every assigned read's k-mer similarity exceeds the empirical-distribution quantile significance threshold (r=10% default)
- Each unphased read is assigned to exactly one best-matching haplotype and recorded in read_haplotype_assignments.tsv
- Fingerprint k-mer sets contain only k-mers that uniquely mark each haplotype (k=12, step=1)
- Downstream SV benchmarking quality is consistent, e.g., 94.19% average deletion F1 across Hifi datasets vs. GIAB gold standard with Truvari p=0.5, P=0.5, r=500, S=30, O=0.01

## Limitations

- Assignment quality depends on haplotype-specific fingerprint coverage; reads with few unique k-mers may fail the r=10% significance test and remain unassigned
- VolcanoSV-asm phasing/assignment is unsuitable for homozygous samples such as CHM13
- Designed for autosome chromosomes only; sex-chromosome phasing differs

## Evidence

- [abstract] VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery: "VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery"
- [full_text] remaining unphased reads are assigned using unique k-mer similarity analysis against haplotype-specific fingerprints: [evidence span withheld — anchor status: not_found]
- [methods] Build a fingerprint (unique) k-mer set for each haplotype using k-mer size k=12 with step=1, retaining k-mers that uniquely mark each haplotype.: [evidence span withheld — anchor status: not_found]
- [methods] Apply the empirical-distribution quantile significance test with the ratio threshold set to r=10% (the default) to decide whether a read's similarity to a haplotype is statistically significant.: [evidence span withheld — anchor status: not_found]
- [results] We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole: "We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole"
