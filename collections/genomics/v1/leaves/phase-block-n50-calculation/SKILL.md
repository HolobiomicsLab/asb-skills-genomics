---
name: phase-block-n50-calculation
description: Use when benchmarking a phasing method (e.g., MethPhaser vs. HapCUT2
  SNV-only phasing) on HG002 or HPRC autosomes against a trio-phased truth set and
  you need corrected phase-block N50, gap counts, or flip/switch error rates.
when_to_use_negative:
- Phasing results on sex chromosomes (X/Y) — random X inactivation makes methylation-based
  connections and the benchmark invalid; autosomes only
- Benchmarks without a trio-phased truth VCF — incorrect block connections cannot
  be identified and removed before N50 computation
- Data below ~30X coverage, where the underlying SNV-only phasing (HapCUT2, WhatsHap)
  is not recommended
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: WhatsHap
  role: compare module benchmarks phased VCFs against the GIAB trio-phased truth VCF
    for flip/switch errors; stats module generates the phase-block GTF used for N50/gap
    computation
- name: MethPhaser
  role: supplies the methylation-phased VCF (output.vcf) whose phase blocks are benchmarked
    against the SNV-only baseline
  repo: github.com/treangenlab/methphaser
- name: HapCUT2
  role: produces the SNV-only phased VCF baseline for the N50 comparison
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:7327cdd5837ff8d7b145b4353f653d1c1de62b6e781c958b00a701c5e8dd0bbb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - HapCUT2
  - MethPhaser
  - WhatsHap
  - WhatsHap compare
  - meth_phaser_parallel
  - minimap2
  - pysam
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# phase-block-n50-calculation

## Summary

Computes haplotype phase-block length N50 (minimum block length such that blocks of that size and larger span ≥50% of total phased length) for SNV-only and methylation-assisted phasing, after removing larger phase blocks that were incorrectly connected relative to the GIAB trio-phased truth VCF.

## When to use

Use when benchmarking a phasing method (e.g., MethPhaser vs. HapCUT2 SNV-only phasing) on HG002 or HPRC autosomes against a trio-phased truth set and you need corrected phase-block N50, gap counts, or flip/switch error rates.

## When NOT to use

- Phasing results on sex chromosomes (X/Y) — random X inactivation makes methylation-based connections and the benchmark invalid; autosomes only
- Benchmarks without a trio-phased truth VCF — incorrect block connections cannot be identified and removed before N50 computation
- Data below ~30X coverage, where the underlying SNV-only phasing (HapCUT2, WhatsHap) is not recommended

## Inputs

- Phased VCF from HapCUT2 (SNV-only baseline, produced with extractHAIRS --ont 1 and hapcut2 --outvcf 1)
- MethPhaser output VCF from meth_phaser_post_processing (output.vcf)
- GIAB trio-phased truth VCF (V4.2.1 for HG002)
- GIAB high-confidence region BED for autosomes
- WhatsHap stats phase-block GTF

## Outputs

- Phase-block N50 per method (e.g., 3,997,227 for MethPhaser HG002 R9 60X vs. 1,706,719 SNV-only)
- Phase gap counts per method (e.g., 1722 vs. 3179)
- Flip and switch error rates from WhatsHap compare (e.g., +0.02% flip, +0.0012% switch)
- Benchmark CSV such as methphaser_hg002_r9_benchmark.csv or methphaser_hprc_benchmark.csv

## How to apply

Restrict the comparison to autosomes, since X/Y are excluded because of random X inactivation. Benchmark both the SNV-only and methylation-phased block sets against the GIAB trio-phased truth VCF with WhatsHap compare; per the methods, first remove the larger phase blocks that were incorrectly connected based on comparison of the SNV-based phasing with the GIAB trio-phased VCF. Then compute N50 as the minimum block length such that all blocks of that size and larger span ≥50% of the total phased length, and count phase gaps between blocks. Report flip and switch error changes as well, expressing the switch-error increase as new switches introduced divided by the number of block connections established. Expected HG002 R9 60X results: N50 rises from 1,706,719 (SNV-only) to 3,997,227 (MethPhaser) with gaps dropping from 3179 to 1722 and only a 0.02% flip-rate increase.

## Related tools

- **WhatsHap** (compare module benchmarks phased VCFs against the GIAB trio-phased truth VCF for flip/switch errors; stats module generates the phase-block GTF used for N50/gap computation)
- **MethPhaser** (supplies the methylation-phased VCF (output.vcf) whose phase blocks are benchmarked against the SNV-only baseline) — github.com/treangenlab/methphaser
- **HapCUT2** (produces the SNV-only phased VCF baseline for the N50 comparison)

## Examples

```
whatshap stats --gtf phase_blocks.gtf output.vcf
```

## Evaluation signals

- N50 invariant: minimum block length such that all larger blocks span ≥50% of total phased length — verify the metric definition used in the script matches this
- HG002 R9 60X reference values: SNV-only N50 1,706,719 vs. MethPhaser 3,997,227; gaps 3179 vs. 1722; flip-rate increase 0.02%; switch-error increase 0.0012%
- Coverage-sweep reference values: N50 fold improvements of 1.78x (30X), 2.34x (60X), 2.53x (80X) with switch error rising only from 0.0010% to 0.0152%
- Confirm incorrectly connected larger phase blocks were removed (per GIAB trio-phased comparison) before N50 computation, and that switch-error increase divided by established connections matches Supplementary Data2

## Limitations

- N50 can be inflated by incorrectly connected blocks, so larger blocks must be removed based on the trio-phased truth comparison before computing N50
- Benchmark excludes X and Y chromosomes because random X inactivation in females leads to an inconsistent haplotype pattern
- MethPhaser depends on initial SNV phasing accuracy — high SNV phasing errors (e.g., very high hamming distance in R9 80X results) negatively impact accuracy

## Evidence

- [methods] Removal of incorrectly connected larger phase blocks before N50 calculation: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [discussion] Excluding X and Y chromosomes from benchmark: "Thus, we exclude the X and Y chromosomes from our benchmark"
- [readme] Autosomes-only scope: "!!! Note: MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes. !!!"
- [discussion] Minimal switch-error cost of block joining: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
