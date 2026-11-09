---
name: giab-truth-set-block-connection-verification
description: Use when benchmarking haplotype phase-block accuracy (flip/switch errors,
  N50, gap counts) of SNV-only vs methylation-extended phasing on HG002 against the
  GIAB trio-phased V4.2.1 truth VCF.
when_to_use_negative:
- Phasing or verifying sex chromosomes (chrX/chrY) — random X inactivation in females
  produces inconsistent haplotype patterns, and MethPhaser explicitly does not work
  there
- Genomes without a trio-phased truth set (e.g., patient blood samples) — block-connection
  correctness cannot be independently established, so N50 cannot be corrected
- 'Before the post-processing step: comparing raw HapCUT2 output rather than the haplotagged
  BAM and phased VCF that MethPhaser requires as input'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: WhatsHap
  role: Provides the compare operation against the GIAB trio-phased truth VCF, plus
    haplotag and stats modules used to generate haplotagged BAMs and phase-block GTFs
    feeding the evaluation
- name: MethPhaser
  role: Produces the methylation-extended phase blocks and the relationship output
    whose connections are being verified against the truth set
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Generates the baseline SNV-only phased VCF (extractHAIRS --ont 1; hapcut2
    --outvcf 1) that serves as the comparison arm for flip/switch error rates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:2a0db69046075ddd93e94d3854fdbd85838327d07136301e8025ab3c979a3cfc
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# giab-truth-set-block-connection-verification

## Summary

Verifies the correctness of each SNV phase-block connection made by SNV-only or MethPhaser phasing on HG002 autosomes against the GIAB trio-phased V4.2.1 truth VCF, quantifying flip/switch error rates and removing incorrectly connected larger blocks so that a corrected phase-length N50 can be reported.

## When to use

Use when benchmarking haplotype phase-block accuracy (flip/switch errors, N50, gap counts) of SNV-only vs methylation-extended phasing on HG002 against the GIAB trio-phased V4.2.1 truth VCF.

## When NOT to use

- Phasing or verifying sex chromosomes (chrX/chrY) — random X inactivation in females produces inconsistent haplotype patterns, and MethPhaser explicitly does not work there
- Genomes without a trio-phased truth set (e.g., patient blood samples) — block-connection correctness cannot be independently established, so N50 cannot be corrected
- Before the post-processing step: comparing raw HapCUT2 output rather than the haplotagged BAM and phased VCF that MethPhaser requires as input

## Inputs

- SNV-only phased VCF (HapCUT2 with extractHAIRS --ont 1, or WhatsHap phase output)
- MethPhaser post-processed phased VCF (meth_phaser_post_processing -ov output)
- MethPhaser relationship output identifying individual SNV phase-block connections
- GIAB trio-phased V4.2.1 truth VCF for HG002
- GIAB high-confidence regions BED (autosomes only)

## Outputs

- Flip and switch error rates for SNV-only vs MethPhaser phasing relative to the truth VCF
- Corrected phase-block N50 (after removing incorrectly connected larger blocks)
- Phase-gap counts per method (e.g., 3179 SNV-only vs 1722 MethPhaser on HG002 R9 60X)
- Per-connection correctness labels validated against Supplementary Data2

## How to apply

Restrict the comparison to GIAB high-confidence autosomal regions (X and Y are excluded from the benchmark because random X inactivation makes haplotype patterns inconsistent), after filtering Clair3 SNVs at allele frequency > 0.25. Run WhatsHap compare on both the SNV-only and the MethPhaser phase block sets against the GIAB trio-phased V4.2.1 truth VCF to obtain flip and switch error rates per method. Because MethPhaser's relationship output reports each block connection, benchmark against the truth VCF to identify which larger phase blocks were incorrectly connected, and remove those blocks before computing N50 — otherwise the joined-block N50 is overstated. Compute the phase block N50 as the minimum block length such that all larger blocks span >= 50% of total phased length, and count phase gaps for both methods. Cross-validate by checking that the switch-error increase divided by the number of established block connections matches the per-connection error rate reported in Supplementary Data2.

## Related tools

- **WhatsHap** (Provides the compare operation against the GIAB trio-phased truth VCF, plus haplotag and stats modules used to generate haplotagged BAMs and phase-block GTFs feeding the evaluation)
- **MethPhaser** (Produces the methylation-extended phase blocks and the relationship output whose connections are being verified against the truth set) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Generates the baseline SNV-only phased VCF (extractHAIRS --ont 1; hapcut2 --outvcf 1) that serves as the comparison arm for flip/switch error rates)

## Examples

```
whatshap compare --sample HG002 MethPhaser.phased.vcf.gz GIAB.trio.V4.2.1.vcf.gz
```

## Evaluation signals

- On HG002 autosomes with ONT R9 60X reads, expect a 0.02% flip-rate increase and ~0.0012% switch-error increase for MethPhaser vs SNV-only phasing
- Expected corrected N50 values: 1,706,719 (SNV-only) vs 3,997,227 (MethPhaser); gap counts 3179 vs 1722
- Switch-error increase divided by the number of established block connections must match the Supplementary Data2 report
- N50 must be computed only after removing larger phase blocks that were incorrectly connected per the GIAB trio-phased comparison
- All comparisons must be restricted to GIAB high-confidence autosomal regions with X and Y excluded

## Limitations

- Correctness verification is only possible where a known truth set exists (HG002); patient samples lack trio-phased truth VCFs
- MethPhaser depends on initial SNV phasing accuracy — high SNV phasing errors (e.g., very high hamming distance in R9 80X results) negatively impact the accuracy of both the connections and the verification
- N50 is inflated if incorrectly connected larger phase blocks are not removed before calculation

## Evidence

- [methods] we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB.: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [methods] we can easily know the correctness of MethPhaser’s each SNV phase block connection with our relationship output and further get the correct N50 on the HG002 sample: "we can easily know the correctness of MethPhaser’s each SNV phase block connection with our relationship output and further get the correct N50 on the HG002 sample"
- [discussion] By benchmarking against HG002, which has a known truth set: "By benchmarking against HG002, which has a known truth set"
- [readme] MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes.: "!!! NOTE: MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes. !!!"
