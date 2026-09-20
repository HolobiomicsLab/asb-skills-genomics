---
name: truth-set-genotype-concordance-evaluation
description: Use when a genotyping stage (e.g., RetroInspector's Sniffles2-based genotyper)
  must be scored against a curated, genotyped truth set such as the 11-case HGSVC2
  cohort, to quantify per-allele genotype accuracy.
when_to_use_negative:
- Evaluating insertion *detection* (precision/recall of call presence) against the
  HGSVC 5,630-insertion truth set — that is a call-level benchmark, not a genotype
  concordance evaluation
- No curated, genotyped truth set exists for the cohort, so concordance cannot be
  computed and results are uninterpretable
- The goal is TE annotation, enrichment, or reassembly quality rather than genotyping
  accuracy
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RetroInspector
  role: Snakemake pipeline whose genotyping stage (Sniffles2 algorithm reimplementation)
    is being benchmarked; discards 0/0-genotyped variants
  repo: https://github.com/javiercguard/retroinspector
- name: Sniffles2
  role: Variant caller whose calls feed the genotyping benchmark; best precision (96.99%)
    achieved combined with cuteSV
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: cuteSV
  role: Variant caller combined with Sniffles2 to produce the callset that achieved
    the best genotype concordance
  repo: https://github.com/tjiangHIT/cuteSV
- name: minimap2
  role: Aligns the 11 HGSVC2 sample FASTQs to the GRCh38 reference before variant
    calling
  repo: https://github.com/lh3/minimap2
- name: mosdepth
  role: Computes per-sample genome-wide coverage consumed by the genotyping stage
    (98.3% of genotyping runtime)
  repo: https://github.com/brentp/mosdepth
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:0698beec5b946d6716592059a3f93347ad4d88a7c38b24f2ee61e962445621e4
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
  - Sniffles2
  - cuteSV
  - minimap2
  - mosdepth
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# truth-set-genotype-concordance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a variant genotyping stage by computing per-allele genotype concordance of called TE insertion/deletion genotypes against a curated truth set (the HGSV2/HGSVC2 cohort), yielding precision metrics per variant-caller combination.

## When to use

Use when a genotyping stage (e.g., RetroInspector's Sniffles2-based genotyper) must be scored against a curated, genotyped truth set such as the 11-case HGSVC2 cohort, to quantify per-allele genotype accuracy.

## When NOT to use

- Evaluating insertion *detection* (precision/recall of call presence) against the HGSVC 5,630-insertion truth set — that is a call-level benchmark, not a genotype concordance evaluation
- No curated, genotyped truth set exists for the cohort, so concordance cannot be computed and results are uninterpretable
- The goal is TE annotation, enrichment, or reassembly quality rather than genotyping accuracy

## Inputs

- 11 HGSVC2 sample FASTQ files (nanopore reads)
- GRCh38 human reference genome (hg38 FASTA)
- HGSV2 curated genotyped truth set for the HGSVC2 subset

## Outputs

- genotyping_benchmark_metrics.tsv with per-allele genotype concordance / precision per variant-caller combination
- Genotyped insertions and deletions with 0/0 variants discarded

## How to apply

Align each sample's nanopore FASTQs to the GRCh38 reference with minimap2 and call variants with cuteSV and Sniffles2 using RetroInspector's default minimum of 3 supporting reads. Compute per-sample genome-wide coverage with mosdepth and genotype insertions and deletions using RetroInspector's reimplementation of the Sniffles2 algorithm, discarding variants genotyped 0/0 (homozygous reference). Restrict the benchmark to autosomal-chromosome variants that are also present in the HGSV2 curated genotyped truth set. Score per-allele genotype concordance, counting a 0/1 genotype as one true positive, 1/1 as two, and 0/0 as none, with the number of predicted positives equal to the number of analyzed variants multiplied by two; repeat per caller combination (e.g., cuteSV+Sniffles2, strict vs. lax filtering) and report precision, where the article reports >95% for all combinations and a best of 96.99% for cuteSV and Sniffles2.

## Related tools

- **RetroInspector** (Snakemake pipeline whose genotyping stage (Sniffles2 algorithm reimplementation) is being benchmarked; discards 0/0-genotyped variants) — https://github.com/javiercguard/retroinspector
- **Sniffles2** (Variant caller whose calls feed the genotyping benchmark; best precision (96.99%) achieved combined with cuteSV) — https://github.com/fritzsedlazeck/Sniffles
- **cuteSV** (Variant caller combined with Sniffles2 to produce the callset that achieved the best genotype concordance) — https://github.com/tjiangHIT/cuteSV
- **minimap2** (Aligns the 11 HGSVC2 sample FASTQs to the GRCh38 reference before variant calling) — https://github.com/lh3/minimap2
- **mosdepth** (Computes per-sample genome-wide coverage consumed by the genotyping stage (98.3% of genotyping runtime)) — https://github.com/brentp/mosdepth

## Examples

```
snakemake --use-conda -np --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- Per-allele genotype precision exceeds 95% for every variant-caller combination tested, with cuteSV+Sniffles2 reaching 96.99%
- Scoring invariants hold: 0/1 truth genotypes count as 1 true positive, 1/1 as 2, 0/0 as 0, and predicted positives = number of analyzed variants × 2
- The analyzed callset contains no 0/0-genotyped variants and is restricted to autosomal chromosomes present in the HGSV2 truth set
- Results are written to genotyping_benchmark_metrics.tsv with one metric row per caller combination (strict vs. lax filtering included)
- Per-allele concordance is reported against the HGSV2 truth set rather than raw call overlap

## Limitations

- The publicly available HGSVC data was basecalled with an older basecaller, and even after reassembly sequences may be imperfect, particularly for homopolymers
- Benchmark restricted to autosomal variants also present in the curated truth set, so unrepresented or sex-chromosome genotypes are excluded from the score
- RetroInspector's comparative evaluation could not include xTea, so the genotyping comparison covers only the tested caller combinations

## Evidence
