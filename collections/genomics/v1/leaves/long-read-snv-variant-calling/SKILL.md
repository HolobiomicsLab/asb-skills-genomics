---
name: long-read-snv-variant-calling
description: Use when input is a sorted, indexed BAM/CRAM of long error-prone reads
  (PacBio or ONT) aligned to an indexed reference FASTA and you need phased, diploid
  SNV calls with precision/recall evaluation against GIAB truth sets.
when_to_use_negative:
- The input is short-read Illumina data — the paper shows short-read callers like
  FreeBayes (GQ cutoff fixed at 50) are the appropriate choice, though they under-call
  in segmental duplications with >=95-99% identity
- The goal is short indel discovery from SMS reads — Longshot only calls SNVs (it
  can genotype indels only if given in an input VCF), and accurate indel calling from
  SMS reads is challenging due to the high insertion/deletion error rate
- The target is structural-variant calling — use an SV method (e.g., Delly or SMRT-SV)
  rather than this SNV-calling skill
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: 'Primary SNV caller: joint SNV detection, pair-HMM local realignment, and
    haplotype-informed phased genotyping of long-read BAMs'
  repo: https://github.com/pjedge/longshot
- name: HapCUT2
  role: Internal phasing engine used iteratively by Longshot to estimate the most
    likely pair of haplotypes during likelihood optimization
- name: WhatsHap
  role: 'Comparator: genotypes the Longshot candidate-variant VCF for benchmarking
    precision/recall'
- name: FreeBayes
  role: Short-read comparator used on Illumina BAMs (GQ cutoff 50) for Table 2 segmental-duplication
    comparisons
- name: SimLoRD
  role: Simulator of PacBio reads (e.g., 30x over a 3-contig, 200 kb reference) used
    to build the example toy dataset
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:2948c58ee42eec06b6efa8421167953739a9a0d3354ed81a3389b99d728faf6a
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
  - WhatsHap
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# long-read-snv-variant-calling

## Summary

Calling and phasing SNVs from error-prone single-molecule sequencing (SMS) long reads (PacBio SMRT, Oxford Nanopore) using Longshot, which combines pileup-based candidate SNV detection with haplotype-informed genotyping. It enables accurate heterozygous SNV calls in diploid genomes, including segmental duplications inaccessible to short reads.

## When to use

Use when input is a sorted, indexed BAM/CRAM of long error-prone reads (PacBio or ONT) aligned to an indexed reference FASTA and you need phased, diploid SNV calls with precision/recall evaluation against GIAB truth sets.

## When NOT to use

- The input is short-read Illumina data — the paper shows short-read callers like FreeBayes (GQ cutoff fixed at 50) are the appropriate choice, though they under-call in segmental duplications with >=95-99% identity
- The goal is short indel discovery from SMS reads — Longshot only calls SNVs (it can genotype indels only if given in an input VCF), and accurate indel calling from SMS reads is challenging due to the high insertion/deletion error rate
- The target is structural-variant calling — use an SV method (e.g., Delly or SMRT-SV) rather than this SNV-calling skill

## Inputs

- Sorted, indexed BAM/CRAM of PacBio SMRT or Oxford Nanopore long reads aligned to a reference (e.g., NGMLR- or BLASR-aligned PacBio BAMs at 23–64x coverage)
- Indexed FASTA reference genome (e.g., hs37d5 or hg38)
- Optional gzipped/tabix-indexed VCF of potential variants to genotype (e.g., Longshot candidate SNVs for WhatsHap genotyping)
- GIAB high-confidence variant call sets for accuracy evaluation

## Outputs

- Phased VCF of SNV calls with genotype qualities (GQ) and haplotype information
- Optional haplotype-tagged BAM (HP:i:1 / HP:i:2) via --out_bam
- Per-sample precision, recall, FDR, and Ts/Tv metrics computed against GIAB truth sets

## How to apply

Install Longshot via Bioconda ('conda install longshot') or compile from source with cargo (Rust >= 1.40.0, zlib, clang, gcc). Run 'longshot --bam <BAM/CRAM> --ref <FASTA> --out <VCF>' on each sample; for whole genomes process chromosomes separately with the --region option and set max coverage to mean_coverage + 5*sqrt(mean_coverage) using the -A flag. Key defaults: min_cov=6, max_cov=8000, min_mapq=20, min_alt_count=3, min_alt_frac=0.125, hap_assignment_qual=20.0; Longshot identifies candidates by genotype likelihood on the pileup, locally realigns reads to ref/alt with a pair-HMM, and jointly phases/genotypes candidates via iterative HapCUT2 optimization. Filter outputs with a two-tailed Fisher's exact strand-bias test (p < 0.01), a density filter (>10 SNVs in 500 bp), read depth > d + 5*sqrt(d), and a variable GQ cutoff matched to median read coverage. For benchmarking, restrict calls to GIAB high-confidence regions, count TP/FP/FN, and report precision = TP/(TP+FP) and recall = TP/(TP+FN).

## Related tools

- **Longshot** (Primary SNV caller: joint SNV detection, pair-HMM local realignment, and haplotype-informed phased genotyping of long-read BAMs) — https://github.com/pjedge/longshot
- **HapCUT2** (Internal phasing engine used iteratively by Longshot to estimate the most likely pair of haplotypes during likelihood optimization)
- **WhatsHap** (Comparator: genotypes the Longshot candidate-variant VCF for benchmarking precision/recall)
- **FreeBayes** (Short-read comparator used on Illumina BAMs (GQ cutoff 50) for Table 2 segmental-duplication comparisons)
- **SimLoRD** (Simulator of PacBio reads (e.g., 30x over a 3-contig, 200 kb reference) used to build the example toy dataset)

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- On the bundled example dataset, 'longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf' completes in ~5-10 seconds and its calls can be compared to the 714 true variants in ground_truth_variants.vcf
- On NA12878 PacBio (NGMLR-aligned, 44x), expected precision ~0.995 and recall ~0.968 with FDR ~0.5%, outperforming WhatsHap (0.972/0.975) and Clairvoyante (0.984/0.957)
- NA24385 recall should rise from 0.9608 at 28x to 0.9798 at 64x, with NA12878 30x P=0.9942/R=0.9592 and 45x R=0.9734
- Filtering calls within 5 bp of Mills + 1000 Genomes Gold Standard indels should reduce false positives by 34-45% and lower FDR to 0.3% with only slight recall loss
- In segmental duplications with >=99% similarity, SMS-based calls should far exceed Illumina calls (78,851 vs 18,684) and Illumina-only Ts/Tv should be ~1.55 versus ~1.99 for SMS-only calls

## Limitations

- Longshot does not consider indels as candidate variants, which generates false positives near indels (71.4% of NA12878 FPs within 5 bp of a true indel); filtering known common indels costs a small amount of recall
- False negatives are enriched in homopolymers >=5 bp (19.5%, ~3.4x random expectation)
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge
- GIAB and Platinum Genomes truth sets were generated using short-read datasets and are biased in favor of short-read technologies; some Longshot FPs may correspond to FNs in GIAB call sets
- Allele observations with phred-scaled quality below the threshold (7.0 default) are discarded, reducing effective read depth for SMS reads
- The -A (--auto_max_cov) option is slower than a fixed max_cov, and --stable_alignment (logspace pair HMM) is significantly slower though tests showed it unnecessary for PacBio CLR reads

## Evidence

- [abstract] Longshot leverages haplotype information in SMS reads for joint SNV detection and phasing: "We introduce a variant calling method, Longshot, which leverages the haplotype information present in SMS reads to accurately detect and phase single-nucleotide variants (SNVs) in diploid genomes."
- [discussion] Candidate SNVs identified from pileup by genotype likelihood, pair-HMM realignment, joint phased genotyping: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
- [discussion] Fisher's exact strand-bias filter: [evidence span withheld — anchor status: not_found]
- [discussion] Coverage and density filters: [evidence span withheld — anchor status: not_found]
- [readme] Example dataset invocation and runtime: "longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf"
- [discussion] Indel-proximate FP mechanism: [evidence span withheld — anchor status: not_found]
- [readme] Longshot supports SNV calling only; indels only via input VCF genotyping: "Currently, it only calls single nucleotide variants (SNVs), but it can genotype indels if they are given in an input VCF."
- [methods] Recommended installation via Bioconda: "It is recommended to install Longshot using [Bioconda](https://bioconda.github.io/):"
