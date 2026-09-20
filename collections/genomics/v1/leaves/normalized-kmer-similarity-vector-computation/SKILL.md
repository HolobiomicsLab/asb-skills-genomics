---
name: normalized-kmer-similarity-vector-computation
description: Use when a phased long-read assembly has leftover unphased reads (e.g.
  a VolcanoSV-asm phasing output with PS_HP haplotype blocks) that must be assigned
  to haplotypes before building a phased diploid assembly.
when_to_use_negative:
- The sample is homozygous and unsuitable for haplotype phasing (e.g. CHM13 derived
  from a single homozygous complete hydatidiform mole), so no haplotype fingerprints
  exist
- Reads are already assigned to haplotype blocks (PS_HP) — this module targets only
  the remaining unphased reads
- Processing sex chromosomes — VolcanoSV is designed only for autosome chromosomes
  since the phasing procedure differs
edam_operation: http://edamontology.org/operation_0358
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: VolcanoSV
  role: Provides the VolcanoSV-asm module that builds haplotype fingerprint k-mer
    sets, computes normalized similarity vectors for unphased reads, and applies the
    quantile significance test for read-to-haplotype assignment
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:a7f52740939d7a742314f9d69e01b8b84da401dd6acd1fdcc4e291199eec0f4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_0358
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# Normalized k-mer similarity vector computation

## Summary

Computes a normalized similarity vector between each unphased long read and haplotype-specific unique k-mer fingerprint sets (k=12, step=1) within VolcanoSV-asm, enabling statistically tested assignment of unphased reads to phased haplotypes during diploid assembly.

## When to use

Use when a phased long-read assembly has leftover unphased reads (e.g. a VolcanoSV-asm phasing output with PS_HP haplotype blocks) that must be assigned to haplotypes before building a phased diploid assembly.

## When NOT to use

- The sample is homozygous and unsuitable for haplotype phasing (e.g. CHM13 derived from a single homozygous complete hydatidiform mole), so no haplotype fingerprints exist
- Reads are already assigned to haplotype blocks (PS_HP) — this module targets only the remaining unphased reads
- Processing sex chromosomes — VolcanoSV is designed only for autosome chromosomes since the phasing procedure differs

## Inputs

- Phased read sets per haplotype block (PS_HP files from VolcanoSV-asm phasing output directory)
- Long-read alignment BAM (Hifi, CLR, or ONT reads) containing unphased reads
- Haplotype-specific unique (fingerprint) k-mer sets built with k=12, step=1

## Outputs

- Normalized similarity vector per unphased read (one similarity value per haplotype)
- Read-to-haplotype assignment table read_haplotype_assignments.tsv

## How to apply

Load the phased reads of each haplotype block (PS_HP) from the VolcanoSV-asm phasing output and build a fingerprint set of unique k-mers per haplotype using k-mer size k=12 with step=1, retaining only k-mers that uniquely mark each haplotype. Extract k-mers from each unphased read in the long-read BAM and compute a normalized similarity vector of that read against every haplotype's fingerprint set. Apply the empirical-distribution quantile significance test with ratio threshold r=10% (default) to decide whether the read's similarity to a given haplotype is statistically significant. Assign each read passing the significance test to its best-matching haplotype and record the assignment; downstream assembly quality on the completed phased read sets is validated by SV benchmarking (e.g. 94.19% average deletion F1 on Hifi datasets vs GIAB with Truvari p=0.5, P=0.5, r=500, S=30, O=0.01).

## Related tools

- **VolcanoSV** (Provides the VolcanoSV-asm module that builds haplotype fingerprint k-mer sets, computes normalized similarity vectors for unphased reads, and applies the quantile significance test for read-to-haplotype assignment) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam chr10.hifi.bam -o out_dir -ref genome.fa -chr chr10 -t 16
```

## Evaluation signals

- Every assigned read's similarity exceeds the empirical-distribution quantile significance threshold (r=10% default)
- Each unphased read yields a normalized similarity vector against every haplotype's fingerprint set, and the assignment goes to the best-matching haplotype
- Output table read_haplotype_assignments.tsv is complete, containing one row per assigned unphased read
- Downstream diploid assembly quality holds: VolcanoSV reports e.g. 94.19% average deletion F1 across Hifi datasets against the GIAB gold standard with Truvari p=0.5, P=0.5, r=500, S=30, O=0.01

## Limitations

- Only reads passing the r=10% quantile significance test are accepted; reads with insignificant similarity to any haplotype remain unassigned
- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- The module was not used for CHM13 diploid assembly because the sample is homozygous and unsuitable for haplotype phasing

## Evidence

- [methods] k-mer fingerprint parameters and similarity computation: [evidence span withheld — anchor status: not_found]
- [methods] significance test threshold: [evidence span withheld — anchor status: not_found]
- [abstract] article-level method claim: "VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery"
- [readme] autosome-only limitation from README: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
