---
name: switch-error-assessment-against-truth-vcf
description: Use when evaluating whether a phasing method (e.g., MethPhaser vs. SNV-only
  HapCUT2/WhatsHap phasing) on HG002 ONT R9/R10 data introduced phasing errors, by
  comparing output VCFs to the GIAB trio-phased V4.2.1 truth VCF.
when_to_use_negative:
- Benchmarking on sex chromosomes — MethPhaser explicitly does not work on X/Y because
  random X inactivation in females produces inconsistent haplotype methylation patterns
- When no trio-phased truth VCF exists for the sample (e.g., GM12878 or patient samples
  without GIAB truth), since connection correctness cannot be verified
- When SNV-only phasing already produces very high errors (e.g., very high hamming
  distance in R9 80X results), as error attribution to MethPhaser is confounded by
  poor initial SNV phasing
edam_operation: http://edamontology.org/operation_0448
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3569
tools:
- name: WhatsHap
  role: compare module benchmarks phased VCFs against the GIAB trio-phased truth VCF
    to compute switch and flip error rates; stats and haplotag modules support block
    GTF generation and read tagging
- name: MethPhaser
  role: produces the methylation-phased VCF whose newly established phase-block connections
    are assessed for switch errors
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: provides the SNV-only baseline phasing (--ont 1, --outvcf 1) that the methylation-assisted
    result is compared against
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:9247519fa897663d2b77ff0c67d65a125e8412ccc75d3d60c9d8ce5e49066451
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - Clair3
  - HapCUT2
  - MethPhaser
  - WhatsHap
  - meth_phaser_parallel
  - minimap2
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0448
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# switch-error-assessment-against-truth-vcf

## Summary

Benchmark haplotype phasing accuracy (switch and flip error rates) by comparing inferred phase sets against a GIAB trio-phased truth VCF with WhatsHap compare, and normalize new switches by the number of phase-block connections a method establishes.

## When to use

Use when evaluating whether a phasing method (e.g., MethPhaser vs. SNV-only HapCUT2/WhatsHap phasing) on HG002 ONT R9/R10 data introduced phasing errors, by comparing output VCFs to the GIAB trio-phased V4.2.1 truth VCF.

## When NOT to use

- Benchmarking on sex chromosomes — MethPhaser explicitly does not work on X/Y because random X inactivation in females produces inconsistent haplotype methylation patterns
- When no trio-phased truth VCF exists for the sample (e.g., GM12878 or patient samples without GIAB truth), since connection correctness cannot be verified
- When SNV-only phasing already produces very high errors (e.g., very high hamming distance in R9 80X results), as error attribution to MethPhaser is confounded by poor initial SNV phasing

## Inputs

- MethPhaser or SNV-only phased VCF (e.g., output.vcf from meth_phaser_post_processing, HapCUT2 --outvcf 1 output)
- GIAB trio-phased V4.2.1 truth VCF for HG002
- GIAB high-confidence autosomal region BED (X/Y excluded)
- Phase block GTF (WhatsHap stats) for block-level comparisons

## Outputs

- Flip error rate and switch error rate per phasing method (from WhatsHap compare)
- Normalized switch-error increase (new switches / number of MethPhaser-established block connections)
- Corrected phase block N50 after removing incorrectly connected larger blocks
- Benchmark metrics table (e.g., methphaser_hg002_r9_benchmark.csv)

## How to apply

Run WhatsHap compare between each phasing result (SNV-only baseline and methylation-phased VCF) and the GIAB trio-phased V4.2.1 truth VCF on HG002 autosomes to obtain flip and switch error rates. Before computing phase-block N50, remove larger phase blocks that were incorrectly connected relative to the truth VCF, since MethPhaser's block chaining can occasionally join blocks wrongly. To make the switch-error cost comparable across coverages and methods, divide the number of new switches introduced by MethPhaser by the number of phase-block connections it established (reported in Supplementary Data2). Validation thresholds from the paper include a 0.0010%-0.0152% switch-error increase on R9 (30X-80X), ~0.0532% on R10 at 30X, and a 0.02% flip-rate increase at R9 60X.

## Related tools

- **WhatsHap** (compare module benchmarks phased VCFs against the GIAB trio-phased truth VCF to compute switch and flip error rates; stats and haplotag modules support block GTF generation and read tagging)
- **MethPhaser** (produces the methylation-phased VCF whose newly established phase-block connections are assessed for switch errors) — https://github.com/treangenlab/methphaser
- **HapCUT2** (provides the SNV-only baseline phasing (--ont 1, --outvcf 1) that the methylation-assisted result is compared against)

## Examples

```
whatshap compare --truth GIAB.trio.phased.vcf.gz --query output.vcf -o comparison.txt
```

## Evaluation signals

- R9 HG002: switch-error increase within 0.0010%-0.0152% across 30X/60X/80X; N50 fold improvements of 1.78x, 2.34x, 2.53x
- R10 HG002 30X: phase-block N50 of 3,152,506 exceeds the 60X SNV-only N50 of 3,034,229, with ~0.0532% switch-error increase (Fig. 2c)
- R9 60X: MethPhaser N50 = 3,997,227, 1,722 phase gaps, 0.02% flip-rate increase vs. SNV-only
- Invariant: switch-error increase divided by the number of established block connections matches the Supplementary Data2 report
- Check that incorrectly connected larger phase blocks were removed before N50 computation

## Limitations

- Accuracy depends on initial SNV phasing quality; very high SNV phasing errors (high hamming distance in R9 80X) negatively impact MethPhaser accuracy
- Results depend on methylation detection accuracy, overall read length, and a minimum data coverage (>=30X recommended)
- Concrete thresholds and checks MethPhaser uses to uphold phasing accuracy are not detailed in the discussion section

## Evidence

- [discussion] Benchmarking against HG002 truth set: "By benchmarking against HG002, which has a known truth set"
- [methods] Removal of incorrectly connected blocks: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [methods] Supplementary Data2 reporting: "We report this number in Supplementary Data2."
- [discussion] Switch error cost: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [readme] Sex chromosome exclusion: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
