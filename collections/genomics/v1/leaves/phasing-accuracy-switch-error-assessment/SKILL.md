---
name: phasing-accuracy-switch-error-assessment
description: Use when evaluating whether joining SNV phase blocks (e.g., with MethPhaser)
  improved phase block N50 without inflating switch error or hamming distance, using
  truth VCFs (GIAB HG002), WhatsHap compare, or neighbor-relationship vote outputs.
when_to_use_negative:
- Phasing sex chromosomes (X/Y) — random X inactivation in females produces inconsistent
  haplotype methylation patterns, so the benchmark explicitly excludes X and Y
- Evaluating haplotype-specific SV phasing as a validated claim — the article states
  this requires more experimental results and is only future benchmarking work
- Samples with very high initial SNV phasing errors (e.g., very high hamming distance
  in R9 80x data) — MethPhaser accuracy is negatively impacted and downstream assessment
  inherits those errors
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: WhatsHap compare
  role: Computes switch and hamming error between phased VCFs (SNV-only vs methylation-phased)
    and against truth sets
  repo: https://github.com/treangenlab/methphaser
- name: WhatsHap
  role: SNV-based phasing baseline and haplotagging prior to accuracy assessment
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Produces the SNV-phased VCF (--ont 1) that serves as the before-MethPhaser
    comparison and error baseline
- name: MethPhaser
  role: Emits block relationship outputs used to verify each SNV phase block connection
    against the truth set
  repo: https://github.com/treangenlab/methphaser
- name: SAMtools
  role: Counts haplotagged reads via HP tags as a benchmark metric
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:fb30696bdd564e9e026d2f04b967653b1ad10df9e2869fbe54db7d82aec746bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - Clair3
  - HapCUT2
  - MethPhaser
  - SAMtools
  - WhatsHap
  - WhatsHap compare
  - meth_phaser_parallel
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# phasing-accuracy-switch-error-assessment

## Summary

Benchmark haplotype phasing accuracy by computing switch and hamming error against a truth set (e.g., GIAB trio-phased HG002) or via WhatsHap compare and MethPhaser's neighbor-relationship votes, so that block-joining gains (N50 improvements) are not achieved at the cost of phasing correctness.

## When to use

Use when evaluating whether joining SNV phase blocks (e.g., with MethPhaser) improved phase block N50 without inflating switch error or hamming distance, using truth VCFs (GIAB HG002), WhatsHap compare, or neighbor-relationship vote outputs.

## When NOT to use

- Phasing sex chromosomes (X/Y) — random X inactivation in females produces inconsistent haplotype methylation patterns, so the benchmark explicitly excludes X and Y
- Evaluating haplotype-specific SV phasing as a validated claim — the article states this requires more experimental results and is only future benchmarking work
- Samples with very high initial SNV phasing errors (e.g., very high hamming distance in R9 80x data) — MethPhaser accuracy is negatively impacted and downstream assessment inherits those errors

## Inputs

- Phased VCF from HapCUT2 or WhatsHap (SNV-only phasing)
- Methylation-phased VCF emitted by meth_phaser_post_processing (-ov)
- GIAB trio-phased HG002 truth VCF (or other truth set)
- Haplotagged BAM with HP tags (from WhatsHap haplotag)
- WhatsHap stats GTF of SNV phase blocks

## Outputs

- Switch error and hamming distance metrics per sample/chromosome
- Phase block N50 and phase block counts before vs after methylation phasing
- Medically relevant gene block counts
- Haplotagged read counts from SAMtools HP tags
- Benchmark table (e.g., methphaser_hprc_benchmark.csv)

## How to apply

Run WhatsHap compare between the phased VCF (SNV-only vs methylation-phased) and a truth set such as the GIAB trio-phased HG002 VCF to compute switch error and hamming distance. For MethPhaser, use its relationship output to verify each SNV phase block connection and, when a truth VCF is supplied via -vt, remove incorrectly connected larger phase blocks before computing N50. In the HPRC benchmark, confirm expected error behavior: HG01109 chr1 showed +5% hamming distance with only one switched block (Fig. 5a, Supplementary Data 2-4), while block joining led to 'only a minimal increase in phasing error (i.e., switch error)'. Per sample, also compute phase block N50 (minimum block length such that all larger blocks span ≥ 50% of total phase length), block counts, medically relevant gene block counts, and haplotagged read counts via SAMtools HP tags.

## Related tools

- **WhatsHap compare** (Computes switch and hamming error between phased VCFs (SNV-only vs methylation-phased) and against truth sets) — https://github.com/treangenlab/methphaser
- **WhatsHap** (SNV-based phasing baseline and haplotagging prior to accuracy assessment) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Produces the SNV-phased VCF (--ont 1) that serves as the before-MethPhaser comparison and error baseline)
- **MethPhaser** (Emits block relationship outputs used to verify each SNV phase block connection against the truth set) — https://github.com/treangenlab/methphaser
- **SAMtools** (Counts haplotagged reads via HP tags as a benchmark metric)

## Examples

```
whatshap compare --sample HG01109 truth.vcf.gz methylation_phased.vcf.gz
```

## Evaluation signals

- HG01109 chr1 evaluation shows +5% hamming distance with only one switched block, matching Fig. 5a and Supplementary Data 2-4
- Joining phase blocks produces only a minimal increase in switch error
- N50 improvement falls within the reported 1.69x–2.55x range (HG03098 to HG02080)
- Phase block counts drop from 947→643 (HG03098) and 1087→743 (HG02080)
- Incorrectly connected larger phase blocks are removed before N50 calculation via comparison with the GIAB trio-phased VCF

## Limitations

- Accuracy depends on initial SNV phasing quality: high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact MethPhaser accuracy
- Results depend on accuracy of methylation detection, overall read length, and minimum coverage; ≥30X ONT coverage is recommended
- Concrete thresholds and checks MethPhaser uses to uphold phasing accuracy are not fully detailed in the discussion section
- Cannot improve phasing of sex chromosomes in clinical samples due to random X inactivation in females

## Evidence

- [discussion] Joining phase blocks leads to only a minimal increase in phasing error (switch error): "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [discussion] Benchmarking against HG002 truth set: "By benchmarking against HG002, which has a known truth set"
- [methods] N50 accuracy on HG002 via relationship output: "we can easily know the correctness of MethPhaser’s each SNV phase block connection with our relationship output and further get the correct N50 on the HG002 sample"
- [methods] Removal of incorrectly connected blocks before N50: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB"
- [readme] README: sex chromosome exclusion: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
- [readme] README: coverage recommendation: "We recommend using MethPhaser for at least 30X coverage ONT reads since most of the SNV phasing methods (HapCUT2, Whatshap) only work reasonably above that coverage"
