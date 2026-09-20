---
name: bam-vcf-haplotag-post-processing
description: Use when meth_phaser_parallel has finished writing block-relationship
  outputs into a work directory and the re-phased VCF and haplotag-updated BAM are
  needed to complete the phasing workflow.
when_to_use_negative:
- The input BAM still contains secondary or supplementary alignments — these must
  be removed first, otherwise a pysam MM tag reading error is triggered
- The phasing target is a sex chromosome — random X inactivation in females produces
  inconsistent haplotype methylation patterns, so MethPhaser (including its post-processing)
  will not work
- The meth_phaser_parallel work directory has not been produced yet — post-processing
  is strictly downstream of the block-relationship step
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: meth_phaser_post_processing
  role: post-processing executable that consumes the work directory and emits the
    flipped/re-phased VCF and modified haplotagged BAM
  repo: https://github.com/treangenlab/methphaser
- name: MethPhaser
  role: the overall method whose methylation-based read phasing and SNV phase-block
    chaining results are finalized by post-processing
  repo: https://github.com/treangenlab/methphaser
- name: samtools
  role: pre-filters the BAM to primary alignments only (samtools view -bF 2304) so
    post-processing can read MM tags without pysam errors
- name: pysam
  role: library used internally to read MM/ML methylation tags from the BAM during
    post-processing
- name: HapCUT2
  role: produces the input SNV-phased VCF (-vc) that post-processing re-phases into
    the extended blocks
- name: WhatsHap
  role: its haplotag module produced the initial SNV-haplotagged BAM that post-processing
    updates with methylation-based tags
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:2baa3eeab525fa28b406986cea9ad1311a0f5a18c0f31cde91b676ed9aed4c5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - HapCUT2
  - MethPhaser
  - WhatsHap
  - meth_phaser_parallel
  - meth_phaser_post_processing
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# bam-vcf-haplotag-post-processing

## Summary

MethPhaser's post-processing step that consumes the methylation-based block-relationship work directory and emits the final re-phased VCF plus a re-haplotagged BAM in which reads carry methylation-informed haplotype tags.

## When to use

Use when meth_phaser_parallel has finished writing block-relationship outputs into a work directory and the re-phased VCF and haplotag-updated BAM are needed to complete the phasing workflow.

## When NOT to use

- The input BAM still contains secondary or supplementary alignments — these must be removed first, otherwise a pysam MM tag reading error is triggered
- The phasing target is a sex chromosome — random X inactivation in females produces inconsistent haplotype methylation patterns, so MethPhaser (including its post-processing) will not work
- The meth_phaser_parallel work directory has not been produced yet — post-processing is strictly downstream of the block-relationship step

## Inputs

- haplotagged BAM with MM/ML methylation tags, filtered to primary alignments (samtools view -bF 2304)
- meth_phaser_parallel work directory containing block-relationship outputs
- HapCUT2-phased SNV VCF (input phased VCF, e.g. gzipped)

## Outputs

- re-phased VCF encoding chained/extended phase blocks (output.vcf)
- modified (re-haplotagged) BAM with methylation-informed haplotype tags
- neighbor relationship CSVs in the work directory recording supporting-read counts per block connection

## How to apply

After meth_phaser_parallel completes its three-step loop (Wilcoxon rank-sum identification of allele-specific CpGs, iterative methylation-based read haplotagging, and SNV phase-block chaining), run meth_phaser_post_processing with the original haplotagged BAM (-ib), the work directory (-if), the desired output VCF path (-ov), the output BAM prefix (-ob), and the input HapCUT2-phased VCF (-vc). The script flips or re-phases SNVs according to the chained phase blocks and applies the new methylation-derived haplotype tags to reads, producing output.vcf and an output BAM. The input BAM must contain only primary alignments (prepared with samtools view -bF 2304), since secondary/supplementary alignments otherwise trigger pysam MM tag reading errors. Correctness is confirmed when output.vcf exists, encodes the chained (larger) phase blocks, and the work-directory neighbor-relationship CSVs record supporting-read counts for each block connection.

## Related tools

- **meth_phaser_post_processing** (post-processing executable that consumes the work directory and emits the flipped/re-phased VCF and modified haplotagged BAM) — https://github.com/treangenlab/methphaser
- **MethPhaser** (the overall method whose methylation-based read phasing and SNV phase-block chaining results are finalized by post-processing) — https://github.com/treangenlab/methphaser
- **samtools** (pre-filters the BAM to primary alignments only (samtools view -bF 2304) so post-processing can read MM tags without pysam errors)
- **pysam** (library used internally to read MM/ML methylation tags from the BAM during post-processing)
- **HapCUT2** (produces the input SNV-phased VCF (-vc) that post-processing re-phases into the extended blocks)
- **WhatsHap** (its haplotag module produced the initial SNV-haplotagged BAM that post-processing updates with methylation-based tags)

## Examples

```
./meth_phaser_post_processing -ib test_data/HLA.R10.haplotagged.bam -if test_data/work/ -ov test_data/output.vcf -ob test_data/output -vc test_data/HLA.R10.phased.vcf.gz
```

## Evaluation signals

- output.vcf exists and encodes the chained (extended) phase blocks relative to the input HapCUT2-phased VCF
- the output BAM carries updated haplotype tags consistent with the methylation voting results
- neighbor relationship CSVs in the work directory record the number of supporting reads for each phase-block connection
- input BAM was pre-filtered with samtools view -bF 2304 (only primary alignments) to avoid pysam MM tag reading errors
- on benchmark samples (e.g., HG002 against the GIAB trio-phased truth set), the re-phased N50 should be longer than the SNV-only phasing result with only minimal increase in switch error

## Limitations

- MethPhaser (and thus its post-processing) will not work on sex chromosomes due to random X inactivation in females
- recommend at least 30X ONT coverage since the upstream SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above that coverage
- accuracy depends on the initial SNV phasing quality; high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact results
- results depend on methylation detection accuracy, overall read length, and a certain minimum dataset coverage
- the -ml flag must be set carefully: default -1 ignores the largest homozygous/centromeric region, and -2 skips nothing

## Evidence

- [methods] Post-processing consumes the work directory and emits flipped/re-phased VCF and modified BAM: "./meth_phaser_post_processing -ib test_data/HLA.R10.haplotagged.bam -if test_data/work/ -ov test_data/output.vcf"
- [readme] Primary-alignment filtering is required before pysam MM tag reading: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error. Suggested command:
`samtools view -bF"
- [methods] MethPhaser input/output chain from haplotagged reads and phased SNVs to read phasing and block chaining: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [readme] Sex chromosome limitation stated explicitly in README: "NOTE: MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
- [discussion] Joined phase blocks incur only minimal switch error, validating the post-processed result: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
