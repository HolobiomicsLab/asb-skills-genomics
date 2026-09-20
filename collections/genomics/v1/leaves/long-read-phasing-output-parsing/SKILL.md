---
name: long-read-phasing-output-parsing
description: Use when a VolcanoSV-asm phasing output leaves unphased reads in a long-read
  BAM and you need to bin every read to a haplotype before haplotype-aware local assembly
  or phased SV calling.
when_to_use_negative:
- The sample is homozygous and unsuitable for haplotype phasing, e.g. VolcanoSV-asm
  was not used for CHM13 because it derives from a single homozygous complete hydatidiform
  mole
- The chromosome is a sex chromosome — VolcanoSV is designed only for autosome chromosomes
  since the phasing procedure for sex chromosomes is very different
- You already have fully phased reads with no unphased remainder, so no k-mer similarity
  assignment step is needed
tools:
- name: VolcanoSV
  role: Provides VolcanoSV-asm, whose modules partition reads into haplotypes and
    assign unphased reads using unique k-mer similarity analysis against haplotype-specific
    fingerprints
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:8c2eddcfa706dcb023c954c452edb0c52ba54ba01b57a297d51328c3d0507a43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# long-read-phasing-output-parsing

## Summary

Parses the phased read blocks (PS_HP) produced by VolcanoSV-asm's phasing step and assigns remaining unphased long reads to haplotypes via unique k-mer similarity against haplotype-specific fingerprints, so a phased diploid assembly and haplotype-resolved SV calls can be built.

## When to use

Use when a VolcanoSV-asm phasing output leaves unphased reads in a long-read BAM and you need to bin every read to a haplotype before haplotype-aware local assembly or phased SV calling.

## When NOT to use

- The sample is homozygous and unsuitable for haplotype phasing, e.g. VolcanoSV-asm was not used for CHM13 because it derives from a single homozygous complete hydatidiform mole
- The chromosome is a sex chromosome — VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- You already have fully phased reads with no unphased remainder, so no k-mer similarity assignment step is needed

## Inputs

- VolcanoSV-asm phasing output directory with phased read blocks per haplotype (PS_HP tags)
- Long-read alignment BAM (Hifi, CLR, or ONT reads mapped to the reference)
- Reference FASTA and its .fai index

## Outputs

- read_haplotype_assignments.tsv table mapping each previously unphased read to its best-matching haplotype
- Haplotype-partitioned read sets ready for haplotype-aware local assembly

## How to apply

Load the phased reads belonging to each phased haplotype block (PS_HP) from the VolcanoSV-asm phasing output directory, then build a fingerprint k-mer set per haplotype using k=12 with step=1, retaining only k-mers that uniquely mark each haplotype. Extract k-mers from each unphased read in the long-read BAM and compute a normalized similarity vector of that read against every haplotype's fingerprint set. Apply the empirical-distribution quantile significance test with the ratio threshold at the default r=10% to decide whether a read's similarity to a given haplotype is statistically significant, then assign each read passing the test to its best-matching haplotype. Write all read-to-haplotype assignments to read_haplotype_assignments.tsv, which feeds the downstream haplotype-aware local assembly that underpins VolcanoSV's SV benchmarking performance (e.g., 94.19% average deletion F1 on Hifi datasets against the GIAB gold standard with Truvari p=0.5, P=0.5, r=500, S=30, O=0.01).

## Related tools

- **VolcanoSV** (Provides VolcanoSV-asm, whose modules partition reads into haplotypes and assign unphased reads using unique k-mer similarity analysis against haplotype-specific fingerprints) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam chr10.hifi.bam -ref genome.fa -o asm_out -chr chr10 -t 16
```

## Evaluation signals

- Every assigned read's k-mer similarity to its haplotype exceeds the empirical-distribution quantile significance threshold (r=10%)
- read_haplotype_assignments.tsv is populated and every unphased read receives at most one best-matching haplotype
- Fingerprint k-mers are unique per haplotype (built with k=12, step=1), so each haplotype has a distinct fingerprint set
- Downstream SV benchmarking stays consistent with reported performance, e.g. 94.19% average deletion F1 across Hifi datasets with Truvari p=0.5, P=0.5, r=500, O=0.01
- Downstream genotyping accuracy on indel SVs reaches the reported range (~98.32% insertions, 99.01% deletions on Hifi data)

## Limitations

- VolcanoSV-asm's phasing/assignment modules should not be applied to sex chromosomes, whose phasing procedure differs from autosomes
- Benchmark quality of assignments is bounded by the GIAB gold standard, which is not a complete set and could also contain false positives
- VolcanoSV's recall for real complex somatic SVs was relatively lower compared to its performance on simulated data or indel SVs

## Evidence

- [other] Remaining unphased reads are assigned using unique k-mer similarity analysis against haplotype-specific fingerprints: [evidence span withheld — anchor status: not_found]
- [abstract] VolcanoSV uses phased SNPs and unique k-mer similarity analysis for haplotype-resolved discovery: "VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery"
- [results] CHM13 not phased because homozygous: "We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole"
- [readme] VolcanoSV autosome-only design: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
