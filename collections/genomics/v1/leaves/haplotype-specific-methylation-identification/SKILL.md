---
name: haplotype-specific-methylation-identification
description: Use when you have ONT long reads basecalled with MM/ML methylation tags
  that are haplotagged by SNV phase (WhatsHap haplotag), and you need to find CpGs
  whose modification probabilities distinguish the two haplotypes.
when_to_use_negative:
- 'Analysis targets human sex chromosomes — random X inactivation in females makes
  haplotype methylation patterns inconsistent (README: autosome only)'
- Input BAM still contains secondary or supplementary alignments — pysam MM tag reading
  will error
- Sequencing depth is well below ~30X ONT coverage, where upstream SNV phasing methods
  (HapCUT2, WhatsHap) do not work reasonably
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3307
tools:
- name: MethPhaser
  role: 'Implements the 3-step loop: Wilcoxon rank sum test for allele-specific CpGs,
    iterative methylation-based read haplotagging, and phase block bridging'
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: Parallel driver that runs methylation-based read phasing and SNV-phase block
    chaining on haplotagged BAM, reference FASTA, WhatsHap GTF, and phased VCF
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_post_processing
  role: Post-processing script emitting the flipped/re-phased output VCF and modified
    BAM from the work directory
  repo: https://github.com/treangenlab/methphaser
- name: WhatsHap
  role: Produces the initial SNV haplotagged BAM and visualization GTF consumed by
    MethPhaser
- name: HapCUT2
  role: Provides the SNV phase blocks that methylation signals extend
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:ec3c1fe9625a1ab3c5825955e540add86af02acaa05394bf1c58eb3a0a133290
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
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# haplotype-specific-methylation-identification

## Summary

Identify allele-specific (haplotype-specific) CpG methylation from methylation-annotated ONT reads, using the Wilcoxon rank sum test on base modification probabilities of SNV-haplotagged reads as the first step of MethPhaser's methylation-based phasing loop.

## When to use

Use when you have ONT long reads basecalled with MM/ML methylation tags that are haplotagged by SNV phase (WhatsHap haplotag), and you need to find CpGs whose modification probabilities distinguish the two haplotypes.

## When NOT to use

- Analysis targets human sex chromosomes — random X inactivation in females makes haplotype methylation patterns inconsistent (README: autosome only)
- Input BAM still contains secondary or supplementary alignments — pysam MM tag reading will error
- Sequencing depth is well below ~30X ONT coverage, where upstream SNV phasing methods (HapCUT2, WhatsHap) do not work reasonably

## Inputs

- Methylation-annotated (MM/ML tag) haplotagged BAM from ONT reads, primary alignments only (samtools view -bF 2304)
- HapCUT2-phased SNV VCF (e.g., HLA.R10.phased.vcf.gz)
- GTF file from WhatsHap visualization (e.g., LSK.filtered.gtf)
- Reference genome FASTA (e.g., GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna)
- Optional truth VCF (-vt) for benchmarking

## Outputs

- Allele-specific CpG locations per extended phase block (statistically different methylation calls)
- Haplotagged reads after iterative methylation voting (modified BAM via meth_phaser_post_processing)
- Chained/bridged phase block VCF (output.vcf)
- Neighbor relationship CSVs in the work directory recording supporting read counts per block connection

## How to apply

Start from WhatsHap-haplotagged BAMs (primary alignments only, samtools view -bF 2304) and HapCUT2-phased SNV VCFs, per MethPhaser's input contract. Within each SNV phase block, run a Wilcoxon rank sum test comparing the two alleles' base modification probabilities at CpG locations and retain CpGs that are statistically different (p-value < 0.05); these allele-specific CpGs feed the iterative methylation-voting step that assigns untagged reads (minimum haplotype coverage 3, minimum 3 votes per read). The identified haplotype-specific methylations then enable SNV-phase block chaining via reads haplotagged across block boundaries, filtered by best-accuracy minimum read number (read coverage/(genome-wide block number/1000)) and voting confidence ((votes for one haplotype - votes for the other)/total votes > 0.5). Note the parameter -ml -1 skips the largest homozygous/centromeric region by default; use -ml -2 when no large poorly mapped gap exists.

## Related tools

- **MethPhaser** (Implements the 3-step loop: Wilcoxon rank sum test for allele-specific CpGs, iterative methylation-based read haplotagging, and phase block bridging) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Parallel driver that runs methylation-based read phasing and SNV-phase block chaining on haplotagged BAM, reference FASTA, WhatsHap GTF, and phased VCF) — https://github.com/treangenlab/methphaser
- **meth_phaser_post_processing** (Post-processing script emitting the flipped/re-phased output VCF and modified BAM from the work directory) — https://github.com/treangenlab/methphaser
- **WhatsHap** (Produces the initial SNV haplotagged BAM and visualization GTF consumed by MethPhaser)
- **HapCUT2** (Provides the SNV phase blocks that methylation signals extend)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- output.vcf exists and encodes the chained phase blocks after post-processing
- Neighbor relationship CSVs in the work directory record the number of supporting reads for each block connection
- Allele-specific CpGs pass the Wilcoxon rank sum test threshold of p-value < 0.05
- Phase length N50 should increase substantially over SNV-only phasing (reported 78%-151% for ONT R9/R10 at 83.4-98.7% accuracy)
- Benchmarked against HG002 truth set, correctness of each SNV phase block connection can be checked against the trio-phased GIAB VCF

## Limitations

- MethPhaser won't work on sex chromosomes because of random X inactivation (README: autosome only)
- Depends on initial SNV phasing accuracy; high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact MethPhaser accuracy
- Results depend on accuracy of methylation detection, overall read length, and a minimum coverage of the dataset
- Improvement was limited in patient samples by initially smaller phase blocks (12-15 kbp read N50)
- By default (-ml -1) the largest homozygous/centromeric region is ignored; use -ml -2 only when no large poorly mapped region exists

## Evidence

- [methods] Wilcoxon rank sum test for allele-specific methylation: "We use the Wilcoxon rank sum test44,59 to determine whether the two allele’s base modification probabilities are statistically different (p-value < 0.05)."
- [methods] minimum votes and coverage thresholds: "a minimum number of votes (default 3) are required for a read’s haplotype assignment."
- [methods] input contract for MethPhaser: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [readme] primary alignment requirement: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
- [readme] sex chromosome exclusion: "!!! NOTE: MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes. !!!"
- [methods] voting confidence filter for block relationships: "(Votes for one haplotype-votes for the other one)/total votes > 0.5"
