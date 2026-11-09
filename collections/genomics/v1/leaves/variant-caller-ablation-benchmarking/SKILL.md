---
name: variant-caller-ablation-benchmarking
description: Decision support for quantifying whether a variant caller's prior-knowledge
  component (e.g. known common variants as variation backgrounds) measurably improves
  precision, recall, or F1 on benchmark datasets such as HG002 ONT k12/k14 reads.
when_to_use_negative:
- The goal is to compare different variant callers against each other (e.g. miniSNV
  vs Clair3 vs Longshot) rather than to isolate the effect of a single caller's internal
  feature via ablation.
- Input is raw signal/FASTQ without a sorted, indexed BAM aligned to GRCh38 — miniSNV
  requires samtools-indexed sorted BAM input.
- The organism or sample lacks a population VCF of known common variants, so the variation-background
  index cannot be built.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: miniSNV
  role: SNV caller whose known-common-variant background is ablated; run twice per
    dataset with and without --useindex
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform providing the HG002 k12 and k14 benchmark reads
- name: Dorado
  role: Basecaller for the k14 (kit v14) ONT read deposits used in the ablation
  repo: https://github.com/nanoporetech/dorado
- name: Clair3
  role: Baseline variant caller for comparison; latest ONT model from rerio adopted
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Diploid SNV calling baseline used in the k12 comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:4fe642d4895839639030dea9193399d9d87edf2b0b1fe04ed2e3ab5ab380725c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - Dorado
  - Longshot
  - Oxford Nanopore Technologies (ONT)
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# variant-caller-ablation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An ablation benchmarking protocol that quantifies how a variant caller's population-variation background (e.g. miniSNV's known-common-variant index from the 1000 Genomes VCF) affects SNV calling precision, recall, and F1 against a GIAB truth set. It isolates the contribution of prior-knowledge features by running the caller with the feature enabled versus disabled under otherwise identical parameters.

## When to use

Decision support for quantifying whether a variant caller's prior-knowledge component (e.g. known common variants as variation backgrounds) measurably improves precision, recall, or F1 on benchmark datasets such as HG002 ONT k12/k14 reads.

## When NOT to use

- The goal is to compare different variant callers against each other (e.g. miniSNV vs Clair3 vs Longshot) rather than to isolate the effect of a single caller's internal feature via ablation.
- Input is raw signal/FASTQ without a sorted, indexed BAM aligned to GRCh38 — miniSNV requires samtools-indexed sorted BAM input.
- The organism or sample lacks a population VCF of known common variants, so the variation-background index cannot be built.

## Inputs

- Sorted, samtools-indexed BAM of ONT reads aligned to GRCh38 (HG002 k12 Guppy-basecalled; k14 Dorado-basecalled)
- GRCh38 FASTA reference (samtools indexed)
- 1000 Genomes population VCF for building the known-common-variant index
- Prebuilt miniSNV index folder (five files) from the released index
- GIAB HG002 truth VCF and GIAB v3.3 stratification BEDs

## Outputs

- ablation_table.csv with precision, recall, and F1 for the four dataset x background conditions (k12/k14 x index-on/index-off)
- Per-condition VCF callsets from miniSNV runs
- Reported deltas: k12 precision +0.12%, F1 +0.04%; k14 precision +0.08%, recall +0.16%, F1 +0.12%

## How to apply

Obtain the benchmark sample's long reads (HG002 k12 ligation kit v12 R10.4 FLO-PRO112 simplex Guppy-basecalled, and k14 Dorado-basecased ONT data), align to GRCh38 with Minimap2, and produce samtools-indexed sorted BAMs, since miniSNV filters alignments below MAPQ 20. Build the known-common-variant index from the 1000 Genomes population VCF using miniSNV's windowing scheme (fixed window L=300 bp with half-length overlap; multiallelic loci split to biallelic with alternative allele frequency >5% recorded as wid_pos_ref_alt UIDs). Run miniSNV twice per dataset: once with --useindex and the known-variant index (known loci classified high-quality when R_alt > 0.3, unknown loci when R_alt > 0.7 with no other candidate within 10 bp), and once without the index so all candidates follow the unknown-locus rule (default R_alt threshold 0.2, MAPQ 20). Evaluate all four callsets against the GIAB HG002 truth set with GIAB v3.3 stratifications, compute precision/recall/F1, and tabulate into ablation_table.csv to compare directional deltas.

## Related tools

- **miniSNV** (SNV caller whose known-common-variant background is ablated; run twice per dataset with and without --useindex) — https://github.com/CuiMiao-HIT/miniSNV
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform providing the HG002 k12 and k14 benchmark reads)
- **Dorado** (Basecaller for the k14 (kit v14) ONT read deposits used in the ablation) — https://github.com/nanoporetech/dorado
- **Clair3** (Baseline variant caller for comparison; latest ONT model from rerio adopted) — https://github.com/nanoporetech/rerio
- **Longshot** (Diploid SNV calling baseline used in the k12 comparison)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_k14.sorted.bam --workDir workdir --useindex -i index_folder --human
```

## Evaluation signals

- The regenerated ablation_table.csv reproduces the directional effect: precision and F1 are higher with the known-variant background enabled than disabled on both k12 and k14.
- With the known-variant background enabled, miniSNV exceeds 99% F1 on the k12 dataset.
- k12 gains over baselines reproduce: +0.02% precision over Clair3 and +0.05%/+1.83%/+0.96% precision/recall/F1 over Longshot, with only miniSNV exceeding 99% F1.
- All callsets are evaluated with GIAB v3.3 stratifications applied; metrics computed without stratification should not match the published deltas.
- Index-on runs use known-locus R_alt > 0.3 vs unknown-locus R_alt > 0.7 thresholds; index-off runs apply R_alt > 0.2 uniformly at MAPQ 20 — parameter logs should confirm this distinction.

## Limitations

- The known-variant index depends on population VCF coverage (1000 Genomes by default); variants absent from the population fall back to the stricter unknown-locus rule (R_alt > 0.7) and may lose sensitivity.
- R_alt thresholds of 0.3–0.7 were selected via in-house experiments (Supplementary Table S1), so sensitivity of results to these cutoffs is not exhaustively characterized.
- Benchmarks rest on a single GIAB trio sample (HG002 primary, with HG003/HG004 in the broader evaluation); conclusions may not transfer to non-human or low-variation-background genomes.
- miniSNV's low-complexity regions are classified as low-quality via GIAB v3.3 annotations, so ablation metrics are conditional on those stratification definitions.

## Evidence

- [results] Incorporating known common variants improved miniSNV's metrics: on k12-data, precision increased by 0.12% and F1-score by 0.04%; on k14-data, precision increased by 0.08%, recall by 0.16%, and F1-score by 0.12%.: [evidence span withheld — anchor status: not_found]
- [methods] High-quality locus classification thresholds differ for known vs unknown loci.: [evidence span withheld — anchor status: not_found]
