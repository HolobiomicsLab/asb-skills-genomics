---
name: snv-phase-block-chaining
description: Use when ONT reads with MM/ML methylation tags have already been SNV-phased
  (HapCUT2/WhatsHap) into fragmented phase blocks and you need to bridge adjacent
  blocks using allele-specific methylation.
when_to_use_negative:
- Data include sex chromosomes — MethPhaser cannot improve phasing there because random
  X inactivation in females yields inconsistent haplotype methylation patterns; restrict
  to autosomes
- Coverage is well below 30X or reads are short — SNV phasing methods (HapCUT2, WhatsHap)
  and methylation voting need at least ~30X ONT coverage and adequate read length
- Input BAM contains secondary or supplementary alignments, which trigger pysam MM
  tag reading errors instead of being usable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MethPhaser
  role: 'Core method: performs methylation-based read phasing and SNV-phase block
    chaining from WhatsHap haplotagged reads and HapCUT2 phased SNVs'
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: 'Runs the parallel block-relationship inference: haplotags unphased reads
    by methylation voting and infers neighbor SNV phase block relationships'
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Applies the block-flipping list to emit the modified phased VCF and the BAM
    with merged haplotype assignments
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Produces the initial SNV-phased VCF that supplies the phase blocks to be chained
- name: WhatsHap
  role: Haplotags reads from HapCUT2 phasing and generates the GTF of SNV phase blocks
    used as input
- name: samtools
  role: Filters the input BAM to primary alignments only (view -bF 2304) before MethPhaser
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:062255fdd3389ddfde8c1b9b007d4a3a9e42ceef7d6ae4c9846c03f4d3574a24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - Clair3
  - HapCUT2
  - MethPhaser
  - WhatsHap
  - WhatsHap compare
  - meth_phaser_parallel
  - meth_phaser_post_processing
  - minimap2
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# snv-phase-block-chaining

## Summary

Chains disconnected SNV phase blocks into larger blocks by leveraging haplotype-specific CpG methylation signals from ONT reads, using majority voting over methylation-haplotagged reads to infer neighbor block relationships and merge them.

## When to use

Use when ONT reads with MM/ML methylation tags have already been SNV-phased (HapCUT2/WhatsHap) into fragmented phase blocks and you need to bridge adjacent blocks using allele-specific methylation.

## When NOT to use

- Data include sex chromosomes — MethPhaser cannot improve phasing there because random X inactivation in females yields inconsistent haplotype methylation patterns; restrict to autosomes
- Coverage is well below 30X or reads are short — SNV phasing methods (HapCUT2, WhatsHap) and methylation voting need at least ~30X ONT coverage and adequate read length
- Input BAM contains secondary or supplementary alignments, which trigger pysam MM tag reading errors instead of being usable

## Inputs

- Methylation-annotated, WhatsHap-haplotagged BAM restricted to primary alignments (samtools view -bF 2304)
- Reference genome FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set, chromosome-restricted)
- WhatsHap GTF file describing SNV phase blocks
- HapCUT2-phased VCF of called SNVs
- Optional truth VCF for benchmarking (-vt)

## Outputs

- Per-chromosome neighbor relationship CSVs in the work directory recording supporting reads per block connection
- Modified phased VCF (output.vcf) with chained/merged phase blocks
- Re-haplotagged BAM with merged methylation-based haplotype assignments (HP tags)

## How to apply

Restrict the methylation-annotated BAM to primary alignments (samtools view -bF 2304) and run meth_phaser_parallel with -b (haplotagged BAM), -r (reference FASTA, e.g. GRCh38 no-alt chr6), -g (WhatsHap GTF of SNV phase blocks), -vc (HapCUT2 phased VCF), and -o work directory, keeping default -ml -1 to skip the largest homozygous/centromeric region. MethPhaser first identifies allele-specific CpGs per SNV phase block with a Wilcoxon rank sum test (p-value < 0.05) on base modification probabilities of SNV-haplotagged reads, then iteratively haplotags unphased reads by methylation voting (minimum 3 votes, minimum haplotype coverage 3) until convergence or the iteration limit. It infers neighbor SNV phase block relationships from reads spanning consecutive extended block boundaries, applying the best-accuracy filter: voting confidence (votes for one haplotype minus votes for the other)/total votes > 0.5 and a minimum read number of read coverage/(genome-wide block number/1000). Finally run meth_phaser_post_processing (-ib input BAM, -if work dir, -ov output.vcf, -ob output BAM, -vc phased VCF) to apply the block-flipping list and emit the merged-haplotype VCF and re-haplotagged BAM. On HG002 HLA-region data this connects 3 SNV blocks with a 132,079 bp extension linking HLA-E and HLA-C.

## Related tools

- **MethPhaser** (Core method: performs methylation-based read phasing and SNV-phase block chaining from WhatsHap haplotagged reads and HapCUT2 phased SNVs) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Runs the parallel block-relationship inference: haplotags unphased reads by methylation voting and infers neighbor SNV phase block relationships) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Applies the block-flipping list to emit the modified phased VCF and the BAM with merged haplotype assignments) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Produces the initial SNV-phased VCF that supplies the phase blocks to be chained)
- **WhatsHap** (Haplotags reads from HapCUT2 phasing and generates the GTF of SNV phase blocks used as input)
- **samtools** (Filters the input BAM to primary alignments only (view -bF 2304) before MethPhaser)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- On HG002 R10 HLA-region data, exactly 3 SNV phase blocks are connected into one block spanning chr6:30,400,000-31,300,000 with a 132,079 bp extension linking HLA-E and HLA-C
- The neighbor relationship CSVs in the work directory record the number of supporting reads for each block connection and confirm correctness of each connection
- output.vcf exists and encodes the chained phase blocks after the block-flipping list is applied by meth_phaser_post_processing
- Every accepted block connection passes the default filter: voting confidence (vote difference)/total votes > 0.5 and minimum read support of read coverage/(genome-wide block number/1000)
- Benchmarked against the GIAB trio-phased HG002 truth set, incorrectly connected larger phase blocks are removed before N50 calculation; expected N50 improvements fall in the reported 1.69-2.55x range for HPRC samples

## Limitations

- Accuracy depends on initial SNV phasing quality; high SNV phasing errors (e.g. very high hamming distance in R9 80x results) negatively impact MethPhaser accuracy
- Results depend on accuracy of methylation detection, overall read length, and a minimum data coverage
- Does not work on sex chromosomes due to random X inactivation in females
- If input data lack a large poorly mapped region (no centromere), -ml must be set to -2; the default -ml -1 ignores the largest homozygous region
- Phasing of haplotype-specific SVs is promising but requires more experimental validation

## Evidence

- [methods] MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining.: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [methods] We use the Wilcoxon rank sum test to determine whether the two allele's base modification probabilities are statistically different.: "We use the Wilcoxon rank sum test44,59 to determine whether the two allele’s base modification probabilities are statistically different (p-value < 0.05)."
- [methods] Best accuracy, with read coverage/(genome-wide block number/1000) as a minimum read number supporting block relationship assignment requirement.: "Best accuracy, with read coverage/(genome-wide block number/1000) as a minimum read number supporting block relationship assignment requirement"
- [discussion] MethPhaser relies on a majority vote to make decisions about combining existing phase blocks.: "MethPhaser relies on a majority vote to make decisions about combining existing phase blocks"
- [readme] MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes.: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
- [readme] The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error.: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
