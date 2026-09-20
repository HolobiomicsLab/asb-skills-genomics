---
name: genomic-coordinate-interval-analysis
description: Use when a phased ONT BAM/VCF needs phase blocks joined within a specific
  coordinate window (e.g., HLA chr6:30,400,000-31,300,000) and SNV-only phasing left
  genes like HLA-E and HLA-C in separate blocks.
when_to_use_negative:
- 'The analysis targets human sex chromosomes (X or Y): random X inactivation in females
  creates inconsistent haplotype methylation patterns, so MethPhaser will not work'
- 'Coverage is below ~30X ONT reads: most SNV phasing methods (HapCUT2, WhatsHap)
  only work reasonably above that coverage, and MethPhaser depends on the accuracy
  of the initial SNV phasing'
- The input BAM still contains secondary or supplementary alignments, which will trigger
  a pysam MM tag reading error
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MethPhaser
  role: Performs methylation-based read phasing and SNV-phase block chaining across
    the chr6 HLA coordinate interval
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: First-step driver that methylation-haplotags unphased reads and infers neighbor
    SNV phase block relationships over the interval
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Second-step script that applies the block-flipping list and emits the modified
    VCF and haplotagged BAM
  repo: https://github.com/treangenlab/methphaser
- name: samtools
  role: Restricts the haplotagged BAM to primary alignments (view -bF 2304) to avoid
    pysam MM tag errors
- name: WhatsHap
  role: Provides the haplotagged BAM and visualization GTF used as interval inputs
    to MethPhaser
- name: HapCUT2
  role: Produces the SNV-phased VCF that MethPhaser extends via block chaining
- name: pysam
  role: Used internally to read MM/ML methylation tags from the haplotagged BAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:4d54481350a8381811afce8ebdca84668e3e60722f60f339e6cf1f9d84166244
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# genomic-coordinate-interval-analysis

## Summary

Restricts methylation-haplotagged long-read alignments to a defined genomic coordinate interval (e.g., the HLA Class I region chr6:30,400,000-31,300,000 on GRCh38 chr6) and uses MethPhaser's methylation-based block chaining to merge SNV phase blocks across that interval, extending phase blocks by ~132,079 bp to link HLA-E and HLA-C.

## When to use

Use when a phased ONT BAM/VCF needs phase blocks joined within a specific coordinate window (e.g., HLA chr6:30,400,000-31,300,000) and SNV-only phasing left genes like HLA-E and HLA-C in separate blocks.

## When NOT to use

- The analysis targets human sex chromosomes (X or Y): random X inactivation in females creates inconsistent haplotype methylation patterns, so MethPhaser will not work
- Coverage is below ~30X ONT reads: most SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above that coverage, and MethPhaser depends on the accuracy of the initial SNV phasing
- The input BAM still contains secondary or supplementary alignments, which will trigger a pysam MM tag reading error

## Inputs

- WhatsHap haplotagged BAM restricted to primary alignments (samtools view -bF 2304), containing MM/ML methylation tags
- GRCh38 chr6 reference FASTA (GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna)
- Filtered GTF from WhatsHap visualization
- HapCUT2-phased SNV VCF (gzipped)
- Coordinate interval definition (chr6:30,400,000-31,300,000, HLA Class I region)

## Outputs

- Per-chromosome neighbor SNV phase block relationship CSVs (block connections with voting confidence and read support)
- Modified phased VCF (output.vcf) with merged haplotype assignments and block flips applied
- Post-processed BAM with methylation-based haplotype tags on previously unphased reads
- Extended HLA phase block statistics (3 SNV blocks connected, 132,079 bp extension linking HLA-E and HLA-C)

## How to apply

Restrict the WhatsHap haplotagged BAM to primary alignments with `samtools view -bF 2304`, since secondary/supplementary alignments trigger pysam MM tag reading errors. Run meth_phaser_parallel with the -b/-r/-g/-vc inputs over the chr6 interval, applying the default best-accuracy block-connection filtering (voting confidence (votes difference)/total votes > 0.5 and a minimum read number of read coverage/(genome-wide block number/1000)) to methylation-haplotag unphased reads and infer neighbor SNV phase block relationships. Run meth_phaser_post_processing to apply the block-flipping list and emit the modified VCF (output.vcf) and BAM with merged haplotype assignments. Parse the per-chromosome neighbor relationship CSVs to count joined SNV phase blocks and compute the extended block length (expected: 3 blocks connected, 132,079 bp extension across HLA-E/HLA-C), then check the output VCF and read assignment CSVs for the haplotype-specific insertion reported in Fig. 4. Rationale: haplotype-specific methylation signals in ONT MM/ML tags provide evidence linking SNV blocks that read-length-limited SNV phasing (HapCUT2/WhatsHap) cannot connect.

## Related tools

- **MethPhaser** (Performs methylation-based read phasing and SNV-phase block chaining across the chr6 HLA coordinate interval) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (First-step driver that methylation-haplotags unphased reads and infers neighbor SNV phase block relationships over the interval) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Second-step script that applies the block-flipping list and emits the modified VCF and haplotagged BAM) — https://github.com/treangenlab/methphaser
- **samtools** (Restricts the haplotagged BAM to primary alignments (view -bF 2304) to avoid pysam MM tag errors)
- **WhatsHap** (Provides the haplotagged BAM and visualization GTF used as interval inputs to MethPhaser)
- **HapCUT2** (Produces the SNV-phased VCF that MethPhaser extends via block chaining)
- **pysam** (Used internally to read MM/ML methylation tags from the haplotagged BAM)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- Exactly 3 SNV phase blocks are connected into one block spanning the HLA region, confirmed by the per-chromosome neighbor relationship CSVs
- The extended phase block spans a 132,079 bp extension linking HLA-E and HLA-C within chr6:30,400,000-31,300,000
- The relationship output confirms correctness of each block connection against the trio-phased HG002 truth set (GIAB)
- The output.vcf contains the haplotype-specific insertion reported in Fig. 4 within the merged HLA-E/HLA-C block
- Block-connection voting confidence ((votes difference)/total votes) exceeds 0.5 with the minimum read number of read coverage/(genome-wide block number/1000) satisfied

## Limitations

- MethPhaser depends on initial SNV phasing accuracy; high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact its accuracy
- Results depend on accuracy of methylation detection, overall read length, and a certain minimum coverage of the dataset
- Sex chromosomes cannot be phased due to random X inactivation in females; performance across tissues is hard to predict due to tissue-specific methylation signals

## Evidence

- [methods] MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining.: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [methods] Voting confidence > 0.5 filter for block relationships: "(Votes for one haplotype-votes for the other one)/total votes > 0.5"
- [methods] Keep only primary alignments (samtools view -bF 2304): "samtools view -bF 2304 -o output.bam input.bam"
- [abstract] MethPhaser further improves phasing across HLA and multiple other medically relevant genes: "MethPhaser further improves phasing across HLA and multiple other medically relevant genes"
- [readme] README primary-alignment requirement and post-processing invocation: "./meth_phaser_post_processing -ib test_data/HLA.R10.haplotagged.bam -if test_data/work/ -ov test_data/output.vcf"
