---
name: cmrg-gene-phasing-coverage-quantification
description: Use when comparing phasing methods on HG002 with a GIAB trio-phased truth
  VCF and you need per-gene CMRG phased-gene counts, genome-wide phase block counts,
  and flip/switch error rates from block connections.
when_to_use_negative:
- 'Benchmarking involves sex chromosomes: MethPhaser phasing is invalid there because
  of random X inactivation in females, so X/Y must be excluded before counting'
- Input data is below ~30X ONT coverage, where SNV phasing methods (HapCUT2, WhatsHap)
  that MethPhaser builds on do not work reasonably
- The BAM contains secondary or supplementary alignments, which trigger pysam MM tag
  reading errors in MethPhaser
edam_operation: http://edamontology.org/operation_3199
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: MethPhaser
  role: Methylation-based read phasing and SNV-phase block chaining that extends HapCUT2
    SNV phasing on autosomes
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: Runs MethPhaser to obtain block relationships using methylation signals from
    the haplotagged BAM
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Produces the final modified phased VCF from the parallel-run work directory
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Provides the SNV-only phased VCF baseline against which CMRG gene coverage
    is compared
- name: WhatsHap
  role: Haplotags reads and compares phasing results (flip/switch errors) via WhatsHap
    compare
- name: SAMtools
  role: Filters the input BAM to primary alignments only (samtools view -bF 2304)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:2929f01e6342cc97d2eb63f3e1455dd5236c6661bdd82c925470658fd45223ff
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
  edam_operation: http://edamontology.org/operation_3199
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cmrg-gene-phasing-coverage-quantification

## Summary

Benchmark how many GIAB CMRG challenging medically relevant genes are fully phased by methylation-extended phasing (MethPhaser) versus SNV-only phasing (HapCUT2) on HG002, and quantify the resulting reduction in phase block number (160 to 140).

## When to use

Use when comparing phasing methods on HG002 with a GIAB trio-phased truth VCF and you need per-gene CMRG phased-gene counts, genome-wide phase block counts, and flip/switch error rates from block connections.

## When NOT to use

- Benchmarking involves sex chromosomes: MethPhaser phasing is invalid there because of random X inactivation in females, so X/Y must be excluded before counting
- Input data is below ~30X ONT coverage, where SNV phasing methods (HapCUT2, WhatsHap) that MethPhaser builds on do not work reasonably
- The BAM contains secondary or supplementary alignments, which trigger pysam MM tag reading errors in MethPhaser

## Inputs

- MethPhaser output VCF from meth_phaser_parallel with default best-accuracy parameters
- HapCUT2 SNV-only phased VCF for HG002
- GIAB trio-phased truth VCF for HG002
- GIAB CMRG challenging medically relevant genes BED/gene set
- MethPhaser relationship output for block-connection verification

## Outputs

- Per-method count of CMRG genes fully contained in phase blocks (265/273 vs 258/273)
- Genome-wide phase block counts per method (140 vs 160)
- Flip and switch error rates from WhatsHap compare on HG002
- New-switch/established-connection rate for Supplementary Data2

## How to apply

Run meth_phaser_parallel on the WhatsHap-haplotagged ONT BAM with best-accuracy parameters (voting confidence (votes_hap1-votes_hap2)/total_votes > 0.5 and minimum read number = read coverage/(genome-wide block number/1000)), then meth_phaser_post_processing to obtain the modified VCF. Assign the true relationship of each SNV phase block against the GIAB trio-phased truth VCF, treating neighboring blocks as the same haplotype when their hap1/hap2 truth relationships match, and verify MethPhaser's block connections with the relationship output before removing incorrectly connected larger phase blocks prior to any N50-style comparison. Restrict all counting to autosomes (sex chromosomes excluded because of random X inactivation), then intersect both the MethPhaser and SNV-only phasing results with the GIAB CMRG gene set to count genes fully contained in phase blocks. Finally, run WhatsHap compare on HG002 to quantify flip and switch errors introduced by block connection and record the new-switch/established-connection rate in Supplementary Data2. Correct application reproduces 265/273 (97.1%) CMRGs phased by MethPhaser versus 258 (94.51%) by SNV-only phasing, with phase blocks reduced from 160 to 140.

## Related tools

- **MethPhaser** (Methylation-based read phasing and SNV-phase block chaining that extends HapCUT2 SNV phasing on autosomes) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Runs MethPhaser to obtain block relationships using methylation signals from the haplotagged BAM) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Produces the final modified phased VCF from the parallel-run work directory) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Provides the SNV-only phased VCF baseline against which CMRG gene coverage is compared)
- **WhatsHap** (Haplotags reads and compares phasing results (flip/switch errors) via WhatsHap compare)
- **SAMtools** (Filters the input BAM to primary alignments only (samtools view -bF 2304))

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- MethPhaser phases 265 of 273 CMRGs (97.1%) on HG002 versus 258 (94.51%) for SNV-only phasing
- Genome-wide phase block count drops from 160 (SNV-only) to 140 (MethPhaser)
- Incorrectly connected larger phase blocks are identified via the relationship output against the GIAB trio-phased VCF and removed before N50-style comparison
- Flip and switch errors introduced by block connection are consistent with Supplementary Data2, showing only a minimal increase in switch error
- All gene counting is restricted to autosomes; X and Y chromosomes are excluded from the benchmark

## Limitations

- MethPhaser cannot improve phasing on sex chromosomes because random deactivation of chromosome X in females leads to inconsistent haplotype patterns
- Results depend on accuracy of methylation detection, overall read length, and a minimum data coverage; initial SNV phasing errors (e.g., high hamming distance in R9 80x data) negatively impact MethPhaser accuracy
- Improvement in patient samples is limited by initially smaller phase blocks from shorter reads (12–15 kbp N50); thresholds used to uphold accuracy are not fully detailed

## Evidence

- [abstract] MethPhaser further improves phasing across HLA and multiple other medically relevant genes: "MethPhaser further improves phasing across HLA and multiple other medically relevant genes"
- [methods] Removal of incorrect blocks before N50: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [methods] Correct N50 via relationship output on HG002: "we can easily know the correctness of MethPhaser’s each SNV phase block connection with our relationship output and further get the correct N50 on the HG002 sample"
- [readme] Autosome-only restriction: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
- [discussion] Minimal switch error from joining blocks: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
