---
name: ont-read-coverage-subsampling
description: Use when benchmarking long-read phasing performance across sequencing
  depths, e.g., downsampling HG002 R9 ONT reads to 30X/60X/80X to test how methylation-based
  phasing N50 gains scale with coverage.
when_to_use_negative:
- Phasing human sex chromosomes — MethPhaser does not work there due to random X inactivation,
  so the sweep must be limited to autosomes
- Data below ~30X coverage — most SNV phasing methods (HapCUT2, WhatsHap) only work
  reasonably above that coverage, so the SNV baseline is unreliable
- Samples with very high SNV phasing errors (e.g., very high hamming distance in R9
  80X results), where subsampled-depth comparison would negatively impact MethPhaser
  accuracy and confound the depth-effect estimate
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns subsampled ONT reads to GRCh38 while preserving MM and ML methylation
    tags for downstream methylation-based phasing
- name: SAMtools
  role: Filters subsampled BAMs to primary alignments (samtools view -bF 2304) to
    avoid pysam MM tag reading errors
- name: Clair3
  role: Calls variants on each coverage level (v0.1-r12) prior to SNV phasing, with
    AF > 0.25 filtering
- name: HapCUT2
  role: Produces the SNV-only baseline phasing (extractHAIRS --ont 1; hapcut2 --outvcf
    1) at each coverage level
- name: WhatsHap
  role: Haplotags reads from HapCUT2 output and compares phased VCFs to compute switch
    error against the truth
- name: MethPhaser
  role: Performs methylation-based read phasing and SNV-phase block chaining on each
    subsampled coverage level (meth_phaser_parallel + meth_phaser_post_processing)
  repo: github.com/treangenlab/methphaser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:785370ea2e7e887fcb9d17bdaad221b4a43cf4ceacc55a4f38f2d75b810da877
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - HapCUT2
  - MethPhaser
  - SAMtools
  - WhatsHap
  - minimap2
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ONT read coverage subsampling

## Summary

Subsampling deep ONT reads to controlled coverage levels (e.g., 30X, 60X, 80X) while preserving methylation tags, so that coverage-dependent effects on methylation-based phasing (phase block N50, switch error) can be quantified.

## When to use

Use when benchmarking long-read phasing performance across sequencing depths, e.g., downsampling HG002 R9 ONT reads to 30X/60X/80X to test how methylation-based phasing N50 gains scale with coverage.

## When NOT to use

- Phasing human sex chromosomes — MethPhaser does not work there due to random X inactivation, so the sweep must be limited to autosomes
- Data below ~30X coverage — most SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above that coverage, so the SNV baseline is unreliable
- Samples with very high SNV phasing errors (e.g., very high hamming distance in R9 80X results), where subsampled-depth comparison would negatively impact MethPhaser accuracy and confound the depth-effect estimate

## Inputs

- Deep-coverage ONT R9 FASTQ/BAM reads from HG002 with methylation (MM/ML) tags
- GRCh38 no-alt analysis set reference genome (GCA_000001405.15_GRCh38_no_alt_analysis_set)
- GIAB trio-phased truth VCF for HG002
- GIAB high confidence region BED for autosomes

## Outputs

- Subsampled, primary-alignment-only BAMs at each target coverage level (30X/60X/80X) retaining MM/ML tags
- Per-coverage phased VCFs from HapCUT2 (SNV-only baseline) and MethPhaser
- Coverage-sweep benchmark table (e.g., methphaser_hg002_r9_benchmark.csv) with phase block N50, N50 fold improvement, and switch error per coverage

## How to apply

Subsample the HG002 ONT R9 dataset to target depths (30X, 60X, 80X) and align each to the GRCh38 (GCA_000001405.15 no-alt analysis set) reference with minimap2 v2.24 keeping the MM and ML methylation tags. Filter each BAM to primary alignments only with `samtools view -bF 2304`, since secondary/supplementary alignments trigger pysam MM tag reading errors in MethPhaser. At each coverage level, call variants with Clair3 v0.1-r12 (filter AF > 0.25, restrict to GIAB high confidence autosomal regions), phase with HapCUT2 (extractHAIRS --ont 1), haplotag reads with WhatsHap, and run MethPhaser (meth_phaser_parallel then meth_phaser_post_processing) with best-accuracy parameters (minimum read number = coverage/(genome-wide block number/1000), voting confidence > 0.5). The rationale is that phasing performance is 'more driven by read length and sequencing coverage than other factors', so a controlled coverage sweep isolates depth as the variable. Report phase block N50 (after removing incorrectly connected blocks vs. the GIAB trio-phased truth VCF) and switch error (WhatsHap compare) per coverage level, then compare against the SNV-only baseline.

## Related tools

- **minimap2** (Aligns subsampled ONT reads to GRCh38 while preserving MM and ML methylation tags for downstream methylation-based phasing)
- **SAMtools** (Filters subsampled BAMs to primary alignments (samtools view -bF 2304) to avoid pysam MM tag reading errors)
- **Clair3** (Calls variants on each coverage level (v0.1-r12) prior to SNV phasing, with AF > 0.25 filtering)
- **HapCUT2** (Produces the SNV-only baseline phasing (extractHAIRS --ont 1; hapcut2 --outvcf 1) at each coverage level)
- **WhatsHap** (Haplotags reads from HapCUT2 output and compares phased VCFs to compute switch error against the truth)
- **MethPhaser** (Performs methylation-based read phasing and SNV-phase block chaining on each subsampled coverage level (meth_phaser_parallel + meth_phaser_post_processing)) — github.com/treangenlab/methphaser

## Examples

```
samtools view -bF 2304 -o output.bam input.bam
```

## Evaluation signals

- Subsampled BAM coverage matches target depths (30X/60X/80X) and all reads are primary alignments (no secondary/supplementary after samtools view -bF 2304)
- N50 fold improvements over SNV-only phasing fall at approximately 1.78x (30X), 2.34x (60X), and 2.53x (80X) on HG002 R9
- Switch error stays within the reported 0.0010%-0.0152% range, matching Supplementary Data2
- N50 is computed as the minimum block length at which blocks of that and greater size span >= 50% of total phased length, after removing incorrectly connected larger blocks relative to the GIAB trio-phased truth VCF
- Results reproduce monotonically increasing N50 improvement with coverage, consistent with performance being driven by read length and sequencing coverage

## Limitations

- Sex chromosomes must be excluded: random deactivation of chromosome X in females leads to inconsistent haplotype patterns, so the sweep covers autosomes only
- MethPhaser depends on initial SNV phasing accuracy; if SNV phasing methods produce high errors (e.g., very high hamming distance in R9 80X results) it negatively impacts accuracy
- Phasing improvement can be limited by initially smaller phase blocks from shorter reads (12-15 kbp N50 in patient samples); performance depends on methylation detection accuracy, read length, and a minimum data coverage

## Evidence

- [readme] We recommend using MethPhaser for at least 30X coverage ONT reads since most of the SNV phasing methods (HapCUT2, Whatshap) only work reasonably above that coverage: "We recommend using MethPhaser for at least 30X coverage ONT reads since most of the SNV phasing methods (HapCUT2, Whatshap) only work reasonably above that coverage"
- [methods] minimap2 alignment preserving methylation tags: "We then used minimap2 v2.2462 to align reads to the reference and keep the MM and ML tags."
- [discussion] coverage drives performance: "its performance is more driven by read length and sequencing coverage than other factors"
- [methods] WhatsHap haplotag for baseline: "The WhatsHap haplotag module was applied to tag reads based on HapCUT2’s SNV phasing results."
- [readme] primary alignment filtering: "samtools view -bF 2304 -o output.bam input.bam"
