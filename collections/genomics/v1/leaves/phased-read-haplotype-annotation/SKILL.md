---
name: phased-read-haplotype-annotation
description: Use when a haplotype-aware long-read assembly pipeline (e.g. VolcanoSV-asm)
  leaves unassigned reads after phasing and they must be partitioned to haplotype-specific
  read sets before local assembly.
when_to_use_negative:
- The sample is homozygous or haploid (e.g. CHM13 from a single homozygous complete
  hydatidiform mole) — VolcanoSV-asm phasing itself was skipped for CHM13, so there
  are no haplotype fingerprints to assign against.
- Working on sex chromosomes — VolcanoSV is designed only for autosome chromosomes
  since the phasing procedure for sex chromosomes is very different.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Pipeline whose VolcanoSV-asm module performs the read partitioning, unique
    k-mer fingerprinting, and significance-based assignment of unphased reads to haplotypes
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:110d9890521accdb076392f892eae6edc9c1335a5cdc6ab16202c268e927c73d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# phased-read-haplotype-annotation

## Summary

Assigns unphased long reads to haplotypes during VolcanoSV-asm's phased diploid assembly by computing unique k-mer (k=12, step=1) similarity fingerprints from haplotype-specific phased read sets and applying an empirical-distribution quantile significance test with a 10% ratio threshold.

## When to use

Use when a haplotype-aware long-read assembly pipeline (e.g. VolcanoSV-asm) leaves unassigned reads after phasing and they must be partitioned to haplotype-specific read sets before local assembly.

## When NOT to use

- The sample is homozygous or haploid (e.g. CHM13 from a single homozygous complete hydatidiform mole) — VolcanoSV-asm phasing itself was skipped for CHM13, so there are no haplotype fingerprints to assign against.
- Working on sex chromosomes — VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different.

## Inputs

- Phased BAM/read sets for each haplotype block (PS_HP) from VolcanoSV-asm phasing output
- Long-read BAM containing unphased reads (Hifi, CLR, or ONT)

## Outputs

- read_haplotype_assignments.tsv — read-to-haplotype assignment table
- Haplotype-specific (fingerprint) k-mer sets per haplotype (k=12, step=1)

## How to apply

Load the phased reads belonging to each haplotype block (PS_HP tags) from the VolcanoSV-asm phasing output directory, then build a fingerprint k-mer set per haplotype using k=12 with step=1, retaining only k-mers that uniquely mark each haplotype. Extract k-mers from each unphased read in the long-read BAM and compute a normalized similarity vector against every haplotype's fingerprint set. Apply the empirical-distribution quantile significance test with ratio threshold r=10% (default) to decide whether a read's similarity to a haplotype is statistically significant, and assign each read passing the test to its best-matching haplotype. Rationale: complete read partitioning improves the downstream haplotype-aware local assembly that underpins VolcanoSV's SV calling accuracy (94.19% average deletion F1 across Hifi datasets vs. GIAB with Truvari p=0.5, P=0.5, r=500, O=0.01).

## Related tools

- **VolcanoSV** (Pipeline whose VolcanoSV-asm module performs the read partitioning, unique k-mer fingerprinting, and significance-based assignment of unphased reads to haplotypes) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam chr10.bam -o out_dir -ref genome.fa -chr chr10 -t 16
```

## Evaluation signals

- Every assigned read's similarity exceeds the empirical-distribution quantile significance threshold (r=10%) — assignments failing this are rejected.
- The output table read_haplotype_assignments.tsv is produced and each read is assigned to exactly one best-matching haplotype.
- Downstream SV benchmarking with Truvari (p=0.5, P=0.5, r=500, O=0.01) against the GIAB gold standard shows expected performance (94.19% average deletion F1, 98.32%/99.01% genotyping accuracy across Hifi datasets).
- Fingerprint k-mer sets are haplotype-specific: k-mers retained with k=12, step=1 uniquely mark one haplotype and are not shared across haplotypes.

## Limitations

- Only statistically significant similarities (empirical quantile test, r=10% default) allow assignment; reads failing the test remain unassigned.
- VolcanoSV-asm is not applied to sex chromosomes, and was not used for CHM13 because the sample derives from a single homozygous complete hydatidiform mole.
- Assignment quality is indirect — it is reflected only in downstream SV benchmarking performance, not validated read-by-read against a truth set.

## Evidence

- [abstract] VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery: "VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery"
- [results] Across Hifi datasets, VolcanoSV achieved the best average F1 and genotyping accuracy: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."
- [results] We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole: "We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole"
- [readme] VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
