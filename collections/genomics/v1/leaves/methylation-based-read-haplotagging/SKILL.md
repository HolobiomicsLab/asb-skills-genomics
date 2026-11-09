---
name: methylation-based-read-haplotagging
description: Use when ONT reads with MM/ML methylation tags have been SNV-phased (HapCUT2)
  and WhatsHap-haplotagged, but many reads remain untagged and SNV phase blocks need
  to be chained across homozygous gaps.
when_to_use_negative:
- 'Data are from sex chromosomes: random X inactivation in females gives inconsistent
  methylation-haplotype patterns, so MethPhaser is autosomes-only.'
- Coverage is below ~30X, where the underlying SNV phasing methods (HapCUT2, WhatsHap)
  do not work reasonably.
- The BAM contains secondary or supplementary alignments, which trigger pysam MM tag
  reading errors unless filtered first.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: MethPhaser
  role: Core method performing methylation-based read phasing and SNV-phase block
    chaining
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: 'Step 1 driver: computes block relationships via methylation voting (e.g.
    -b/-r/-g/-vc/-o inputs)'
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: 'Step 2 driver: applies block flips and emits the modified VCF and haplotagged
    BAM'
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Produces the input SNV-phased VCF (--ont 1, --outvcf 1) consumed by MethPhaser
- name: WhatsHap
  role: Haplotags reads from HapCUT2 results and generates the phase-block GTF; the
    haplotagged BAM is MethPhaser's input
- name: SAMtools
  role: Filters the BAM to primary alignments only (samtools view -bF 2304) before
    running MethPhaser
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_006
  - task_007
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:48573f370779c7e6280320f95a4903f610eba5522c8983e393a25d0473caa5f6
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
  - meth_phaser_post_processing
  - minimap2
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 28
---

# methylation-based-read-haplotagging

## Summary

Iteratively assign haplotype labels to unphased ONT reads using allele-specific CpG methylation signals, then bridge SNV phase blocks to produce an extended phased VCF and methylation-haplotagged BAM (MethPhaser).

## When to use

Use when ONT reads with MM/ML methylation tags have been SNV-phased (HapCUT2) and WhatsHap-haplotagged, but many reads remain untagged and SNV phase blocks need to be chained across homozygous gaps.

## When NOT to use

- Data are from sex chromosomes: random X inactivation in females gives inconsistent methylation-haplotype patterns, so MethPhaser is autosomes-only.
- Coverage is below ~30X, where the underlying SNV phasing methods (HapCUT2, WhatsHap) do not work reasonably.
- The BAM contains secondary or supplementary alignments, which trigger pysam MM tag reading errors unless filtered first.

## Inputs

- Methylation-annotated haplotagged BAM (MM/ML tags, primary alignments only, e.g. HLA.R10.haplotagged.bam)
- Reference genome FASTA (GCA_000001405.15_GRCh38_no_alt_analysis_set)
- Phase-block GTF from WhatsHap stats/visualization (e.g. LSK.filtered.gtf)
- HapCUT2-phased SNV VCF (e.g. HLA.R10.phased.vcf.gz)
- Optional truth VCF for benchmarking (-vt)

## Outputs

- Methylation-phased VCF with chained phase blocks (output.vcf)
- Updated haplotagged BAM with methylation-derived HP assignments (output)
- Per-chromosome neighbor relationship CSVs recording supporting reads per block connection
- Work directory with per-block haplotag assignment and block-flipping lists

## How to apply

Run meth_phaser_parallel on the primary-alignment-only haplotagged BAM (-b, produced with samtools view -bF 2304), the GRCh38 no-alt reference (-r), the WhatsHap stats GTF (-g), and the HapCUT2-phased VCF (-vc), keeping the default -ml -1 to skip the largest centromeric gap (use -ml -2 if no centromere is present). MethPhaser first identifies allele-specific CpGs per SNV phase block with a Wilcoxon rank sum test (p < 0.05) on base modification probabilities of SNV-haplotagged reads, then iteratively assigns untagged reads by methylation voting (minimum haplotype coverage 3, minimum 3 votes per read) until no more reads can be assigned. Neighbor SNV phase block relationships are then inferred with the best-accuracy filter: voting confidence (votes difference)/total votes > 0.5 and minimum read number = read coverage/(genome-wide block number/1000). Finally run meth_phaser_post_processing (-ib input BAM, -if work dir, -ov output.vcf, -ob output) to apply the block-flipping list and emit the methylation-phased VCF and updated haplotagged BAM.

## Related tools

- **MethPhaser** (Core method performing methylation-based read phasing and SNV-phase block chaining) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Step 1 driver: computes block relationships via methylation voting (e.g. -b/-r/-g/-vc/-o inputs)) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Step 2 driver: applies block flips and emits the modified VCF and haplotagged BAM) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Produces the input SNV-phased VCF (--ont 1, --outvcf 1) consumed by MethPhaser)
- **WhatsHap** (Haplotags reads from HapCUT2 results and generates the phase-block GTF; the haplotagged BAM is MethPhaser's input)
- **SAMtools** (Filters the BAM to primary alignments only (samtools view -bF 2304) before running MethPhaser)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work; ./meth_phaser_post_processing -ib test_data/HLA.R10.haplotagged.bam -if test_data/work/ -ov test_data/output.vcf -ob test_data/output -vc test_data/HLA.R10.phased.vcf.gz
```

## Evaluation signals

- output.vcf exists and encodes the chained phase blocks; the per-chromosome neighbor relationship CSVs in the work directory record the number of supporting reads for each block connection.
- Against the GIAB trio-phased V4.2.1 truth VCF via WhatsHap compare, switch-error increase per established block connection is minimal (e.g. 0.0010%-0.0152% on HG002 R9).
- On HG002 R9 60X autosomes, expected phase length N50 rises from 1,706,719 to 3,997,227 with gaps dropping from 3179 to 1722 and flip rate increasing only 0.02%.
- In the HLA region (chr6:30,400,000-31,300,000) on R10 data, exactly 3 SNV phase blocks are joined with a 132,079 bp extension linking HLA-E and HLA-C.
- Read-assignment CSVs show each read assigned with at least 3 votes and minimum haplotype coverage of 3, consistent with the default filters.

## Limitations

- Cannot improve phasing on sex chromosomes because random X inactivation in females leads to inconsistent haplotype methylation patterns.
- Depends on initial SNV phasing accuracy: high SNV phasing errors (e.g. very high hamming distance in R9 80X results) negatively impact MethPhaser's accuracy.
- Results depend on accuracy of methylation detection, overall read length, and a minimum coverage; the default -ml -1 assumes a centromere-like largest gap is present (use -ml -2 otherwise).
- Haplotype-specific SV phasing potential requires more experimental results; heterogeneous methylation in cancer genomes is not yet handled.

## Evidence

- [methods] MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining.: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [methods] We use the Wilcoxon rank sum test to determine whether the two allele's base modification probabilities are statistically different (p-value < 0.05).: "We use the Wilcoxon rank sum test44,59 to determine whether the two allele’s base modification probabilities are statistically different (p-value < 0.05)."
- [methods] a minimum number of votes (default 3) are required for a read's haplotype assignment.: "a minimum number of votes (default 3) are required for a read’s haplotype assignment."
- [readme] MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes.: "!!! NOTE: MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes. !!!"
- [readme] The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error.: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
