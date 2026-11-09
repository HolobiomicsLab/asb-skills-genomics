---
name: hla-region-phase-block-verification
description: Use when asked to verify or reproduce methylation-based SNV phase block
  chaining in the HLA region from WhatsHap-haplotagged ONT BAMs and HapCUT2-phased
  VCFs, or to check whether SNV-only phasing fails to link HLA-E and HLA-C.
when_to_use_negative:
- Analysis targets sex chromosomes — random X inactivation in females produces inconsistent
  haplotype methylation patterns, and the README warns MethPhaser 'won't work on sex
  chromosomes'; autosomes only
- Coverage is below ~30X — the README notes SNV phasing methods (HapCUT2, WhatsHap)
  only work reasonably above that coverage
- The input BAM contains secondary or supplementary alignments, which will trigger
  pysam MM tag reading errors unless first filtered with samtools view -bF 2304
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MethPhaser
  role: Core method that performs methylation-based read phasing and SNV-phase block
    chaining using ONT methylation signals
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: Methylation-haplotags unphased reads and infers neighbor SNV phase block relationships
    across the HLA interval using majority-vote block connection
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Applies the block-flipping list and emits the modified VCF (output.vcf) and
    BAM with merged haplotype assignments
  repo: https://github.com/treangenlab/methphaser
- name: samtools
  role: Filters the input BAM to primary alignments only via samtools view -bF 2304
    before MethPhaser runs
- name: WhatsHap
  role: Provides the haplotagged BAM (haplotag module) that MethPhaser takes as input
- name: HapCUT2
  role: Produces the initial SNV-phased VCF consumed by MethPhaser as -vc input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:88e06fa9019d15433d60d5e6fd7dedab79c98e7efb25217060d986241853d270
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# hla-region-phase-block-verification

## Summary

Reproduces MethPhaser's methylation-assisted extension of SNV phase blocks across the HLA Class I region (chr6:30,400,000-31,300,000) on HG002 R10 ONT data, verifying that HLA-E and HLA-C are linked into a single block with a 132,079 bp extension.

## When to use

Use when asked to verify or reproduce methylation-based SNV phase block chaining in the HLA region from WhatsHap-haplotagged ONT BAMs and HapCUT2-phased VCFs, or to check whether SNV-only phasing fails to link HLA-E and HLA-C.

## When NOT to use

- Analysis targets sex chromosomes — random X inactivation in females produces inconsistent haplotype methylation patterns, and the README warns MethPhaser 'won't work on sex chromosomes'; autosomes only
- Coverage is below ~30X — the README notes SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above that coverage
- The input BAM contains secondary or supplementary alignments, which will trigger pysam MM tag reading errors unless first filtered with samtools view -bF 2304

## Inputs

- WhatsHap haplotagged BAM of HG002 R10 60X ONT reads restricted to the HLA region (primary alignments only)
- chr6 GRCh38 reference (GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna)
- Filtered GTF from WhatsHap visualization (LSK.filtered.gtf)
- HapCUT2-phased VCF (HLA.R10.phased.vcf.gz)

## Outputs

- Per-chromosome neighbor relationship CSVs quantifying connected SNV phase blocks across the HLA interval
- Modified phased VCF (output.vcf) with merged haplotype assignments and extended phase blocks
- BAM with merged methylation-based haplotype assignments
- Extended HLA phase block length measurement (expected 132,079 bp extension linking HLA-E and HLA-C)

## How to apply

Restrict the WhatsHap haplotagged HG002 R10 BAM to primary alignments with `samtools view -bF 2304` (secondary/supplementary alignments trigger pysam MM tag reading errors), then run meth_phaser_parallel with -b/-r/-g/-vc inputs over chr6:30,400,000-31,300,000 using the default best-accuracy block-connection filtering: voting confidence (votes difference)/total votes > 0.5 and minimum read number = read coverage/(genome-wide block number/1000). Run meth_phaser_post_processing to apply the block-flipping list and emit output.vcf and the merged haplotagged BAM. Parse the per-chromosome neighbor relationship CSVs to count connected SNV phase blocks and compute the extended block length, then inspect the output VCF and read assignment CSVs for the haplotype-specific insertion reported in Fig. 4. Correctness is confirmed when exactly 3 SNV phase blocks are joined spanning the HLA interval with a 132,079 bp extension linking HLA-E and HLA-C, and the relationship output confirms each block connection against the HG002 trio-phased GIAB truth.

## Related tools

- **MethPhaser** (Core method that performs methylation-based read phasing and SNV-phase block chaining using ONT methylation signals) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Methylation-haplotags unphased reads and infers neighbor SNV phase block relationships across the HLA interval using majority-vote block connection) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Applies the block-flipping list and emits the modified VCF (output.vcf) and BAM with merged haplotype assignments) — https://github.com/treangenlab/methphaser
- **samtools** (Filters the input BAM to primary alignments only via samtools view -bF 2304 before MethPhaser runs)
- **WhatsHap** (Provides the haplotagged BAM (haplotag module) that MethPhaser takes as input)
- **HapCUT2** (Produces the initial SNV-phased VCF consumed by MethPhaser as -vc input)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- Exactly 3 SNV phase blocks are connected into one block spanning chr6:30,400,000-31,300,000
- Extended block length equals the reported 132,079 bp extension linking HLA-E and HLA-C
- The per-block relationship output confirms correctness of each block connection (benchmarkable against the GIAB trio-phased HG002 truth VCF)
- The haplotype-specific insertion reported in Fig. 4 appears within the merged HLA-E/HLA-C block in the output VCF and read assignment CSVs
- Block connections satisfy the voting confidence filter (votes difference)/total votes > 0.5 with the minimum read number = read coverage/(genome-wide block number/1000)

## Limitations

- MethPhaser cannot improve phasing for human sex chromosomes in clinical samples because random X inactivation in females leads to an inconsistent haplotype pattern
- Accuracy depends on the initial SNV phasing quality; high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact MethPhaser
- Results depend on the accuracy of methylation detection, overall read length, and a certain minimum coverage of the dataset; improvement is limited in patient samples by initially smaller phase blocks from shorter reads (12–15 kbp N50)

## Evidence

- [other] MethPhaser extends the HLA-region phase block by 132,079 bp while connecting 3 SNV phase blocks: [evidence span withheld — anchor status: not_found]
- [readme] samtools view -bF 2304 -o output.bam input.bam: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
- [methods] MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining.: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [methods] Best accuracy, with read coverage/(genome-wide block number/1000) as a minimum read number supporting block relationship assignment requirement: "Best accuracy, with read coverage/(genome-wide block number/1000) as a minimum read number supporting block relationship assignment requirement"
- [discussion] MethPhaser relies on a majority vote to make decisions about combining existing phase blocks: "MethPhaser relies on a majority vote to make decisions about combining existing phase blocks"
