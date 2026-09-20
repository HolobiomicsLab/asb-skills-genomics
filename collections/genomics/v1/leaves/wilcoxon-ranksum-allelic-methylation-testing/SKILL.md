---
name: wilcoxon-ranksum-allelic-methylation-testing
description: 'Use when extending SNV-based phase blocks with ONT methylation signals:
  haplotagged BAM (MM/ML tags), a HapCUT2-phased VCF, and a WhatsHap GTF are available
  and allele-specific CpGs must be identified to assign untagged reads.'
when_to_use_negative:
- 'The input includes sex chromosomes: random X inactivation in females produces inconsistent
  haplotype methylation patterns, so MethPhaser will not work on X/Y — restrict to
  autosomes.'
- 'Coverage is below ~30X ONT reads: SNV phasing methods (HapCUT2, WhatsHap) only
  work reasonably above that coverage, undermining the haplotag input to the rank
  sum test.'
- The BAM contains secondary or supplementary alignments, which trigger pysam MM tag
  reading errors — filter to primary alignments first or the test cannot run.
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: MethPhaser
  role: Implements the Wilcoxon rank sum test on base modification probabilities of
    SNV-haplotagged reads to identify haplotype-specific methylation, then methylation-votes
    reads and chains phase blocks
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: CLI driver that executes the per-block rank sum testing and iterative methylation
    haplotagging across the genome in parallel
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Consumes the work directory to emit the flipped/re-phased output VCF and the
    modified haplotagged BAM
  repo: https://github.com/treangenlab/methphaser
- name: WhatsHap
  role: Produces the SNV-haplotagged reads and the GTF that serve as the two-allele
    read groups compared by the rank sum test
- name: HapCUT2
  role: Provides the initial SNV phase blocks and the phased VCF that define the blocks
    within which allele-specific methylation is tested
- name: samtools
  role: Filters the BAM to primary alignments (samtools view -bF 2304) so MM tags
    can be read safely by pysam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:649ece1455e1cb1219b3bc38095a4040b54c8de9a9faea195b55d2861034a267
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
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# wilcoxon-ranksum-allelic-methylation-testing

## Summary

A statistical core of MethPhaser that applies the Wilcoxon rank sum test (p-value < 0.05) to base modification probabilities of SNV-haplotagged ONT reads to identify allele-specific (haplotype-specific) CpG methylation per phase block, which then drives methylation-based read voting and phase block bridging.

## When to use

Use when extending SNV-based phase blocks with ONT methylation signals: haplotagged BAM (MM/ML tags), a HapCUT2-phased VCF, and a WhatsHap GTF are available and allele-specific CpGs must be identified to assign untagged reads.

## When NOT to use

- The input includes sex chromosomes: random X inactivation in females produces inconsistent haplotype methylation patterns, so MethPhaser will not work on X/Y — restrict to autosomes.
- Coverage is below ~30X ONT reads: SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above that coverage, undermining the haplotag input to the rank sum test.
- The BAM contains secondary or supplementary alignments, which trigger pysam MM tag reading errors — filter to primary alignments first or the test cannot run.

## Inputs

- Methylation-annotated ONT BAM (WhatsHap haplotagged, MM/ML tags retained, primary alignments only via samtools view -bF 2304)
- HapCUT2-phased SNV VCF (e.g., HLA.R10.phased.vcf.gz)
- GTF file from WhatsHap visualization (e.g., LSK.filtered.gtf)
- Reference genome FASTA (e.g., GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna)

## Outputs

- Work directory with allele-specific CpG calls per phase block and neighbor block-relationship CSVs recording supporting read numbers
- Re-phased (block-chained, potentially flipped) output VCF from meth_phaser_post_processing
- Modified (re-haplotagged by methylation) output BAM

## How to apply

First restrict the input BAM to primary alignments only (`samtools view -bF 2304`), since secondary/supplementary alignments trigger pysam MM tag reading errors. Run `meth_phaser_parallel` with the haplotagged BAM, chr6/autosome GRCh38 no-alt reference, WhatsHap GTF, and HapCUT2-phased VCF; the tool computes per-SNV-phase-block distributions of base modification probabilities for the two SNP-haplotagged read sets and applies the Wilcoxon rank sum test at p-value < 0.05 to call statistically different (allele-specific) CpG locations. MethPhaser then iteratively haplotags untagged reads by methylation voting, requiring a minimum haplotype coverage of 3, a minimum of 3 votes per read, and a vote-percentage cutoff (-c) until no more reads can be assigned or the -k iteration limit is reached. Finally, block relationships are inferred from reads spanning consecutive block boundaries using the best-accuracy minimum read number (read coverage/(genome-wide block number/1000)) and a voting confidence of (votes_one - votes_other)/total_votes > 0.5. Post-processing via `meth_phaser_post_processing` emits the chained VCF and modified BAM.

## Related tools

- **MethPhaser** (Implements the Wilcoxon rank sum test on base modification probabilities of SNV-haplotagged reads to identify haplotype-specific methylation, then methylation-votes reads and chains phase blocks) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (CLI driver that executes the per-block rank sum testing and iterative methylation haplotagging across the genome in parallel) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Consumes the work directory to emit the flipped/re-phased output VCF and the modified haplotagged BAM) — https://github.com/treangenlab/methphaser
- **WhatsHap** (Produces the SNV-haplotagged reads and the GTF that serve as the two-allele read groups compared by the rank sum test)
- **HapCUT2** (Provides the initial SNV phase blocks and the phased VCF that define the blocks within which allele-specific methylation is tested)
- **samtools** (Filters the BAM to primary alignments (samtools view -bF 2304) so MM tags can be read safely by pysam)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- output.vcf exists and encodes chained (bridged) phase blocks relative to the input HapCUT2-phased VCF
- Neighbor relationship CSVs in the work directory record the number of supporting reads for each block connection, with voting confidence (votes difference)/total votes > 0.5
- Allele-specific CpGs are reported only where the Wilcoxon rank sum test yields p-value < 0.05 between the two haplotagged read sets
- Every methylation-assigned read has at least 3 votes and each haplotype has minimum coverage of 3 in the block
- Phase length N50 should increase (expected 78%-151% for ONT R9/R10 cell line data) at maintained phasing accuracy (83.4-98.7%) when benchmarked against the HG002 truth set

## Limitations

- Cannot improve phasing for sex chromosomes: random X inactivation in females leads to inconsistent haplotype methylation patterns; README says 'MethPhaser won't work on sex chromosomes'
- Depends on initial SNV phasing accuracy — high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact accuracy
- Results depend on accuracy of methylation detection, overall read length, and a minimum coverage of the dataset
- Improvement is limited in patient samples by smaller initial phase blocks from shorter reads (12–15 kbp N50)
- By default the largest homozygous/centromeric region is ignored (-ml -1); use -ml -2 only if the input has no large poorly mapped region

## Evidence

- [methods] Wilcoxon rank sum test identifies allele-specific CpGs: "We use the Wilcoxon rank sum test44,59 to determine whether the two allele’s base modification probabilities are statistically different (p-value < 0.05)."
- [methods] Input/output contract of MethPhaser: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [methods] Vote thresholds for read haplotype assignment: "a minimum number of votes (default 3) are required for a read’s haplotype assignment."
- [readme] Primary-alignment filter required before reading MM tags: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
- [readme] Sex chromosome exclusion: "!!! NOTE: MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes. !!!"
- [readme] Coverage recommendation: "We recommend using MethPhaser for at least 30X coverage ONT reads since most of the SNV phasing methods (HapCUT2, Whatshap) only work reasonably above that coverage !!!"
