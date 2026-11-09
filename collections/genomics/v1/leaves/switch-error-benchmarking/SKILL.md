---
name: switch-error-benchmarking
description: Use when evaluating whether methylation-based block chaining of SNV phase
  blocks improves phasing (longer blocks, fewer blocks, phased CMRG gene counts) without
  inflating switch/flip error against a known truth set such as GIAB HG002.
when_to_use_negative:
- Benchmarking on sex chromosomes — random X inactivation in females produces inconsistent
  haplotype patterns, so X/Y must be excluded
- Sample lacks a truth set (e.g., no GIAB trio-phased VCF), making relationship assignment
  of phase blocks impossible
- Input BAM contains secondary/supplementary alignments, which trigger pysam MM tag
  reading errors in MethPhaser
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: MethPhaser
  role: Produces the methylation-extended phased VCF and relationship output used
    to verify block connections against the truth VCF
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: Runs methylation-based read phasing and SNV-phase block chaining with best-accuracy
    cutoff parameters
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Generates the modified reads and output VCF used in the comparison against
    SNV-only phasing
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Provides the baseline SNV-only phased VCF against which MethPhaser is benchmarked
- name: WhatsHap
  role: WhatsHap compare quantifies flip and switch errors introduced by block connection;
    haplotag and phase modules supply pipeline inputs
- name: SAMtools
  role: Filters the input BAM to primary alignments (samtools view -bF 2304) to avoid
    pysam MM tag errors
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:7f580e54b36a4cfeb2268d09b3452c8696ebaccc35e8666d78862087ece332a3
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
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# switch-error-benchmarking

## Summary

Benchmarks methylation-extended phasing (MethPhaser) against SNV-only phasing (HapCUT2/WhatsHap) on HG002 by comparing block connections to the GIAB trio-phased truth VCF, removing incorrectly connected blocks before N50-style comparison, and quantifying flip/switch errors with WhatsHap compare.

## When to use

Use when evaluating whether methylation-based block chaining of SNV phase blocks improves phasing (longer blocks, fewer blocks, phased CMRG gene counts) without inflating switch/flip error against a known truth set such as GIAB HG002.

## When NOT to use

- Benchmarking on sex chromosomes — random X inactivation in females produces inconsistent haplotype patterns, so X/Y must be excluded
- Sample lacks a truth set (e.g., no GIAB trio-phased VCF), making relationship assignment of phase blocks impossible
- Input BAM contains secondary/supplementary alignments, which trigger pysam MM tag reading errors in MethPhaser

## Inputs

- MethPhaser output VCF (meth_phaser_parallel + meth_phaser_post_processing with best-accuracy parameters)
- HapCUT2 SNV-only phased VCF for HG002
- GIAB trio-phased truth VCF
- GIAB CMRG challenging medically relevant gene set
- MethPhaser relationship output for block-connection verification

## Outputs

- Per-method counts of phased CMRG genes (e.g., 265/273 = 97.1% MethPhaser vs 258 = 94.51% SNV-only)
- Genome-wide phase block counts per method (140 vs 160)
- N50-style phase-block metrics after removing incorrectly connected blocks
- Flip and switch error rates from WhatsHap compare; new-switch/established-connection rate for Supplementary Data2

## How to apply

Run meth_phaser_parallel with best-accuracy parameters — voting confidence (votes_hap1-votes_hap2)/total_votes > 0.5 and minimum read number = read coverage/(genome-wide block number/1000) — plus meth_phaser_post_processing, alongside a HapCUT2 SNV-only phased VCF for HG002. Assign each SNV phase block's true relationship using the GIAB trio-phased truth VCF, treating neighboring blocks as sharing a haplotype when their hap1/hap2 truth relationships match, and verify MethPhaser's block connections with the relationship output. Remove larger phase blocks that were incorrectly connected before any N50-style comparison, and restrict to autosomes since random X inactivation makes sex chromosomes uninformative. Intersect both phasing results with the GIAB CMRG set to count fully phased genes (expect 265/273, 97.1% for MethPhaser vs 258, 94.51% for SNV-only, with blocks reduced from 160 to 140), and use WhatsHap compare to quantify flip and switch errors introduced by block connection, recording the new-switch/established-connection rate in Supplementary Data2 format.

## Related tools

- **MethPhaser** (Produces the methylation-extended phased VCF and relationship output used to verify block connections against the truth VCF) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Runs methylation-based read phasing and SNV-phase block chaining with best-accuracy cutoff parameters) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Generates the modified reads and output VCF used in the comparison against SNV-only phasing) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Provides the baseline SNV-only phased VCF against which MethPhaser is benchmarked)
- **WhatsHap** (WhatsHap compare quantifies flip and switch errors introduced by block connection; haplotag and phase modules supply pipeline inputs)
- **SAMtools** (Filters the input BAM to primary alignments (samtools view -bF 2304) to avoid pysam MM tag errors)

## Examples

```
./meth_phaser_parallel -b HG002.haplotagged.bam -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fna -vc HG002.hapcut2.phased.vcf.gz -o work/ && whatshap compare --truth HG002.trio.vcf.gz output.vcf
```

## Evaluation signals

- CMRG table reproduces 265/273 (97.1%) genes phased by MethPhaser vs 258 (94.51%) by SNV-only phasing
- Phase block count drops from 160 (SNV-only) to 140 (MethPhaser) on autosomes
- Incorrectly connected larger phase blocks are removed before N50 calculation, verified via the relationship output against the GIAB trio-phased VCF
- Flip/switch error figures from WhatsHap compare are consistent with Supplementary Data2
- All benchmarked blocks restricted to autosomes with X and Y chromosomes excluded

## Limitations

- Cannot improve phasing for human sex chromosomes in clinical samples due to random X inactivation in females
- MethPhaser depends on initial SNV phasing accuracy; high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact its accuracy
- Results depend on methylation detection accuracy, overall read length, and minimum data coverage; the README recommends at least 30X ONT coverage

## Evidence

- [methods] Incorrectly connected blocks removed before N50: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [discussion] Exclude X and Y from benchmark: "Thus, we exclude the X and Y chromosomes from our benchmark"
- [discussion] Minimal switch error from block joining: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [methods] Switch/flip reporting in SI: "We report this number in Supplementary Data2."
- [readme] Autosomes only (README): "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
