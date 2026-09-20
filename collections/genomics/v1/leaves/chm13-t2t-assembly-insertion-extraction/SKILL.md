---
name: chm13-t2t-assembly-insertion-extraction
description: Use when validating novel-sequence-insertion calls against a haploid
  genome truth set, specifically when CHM13 synthetic long-read callsets need to be
  confirmed against the T2T-CHM13 assembly (target ~78% confirmation rate).
when_to_use_negative:
- The truth set should come from a diploid genome where assembly-derived insertions
  cannot be treated as unambiguous ground truth (CHM13/CHM1 haploid status is what
  makes this benchmark valid).
- Insertions shorter than 300 bp are the target — the paper's filters and analysis
  only cover insertions > 300 bp with anchor length sums exceeding 300 bp.
- A caller-specific benchmark (e.g. GIAB HG002 Snakemake evaluation) is required rather
  than assembly-vs-reference insertion confirmation.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: Minimap2
  role: Aligns the T2T-CHM13 assembly v2.0 to the GRCh38 reference genome to reveal
    assembly-vs-reference insertions
- name: Novel-X
  role: Produces the CHM13 insertion VCF callsets being validated against the T2T-derived
    benchmark
  repo: github.com/1dayac/Novel-X
- name: LongRanger
  role: Converts/aligns 10x and TELL-Seq CHM13 synthetic long-read data to GRCh38
    so it is LongRanger-compatible before Novel-X
- name: stlfr2supernova_pipeline
  role: Converts stLFR CHM13 reads into a 10x-compatible format for the Novel-X workflow
  repo: github.com/BGI-Qingdao/stlfr2supernova_pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:63f74e9c2fa337e3a2c6b9671485194a767445570a014746648a5f74f4858510
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Minimap2
  - Novel-X
  - stlfr2supernova
  - stlfr2supernova_pipeline
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chm13-t2t-assembly-insertion-extraction

## Summary

Builds a benchmark set of non-reference sequence insertions in CHM13 by aligning the T2T consortium CHM13 assembly v2.0 to the GRCh38 reference with Minimap2 and extracting long insertion events, so that caller output (e.g. Novel-X VCFs) can be scored for confirmation rate.

## When to use

Use when validating novel-sequence-insertion calls against a haploid genome truth set, specifically when CHM13 synthetic long-read callsets need to be confirmed against the T2T-CHM13 assembly (target ~78% confirmation rate).

## When NOT to use

- The truth set should come from a diploid genome where assembly-derived insertions cannot be treated as unambiguous ground truth (CHM13/CHM1 haploid status is what makes this benchmark valid).
- Insertions shorter than 300 bp are the target — the paper's filters and analysis only cover insertions > 300 bp with anchor length sums exceeding 300 bp.
- A caller-specific benchmark (e.g. GIAB HG002 Snakemake evaluation) is required rather than assembly-vs-reference insertion confirmation.

## Inputs

- T2T-CHM13 assembly v2.0 FASTA (chm13v2.0.fa.gz or chm13v2.0_noY.fa.gz for benchmarking)
- GRCh38 reference genome FASTA
- Novel-X VCF callsets for CHM13 synthetic long-read datasets (10x Chromium, TELL-Seq, stLFR)

## Outputs

- Minimap2 alignment of T2T-CHM13 assembly to GRCh38 (e.g. PAF/SAM)
- Benchmark insertion set extracted from the T2T alignment
- chm13_t2t_validation_summary.csv with confirmation counts and confirmation rate per platform

## How to apply

Download the T2T consortium CHM13 assembly v2.0 (e.g. chm13v2.0.fa.gz from the nanopore-wgs-consortium/chm13 repository) and align it to GRCh38 using Minimap2. Extract long insertion calls from the resulting alignment to form the benchmark insertion set — events corresponding to sequence present in CHM13 but absent from GRCh38. Match Novel-X CHM13 VCF calls (insertions > 300 bp with anchor lengths summing to more than 300 bp, per the default filters) against this T2T-derived insertion set, computing the fraction of confirmed calls per platform (10x Genomics, TELL-Seq, stLFR). Write confirmation counts and rates into a numeric summary table (chm13_t2t_validation_summary.csv); a reproduced rate near the reported 78% indicates the pipeline was applied correctly. The haploid nature of CHM13 means insertions in the T2T assembly are essentially true positive ground truth, avoiding diploid genotype ambiguity.

## Related tools

- **Minimap2** (Aligns the T2T-CHM13 assembly v2.0 to the GRCh38 reference genome to reveal assembly-vs-reference insertions)
- **Novel-X** (Produces the CHM13 insertion VCF callsets being validated against the T2T-derived benchmark) — github.com/1dayac/Novel-X
- **LongRanger** (Converts/aligns 10x and TELL-Seq CHM13 synthetic long-read data to GRCh38 so it is LongRanger-compatible before Novel-X)
- **stlfr2supernova_pipeline** (Converts stLFR CHM13 reads into a 10x-compatible format for the Novel-X workflow) — github.com/BGI-Qingdao/stlfr2supernova_pipeline

## Examples

```
minimap2 -ax asm5 GRCh38.fa chm13v2.0.fa.gz > chm13_vs_grch38.sam; novel-x.py run --ref GRCh38 --bam chm13_10x.bam
```

## Evaluation signals

- Reproduced confirmation rate of Novel-X CHM13 calls against the T2T-derived benchmark is approximately 78%.
- Only insertions longer than 300 bp with anchor lengths summing to more than 300 bp are retained in the callset before matching.
- chm13_t2t_validation_summary.csv contains numeric confirmation counts and rates for each platform (10x, TELL-Seq, stLFR).
- The benchmark extraction is derived from a Minimap2 alignment of chm13v2.0 against GRCh38, not from simulated data.
- Novel-X calls matched to T2T insertions overlap in both sequence content and reference placement.

## Limitations

- Novel-X needs a substantial number of distinct barcodes per insertion; small insertions usually lack sufficient coverage for anchor assembly, so the benchmark only covers events > 300 bp.
- The CHM13 sequencing data used span multiple platforms (10x Genomics, TELL-Seq, stLFR) and platform-specific conversion pipelines (LongRanger, stlfr2supernova) can introduce format incompatibilities.
- The T2T assembly itself may contain errors; only well-characterized haploid genomes such as CHM13 make assembly-derived truth tractable.

## Evidence

- [supplementary] The CHM13 T2T assembly v2.0. used for CHM13 calls validation can be found at https://github.com/nanopore-wgs-consortium/chm13.: "The CHM13 T2T assembly v2.0. used for CHM13 calls validation can be found at https://github.com/nanopore-wgs-consortium/chm13."
- [methods] the recent T2T consortium (45) assembly (v2.0) of the CHM13 genome: "the recent T2T consortium (45) assembly (v2.0) of the CHM13 genome"
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [methods] our CHM1 or CHM13 haploid datasets: "they do not work with our CHM1 or CHM13 haploid datasets"
- [other] Novel-X is reported to be highly sensitive when tested on both simulated data and real data, including well-characterized haploid and diploid human genomes.: [evidence span withheld — anchor status: not_found]
- [readme] Use this file for benchmarking assemblies of CHM13.: [evidence span withheld — anchor status: not_found]
