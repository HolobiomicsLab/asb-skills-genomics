---
name: bam-alignment-file-processing
description: Use when long-read (PacBio/ONT) reads have been aligned to a reference
  FASTA and you need to process the resulting BAM/CRAM for SNV calling and haplotyping,
  or to emit haplotype-separated reads.
when_to_use_negative:
- The task is calling indels or structural variants de novo — Longshot only calls
  SNVs and can only genotype indels given in an input VCF
- The input is unaligned reads (FASTQ) with no BAM/CRAM alignment and indexed reference
  available
- The data are short Illumina reads better served by short-read callers such as GATK
  HaplotypeCaller or FreeBayes
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: 'Primary caller: processes the sorted/indexed BAM of long reads against an
    indexed FASTA to produce a phased VCF and optional haplotype-tagged BAM'
  repo: https://github.com/pjedge/longshot
- name: SimLoRD
  role: Generates simulated PacBio BAM input (e.g. 30x coverage reads) for validating
    the BAM-processing workflow on the example_data toy dataset
  repo: https://bitbucket.org/genomeinformatics/simlord/
- name: HapCUT2
  role: Called internally by Longshot to estimate the most likely pair of haplotypes
    during iterative phased genotyping of the BAM-derived candidate SNVs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:bdec94d426b917c5d8920f66bf3920d9af70c2aed5935cb217caef3a604af525
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - FreeBayes
  - HapCUT2
  - Longshot
  - Pacific Biosciences
  - SimLoRD
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# bam-alignment-file-processing

## Summary

Prepare and consume sorted, indexed BAM/CRAM alignments of error-prone single-molecule long reads (PacBio SMRT, ONT) as input to Longshot diploid SNV calling, with coverage/mapping-quality filters shaping which positions become candidate SNVs.

## When to use

Use when long-read (PacBio/ONT) reads have been aligned to a reference FASTA and you need to process the resulting BAM/CRAM for SNV calling and haplotyping, or to emit haplotype-separated reads.

## When NOT to use

- The task is calling indels or structural variants de novo — Longshot only calls SNVs and can only genotype indels given in an input VCF
- The input is unaligned reads (FASTQ) with no BAM/CRAM alignment and indexed reference available
- The data are short Illumina reads better served by short-read callers such as GATK HaplotypeCaller or FreeBayes

## Inputs

- Sorted, indexed BAM/CRAM file of error-prone long reads (e.g. PacBio SMRT reads aligned with BLASR)
- Indexed FASTA reference genome (e.g. hs37d5 or hg38)
- Optional gzipped, tabix-indexed VCF of potential variants to genotype

## Outputs

- Phased VCF file with called variants and haplotype information
- Optional BAM with reads tagged HP:i:1 / HP:i:2 for haplotype assignment

## How to apply

Take a sorted, indexed BAM (CRAM supported) of error-prone reads aligned to an indexed FASTA reference and run Longshot with default filters: min_cov=6, max_cov=8000, min_mapq=20, min_alt_count=3, min_alt_frac=0.125, hap_assignment_qual=20.0. Read-depth outliers are handled by filtering variants with depth >d+5√d (or via --auto_max_cov), where d is the median read depth across the dataset, and allele observations with phred-scaled quality below 7.0 are discarded. Optionally supply a gzipped/tabix-indexed potential-variants VCF via -v to genotype instead of pileup detection, or -O to write a BAM where reads carry HP:i:1 / HP:i:2 haplotype tags. HapCUT2 is invoked internally to phase the jointly genotyped candidate SNVs. Rationale: pileup-based candidate identification with genotype-likelihood scoring plus local pair-HMM realignment allows accurate heterozygous SNV calling despite high SMS error rates.

## Related tools

- **Longshot** (Primary caller: processes the sorted/indexed BAM of long reads against an indexed FASTA to produce a phased VCF and optional haplotype-tagged BAM) — https://github.com/pjedge/longshot
- **SimLoRD** (Generates simulated PacBio BAM input (e.g. 30x coverage reads) for validating the BAM-processing workflow on the example_data toy dataset) — https://bitbucket.org/genomeinformatics/simlord/
- **HapCUT2** (Called internally by Longshot to estimate the most likely pair of haplotypes during iterative phased genotyping of the BAM-derived candidate SNVs)

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Output VCF is produced and its calls are comparable to ground_truth_variants.vcf (714 true variants) on the example_data toy dataset
- Runtime is roughly 5–10 seconds on a typical desktop machine for the 3-contig, 600 kb example dataset at 30x coverage
- Reads failing min_mapq=20 are excluded before coverage assessment; candidate positions must have ≥6 reads passing filters
- Genome-wide SNV calling precision ≥0.9999 across 20–60x simulated coverages, matching the reported simulation benchmark
- With -O, output BAM reads carry only HP:i:1 or HP:i:2 tags and any pre-existing HP/PS tags are removed

## Limitations

- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge
- Allele quality filtering (threshold 7.0 by default) reduces the effective read depth for SMS reads
- Accurate calling of short indels using SMS reads is challenging due to the high insertion/deletion error rate
- The --auto_max_cov flag is slower because it computes mean coverage per region; --stable_alignment logspace pair-HMM is significantly slower and tests show it is not necessary for highly error-prone PacBio CLR reads

## Evidence

- [readme] It takes as input an aligned BAM/CRAM file and outputs a phased VCF file with variants and haplotype information.: "It takes as input an aligned BAM/CRAM file and outputs a phased VCF file with variants and haplotype information."
- [readme] sorted, indexed BAM file with error-prone reads (CRAM files also supported): "sorted, indexed BAM file with error-prone reads (CRAM files also supported)"
- [methods] Minimum mapping quality to use a read. [default: 20]: "Minimum mapping quality to use a read. [default: 20]"
- [discussion] Variants with read depth >d+5√d, where d is the median read depth across the entire dataset were filtered out.: [evidence span withheld — anchor status: not_found]
- [readme] Write new bam file with haplotype tags (HP:i:1 and HP:i:2) for reads assigned to each haplotype, any existing HP and PS tags are removed: "Write new bam file with haplotype tags (HP:i:1 and HP:i:2) for reads assigned to each haplotype, any existing HP and PS tags are removed"
- [results] Across all simulated coverages (20-60×), genome-wide SNV calling precision was consistently high (≥0.9999) for both short-read and SMS read-based SNV calling.: [evidence span withheld — anchor status: not_found]
