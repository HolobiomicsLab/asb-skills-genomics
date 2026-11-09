---
name: haplotype-phase-block-truth-comparison
description: Use when benchmarking a phasing method's block-chaining decisions against
  a trio-phased truth VCF (e.g. GIAB HG002) to report phased medically relevant gene
  counts and N50-style block statistics.
when_to_use_negative:
- The analysis includes sex chromosomes — MethPhaser block connections are invalid
  there because random X inactivation in females leads to inconsistent haplotype patterns
- No trio-phased truth VCF is available, so block-connection correctness cannot be
  independently verified
- The sample lacks ONT methylation signal or coverage below the recommended 30X, since
  SNV phasing methods only work reasonably above that coverage
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MethPhaser
  role: Produces the methylation-extended phased VCF and relationship output used
    to connect SNV phase blocks
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: Runs the methylation-based read phasing and block-relationship voting step
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Generates the final modified reads and phased VCF output for comparison
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Provides the SNV-only baseline phased VCF for comparison
- name: WhatsHap
  role: Haplotags reads and quantifies flip/switch errors via WhatsHap compare
- name: SAMtools
  role: Filters BAM to primary alignments (samtools view -bF 2304) before phasing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:ba01021e1b23a3aee16440ff6e18170c4f7c89433cd07d307701147b55f18a52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - HapCUT2
  - MethPhaser
  - SAMtools
  - WhatsHap
  - meth_phaser_parallel
  - meth_phaser_post_processing
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# haplotype-phase-block-truth-comparison

## Summary

Compares methylation-extended phasing (MethPhaser) against SNV-only phasing by validating each SNV phase-block connection against the GIAB trio-phased truth VCF, then quantifying phased CMRG gene counts, phase-block counts, and flip/switch errors on HG002.

## When to use

Use when benchmarking a phasing method's block-chaining decisions against a trio-phased truth VCF (e.g. GIAB HG002) to report phased medically relevant gene counts and N50-style block statistics.

## When NOT to use

- The analysis includes sex chromosomes — MethPhaser block connections are invalid there because random X inactivation in females leads to inconsistent haplotype patterns
- No trio-phased truth VCF is available, so block-connection correctness cannot be independently verified
- The sample lacks ONT methylation signal or coverage below the recommended 30X, since SNV phasing methods only work reasonably above that coverage

## Inputs

- MethPhaser output VCF from meth_phaser_parallel with default best-accuracy parameters
- HapCUT2 SNV-only phased VCF for HG002
- GIAB trio-phased truth VCF
- GIAB CMRG (challenging medically relevant genes) gene set
- MethPhaser relationship output for block-connection verification

## Outputs

- Per-method counts of CMRG genes fully contained in phase blocks (e.g. 265/273 for MethPhaser vs 258 for SNV-only)
- Genome-wide phase block counts per method (140 vs 160)
- Flip/switch error metrics from WhatsHap compare and new-switch/established-connection rates for Supplementary Data2

## How to apply

Run meth_phaser_parallel with best-accuracy parameters (voting confidence (votes_hap1-votes_hap2)/total_votes > 0.5; minimum read number = read coverage/(genome-wide block number/1000)) and obtain the HapCUT2 SNV-only phased VCF for HG002. Assign the true relationship of each SNV phase block to the GIAB trio-phased truth VCF, treating neighboring blocks with matching hap1/hap2 truth relationships as the same haplotype assignment. Verify MethPhaser's block connections with the relationship output, remove incorrectly connected larger phase blocks before any N50-style comparison, and restrict to autosomes because of random X inactivation. Intersect both phasing results with the GIAB CMRG set to count fully phased genes per method, and quantify flip/switch errors introduced by block connection with WhatsHap compare, recording the new-switch/established-connection rate in Supplementary Data2.

## Related tools

- **MethPhaser** (Produces the methylation-extended phased VCF and relationship output used to connect SNV phase blocks) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Runs the methylation-based read phasing and block-relationship voting step) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Generates the final modified reads and phased VCF output for comparison) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Provides the SNV-only baseline phased VCF for comparison)
- **WhatsHap** (Haplotags reads and quantifies flip/switch errors via WhatsHap compare)
- **SAMtools** (Filters BAM to primary alignments (samtools view -bF 2304) before phasing)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- Reproduce 265/273 (97.1%) CMRGs phased by MethPhaser vs 258 (94.51%) by SNV-only phasing on HG002
- Phase block count drops from 160 (SNV-only) to 140 (MethPhaser) after connection
- Incorrectly connected larger phase blocks are removed before N50-style comparison; remaining N50 must be consistent with the cleaned blocks
- Flip/switch error figures from WhatsHap compare match Supplementary Data2, with only a minimal increase in switch error from joining blocks
- Relationship output confirms each MethPhaser block connection matches the GIAB trio-phased truth VCF hap1/hap2 assignment

## Limitations

- MethPhaser cannot improve phasing on sex chromosomes because of random X inactivation in females; autosomes only
- Depends on initial SNV phasing accuracy — high SNV phasing errors (e.g. high hamming distance in R9 80x results) negatively impact MethPhaser accuracy
- Results depend on methylation detection accuracy, overall read length, and minimum data coverage; patient samples with shorter reads (12–15 kbp N50) yield smaller initial phase blocks and limited improvement

## Evidence

- [methods] Remove incorrect blocks before N50: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [methods] Relationship output verifies connections: "we can easily know the correctness of MethPhaser’s each SNV phase block connection with our relationship output and further get the correct N50 on the HG002 sample"
- [readme] Restrict to autosomes: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
- [abstract] Medically relevant genes improved: "MethPhaser further improves phasing across HLA and multiple other medically relevant genes"
- [methods] High confidence region filter: "limited our interested region to “high confidence” regions provided by GIAB to remove potential Clair3’s false positive calls."
