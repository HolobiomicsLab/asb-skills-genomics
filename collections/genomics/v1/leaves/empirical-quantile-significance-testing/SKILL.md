---
name: empirical-quantile-significance-testing
description: Use when unphased long reads (Hifi, CLR, or ONT BAM) must be assigned
  to already-phased haplotype blocks during diploid assembly, i.e. the read-partitioning
  step of VolcanoSV-asm where similarity must be tested for significance rather than
  assumed.
when_to_use_negative:
- The sample is homozygous/haploid (e.g. CHM13, which derives from a single homozygous
  complete hydatidiform mole) — the article explicitly did not use VolcanoSV-asm for
  its diploid assembly since the sample is unsuitable for haplotype phasing
- The data are sex chromosomes — the VolcanoSV README states the pipeline is designed
  only for autosome chromosomes since the phasing procedure for sex chromosomes is
  very different
- Reads are already confidently phased by upstream SNP-based phasing and do not require
  k-mer-similarity-based assignment
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: VolcanoSV
  role: VolcanoSV-asm implements the fingerprint k-mer (k=12, step=1) similarity vectors
    and the empirical-distribution quantile significance test (r=10% default) that
    assign unphased reads to haplotypes in its diploid assembly module
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:55fd929bf6652cf5d9cc1738f59f4c77eaa01d77022633bf4e76c6c4e1d69455
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# empirical-quantile-significance-testing

## Summary

A statistical assignment procedure from VolcanoSV-asm that tests whether an unphased long read's unique k-mer similarity to haplotype-specific fingerprint k-mer sets (k=12, step=1) is statistically significant under an empirical-distribution quantile test (default ratio threshold r=10%), so unphased reads can be attached to the correct haplotype during diploid assembly.

## When to use

Use when unphased long reads (Hifi, CLR, or ONT BAM) must be assigned to already-phased haplotype blocks during diploid assembly, i.e. the read-partitioning step of VolcanoSV-asm where similarity must be tested for significance rather than assumed.

## When NOT to use

- The sample is homozygous/haploid (e.g. CHM13, which derives from a single homozygous complete hydatidiform mole) — the article explicitly did not use VolcanoSV-asm for its diploid assembly since the sample is unsuitable for haplotype phasing
- The data are sex chromosomes — the VolcanoSV README states the pipeline is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Reads are already confidently phased by upstream SNP-based phasing and do not require k-mer-similarity-based assignment

## Inputs

- Long-read alignment BAM (PacBio HiFi, CLR, or ONT reads) containing phased and unphased reads
- VolcanoSV-asm phasing output directory with phased reads per haplotype block (PS_HP tags)
- k-mer parameters: k-mer size k=12, step=1
- Empirical-distribution quantile test ratio threshold r (default 10%)

## Outputs

- read_haplotype_assignments.tsv: read-to-haplotype assignment table for all unphased reads passing the significance test
- Haplotype-partitioned read sets feeding the haplotype-aware local assembly module of VolcanoSV-asm

## How to apply

Load the phased reads of each haplotype block (PS_HP) from the VolcanoSV-asm phasing output and build a fingerprint set of unique k-mers per haplotype using k-mer size k=12 with step=1, retaining k-mers that uniquely mark each haplotype. Extract k-mers from every unphased read in the long-read BAM and compute a normalized similarity vector of the read against each haplotype's fingerprint set. Apply the empirical-distribution quantile significance test with the ratio threshold set to r=10% (the default) to decide whether the read's similarity to a haplotype is statistically significant rather than background noise. For each read passing the test, assign it to its best-matching haplotype and record the assignment; all read-to-haplotype assignments are written to read_haplotype_assignments.tsv. The rationale is that unique haplotype-specific k-mers provide a SNP-independent signal that complements VolcanoSV's phased-SNP phasing, enabling precise haplotype-resolved SV discovery.

## Related tools

- **VolcanoSV** (VolcanoSV-asm implements the fingerprint k-mer (k=12, step=1) similarity vectors and the empirical-distribution quantile significance test (r=10% default) that assign unphased reads to haplotypes in its diploid assembly module) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py --bam_file chr10.bam --reference genome.fa --chr chr10 --output_dir out_dir --n_thread 16 --assembler hifiasm
```

## Evaluation signals

- Every assigned read's normalized similarity to its haplotype fingerprint set exceeds the empirical-distribution quantile significance threshold (r=10% default); reads below the threshold must not be assigned
- read_haplotype_assignments.tsv exists and contains exactly one best-matching haplotype per passing read
- Fingerprint k-mer sets are built at k=12 with step=1 and contain only k-mers unique to each haplotype (fingerprint k-mers), not shared k-mers
- Downstream SV benchmarking stays in the paper's reported range, e.g. average deletion F1 of 94.19% on HiFi datasets against the GIAB gold standard with Truvari p=0.5, P=0.5, r=500, O=0.01

## Limitations

- Reads whose similarity fails the r=10% quantile significance test remain unassigned, so coverage of the final haplotype-aware assembly depends on test sensitivity
- Assignment quality is conditional on the correctness of the upstream SNP-based phasing that defines the PS_HP haplotype blocks
- VolcanoSV-asm was not used for CHM13 because the sample is homozygous and unsuitable for haplotype phasing, so this skill cannot produce meaningful haplotype assignments on such data

## Evidence

- [abstract] VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery: "VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery"
- [readme] VolcanoSV is an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly.: "VolcanoSV is an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly."
- [results] Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions.: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."
