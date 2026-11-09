---
name: autosomal-variant-subsetting
description: Use when benchmarking TE insertion/deletion genotype calls from nanopore
  data against a curated truth set and you need to exclude non-autosomal regions and
  homozygous-reference calls before scoring concordance.
when_to_use_negative:
- The goal is discovery of novel TE insertions rather than genotyping accuracy benchmarking
  against a curated truth set
- The callset has not been genotyped (no per-sample 0/0, 0/1, 1/1 genotype assignments
  available)
- Variants of interest lie on non-autosomal chromosomes or allosomes, which are excluded
  by this skill
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: RetroInspector
  role: Runs the genotyping stage (Sniffles2-algorithm reimplementation) and applies
    the 0/0 filter before subsetting
  repo: https://github.com/javiercguard/retroinspector
- name: mosdepth
  role: Computes per-sample genome-wide coverage consumed by the genotyping step
  repo: https://github.com/brentp/mosdepth
- name: Sniffles2
  role: Variant caller whose calls are genotyped and whose algorithm RetroInspector
    reimplements for genotyping
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: minimap2
  role: Aligns the 11 HGSVC2 sample FASTQ files to GRCh38 prior to variant calling
    and genotyping
  repo: https://github.com/lh3/minimap2
- name: Snakemake
  role: Orchestrates the RetroInspector pipeline stages including genotyping and metric
    output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:85624be5d050befe79215ee9f6118c5f6b590e9c5d17d4e58cb543f4e5234d64
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - RetroInspector
  - Snakemake
  - Sniffles2
  - minimap2
  - mosdepth
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# autosomal-variant-subsetting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Restricts a structural-variant callset to autosomal-chromosome variants overlapping a curated truth set and scores per-allele genotype concordance, as RetroInspector does against the HGSV2/HGSVC2 cohort genotyping truth set.

## When to use

Use when benchmarking TE insertion/deletion genotype calls from nanopore data against a curated truth set and you need to exclude non-autosomal regions and homozygous-reference calls before scoring concordance.

## When NOT to use

- The goal is discovery of novel TE insertions rather than genotyping accuracy benchmarking against a curated truth set
- The callset has not been genotyped (no per-sample 0/0, 0/1, 1/1 genotype assignments available)
- Variants of interest lie on non-autosomal chromosomes or allosomes, which are excluded by this skill

## Inputs

- Genotyped insertion/deletion VCF-like callsets from RetroInspector's genotyping stage (Sniffles2-algorithm reimplementation)
- HGSV2 curated genotyped truth set for the 11-case HGSVC2 cohort subset
- Per-sample genome-wide coverage from mosdepth
- FASTQ files of the 11 HGSVC2 samples aligned to GRCh38 with minimap2

## Outputs

- Autosomal, truth-set-restricted callset with 0/0 genotypes removed
- Per-allele genotype concordance metrics written to genotyping_benchmark_metrics.tsv

## How to apply

After genotyping insertions and deletions with RetroInspector's Sniffles2-algorithm reimplementation (using genome-wide coverage computed with mosdepth), discard variants genotyped 0/0 (homozygous reference) across samples. Restrict the callset to autosomal chromosomes only, and further intersect with variants present in the HGSV2 curated genotyped truth set. Score per-allele genotype concordance by counting each 0/1 truth genotype as one true positive and each 1/1 as two, while the number of predicted positives equals the number of analyzed variants multiplied by two. Write the resulting metrics (e.g., precision) to genotyping_benchmark_metrics.tsv; accept the output when concordance with the truth set is high, as the paper reports precision exceeding 95% for all caller combinations (best 96.99% with cuteSV and Sniffles2).

## Related tools

- **RetroInspector** (Runs the genotyping stage (Sniffles2-algorithm reimplementation) and applies the 0/0 filter before subsetting) — https://github.com/javiercguard/retroinspector
- **mosdepth** (Computes per-sample genome-wide coverage consumed by the genotyping step) — https://github.com/brentp/mosdepth
- **Sniffles2** (Variant caller whose calls are genotyped and whose algorithm RetroInspector reimplements for genotyping) — https://github.com/fritzsedlazeck/Sniffles
- **minimap2** (Aligns the 11 HGSVC2 sample FASTQ files to GRCh38 prior to variant calling and genotyping) — https://github.com/lh3/minimap2
- **Snakemake** (Orchestrates the RetroInspector pipeline stages including genotyping and metric output)

## Examples

```
snakemake --use-conda -c 8 --config referenceGenome=/home/user/hg38.fa outputPath=/home/user/experiment_results fastq_directory=/home/user/data/samples
```

## Evaluation signals

- Per-allele genotype concordance against the HGSV2 truth set is high (paper reports >95% precision for all variant caller combinations)
- Best-case precision of 96.99% is reproducible with cuteSV and Sniffles2 at the default minimum read support of 3
- No variants with 0/0 genotype remain in the analyzed callset
- All analyzed variants are on autosomal chromosomes and present in the HGSV2 truth set
- Metrics file genotyping_benchmark_metrics.tsv exists and contains precision/concordance values

## Limitations

- Benchmark restricted to autosomal variants also present in the curated truth set, so calls outside the truth set are not evaluated
- Publicly available HGSVC data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers
- The number of predicted positives is fixed at twice the number of analyzed variants (per-allele counting), which assumes diploid autosomal genotypes

## Evidence

- [results] Restrict the benchmark to autosomal-chromosome variants that are also present in the HGSV2 curated genotyped truth set.: [evidence span withheld — anchor status: not_found]
- [results] RetroInspector's genotyping results were highly concordant with the HGSV2 truth set: [evidence span withheld — anchor status: not_found]
- [results] These genotypes are then used to filter the variants that are found to have a 0/0 genotype (homozygous for the reference allele).: "These genotypes are then used to filter the variants that are found to have a 0/0 genotype (homozygous for the reference allele)."
- [results] Genotyping insertions and deletions consumed 37.83 ± 3.59 min per sample, with 98.3% of this time dedicated to calculating genome-wide coverage.: "Genotyping insertions and deletions consumed 37.83 ± 3.59 min per sample, with 98.3% of this time dedicated to calculating genome-wide coverage."
- [results] a subset of the HGSV2 cohort24 was used. RetroInspector’s results and the truth set were highly concordant: "a subset of the HGSV2 cohort24 was used. RetroInspector’s results and the truth set were highly concordant"
