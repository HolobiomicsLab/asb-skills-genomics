---
name: vcf-variant-call-comparison
description: Use when a called-variant VCF (e.g., from Longshot on PacBio SMS reads)
  must be validated or compared against a ground-truth or short-read call set (e.g.,
  ground_truth_variants.vcf, GIAB high-confidence sets).
when_to_use_negative:
- The task requires short indel calling from SMS reads — Longshot only calls SNVs
  and the article notes accurate short indel calling with SMS reads is challenging
  due to the high insertion/deletion error rate
- Input is structural-variant discovery rather than SNV genotyping — use assembly-
  or split-read-based SV callers instead (Longshot does not support SVs >50 bp even
  as genotyped variants)
- The truth set was generated from short-read data and the region of interest is a
  segmental duplication — the GIAB/Platinum Genomes sets are biased in favor of short-read
  technologies, so apparent Longshot false positives may be truth-set false negatives
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: 'Primary SNV caller: performs pileup candidate detection, pair-HMM realignment,
    haplotype-informed phased genotyping, and outputs a phased VCF'
  repo: https://github.com/pjedge/longshot
- name: HapCUT2
  role: Internal phasing engine used by Longshot in its iterative likelihood optimization
    of haplotypes and local genotypes
- name: FreeBayes
  role: Short-read variant caller used as a baseline call set for comparison against
    Longshot SMS-based calls
- name: SimLoRD
  role: Simulator used to generate the 30x PacBio reads in the example_data toy benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:ddce1aea33ba8729c4a7ec39d3c02cc2c08d8062de42c5f1b2e89c780ee6f2a9
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# vcf-variant-call-comparison

## Summary

Call SNVs from long SMS reads with Longshot and benchmark the resulting VCF against a short-read-derived or ground-truth VCF to quantify calling accuracy. This is the core evaluation pattern used to show Longshot matches short-read precision (≥0.9999) on simulated diploid genomes.

## When to use

Use when a called-variant VCF (e.g., from Longshot on PacBio SMS reads) must be validated or compared against a ground-truth or short-read call set (e.g., ground_truth_variants.vcf, GIAB high-confidence sets).

## When NOT to use

- The task requires short indel calling from SMS reads — Longshot only calls SNVs and the article notes accurate short indel calling with SMS reads is challenging due to the high insertion/deletion error rate
- Input is structural-variant discovery rather than SNV genotyping — use assembly- or split-read-based SV callers instead (Longshot does not support SVs >50 bp even as genotyped variants)
- The truth set was generated from short-read data and the region of interest is a segmental duplication — the GIAB/Platinum Genomes sets are biased in favor of short-read technologies, so apparent Longshot false positives may be truth-set false negatives

## Inputs

- Sorted, indexed BAM/CRAM of error-prone long SMS reads (e.g., PacBio SMRT reads) aligned to the reference
- Indexed FASTA reference genome (e.g., example_data/genome.fa, 3 contigs of 200 kb each)
- Ground-truth variant VCF for validation (e.g., ground_truth_variants.vcf with 714 true variants, or GIAB high-confidence call sets)
- Optionally, a gzipped/tabix-indexed VCF of potential variants to genotype (--potential_variants)

## Outputs

- Phased VCF of called SNVs with genotype qualities (e.g., longshot_output.vcf)
- Accuracy metrics (precision, recall, FDR) from comparison to the ground-truth VCF
- Optionally, haplotype-separated BAM with HP:i:1/HP:i:2 read tags for downstream analysis

## How to apply

Run Longshot on a sorted BAM of error-prone SMS reads aligned to an indexed FASTA reference, using its default pileup parameters (min_cov=6, max_cov=8000, min_mapq=20, min_alt_count=3, min_alt_frac=0.125) and built-in filters (Fisher's exact strand-bias p<0.01, read-depth cap d+5√d, density filter >10 SNVs per 500 bp, allele quality threshold 7.0). Longshot performs pair-HMM local realignment, haplotype-informed phased genotyping, and iterative HapCUT2-based optimization, producing a phased VCF (and optionally an HP-tagged haplotype-separated BAM). To quantify accuracy, compare the called SNVs against the truth VCF (e.g., the 714 true variants in example_data/ground_truth_variants.vcf, or GIAB/Platinum Genomes call sets for real data), computing precision, recall, and false-discovery rate. For short-read baselines, call variants with tools like FreeBayes or GATK HaplotyperCaller and apply a GQ cutoff of 50; for Longshot use a variable GQ cutoff matched to the median read coverage.

## Related tools

- **Longshot** (Primary SNV caller: performs pileup candidate detection, pair-HMM realignment, haplotype-informed phased genotyping, and outputs a phased VCF) — https://github.com/pjedge/longshot
- **HapCUT2** (Internal phasing engine used by Longshot in its iterative likelihood optimization of haplotypes and local genotypes)
- **FreeBayes** (Short-read variant caller used as a baseline call set for comparison against Longshot SMS-based calls)
- **SimLoRD** (Simulator used to generate the 30x PacBio reads in the example_data toy benchmark)

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- On the example_data toy benchmark, the output VCF (longshot_output.vcf) is produced in roughly 5–10 seconds on a typical desktop machine and its calls can be directly compared to the 714 true variants in ground_truth_variants.vcf
- In the simulated diploid-genome benchmark at 20–60× coverage, genome-wide SNV calling precision should be ≥0.9999 for both short-read and SMS read-based calling
- Filtered candidate positions should show no variants with Fisher's exact strand-bias p<0.01, read depth >d+5√d (d = median read depth), or >10 SNVs in any 500 bp window
- The output VCF must be a valid phased VCF with genotype information; HapCUT2 phasing runs unless -n/--no_haps is set
- Short-read baseline calls must pass the fixed GQ cutoff of 50; Longshot calls should be filtered with a GQ cutoff matched to the median read coverage

## Limitations

- Longshot only calls SNVs; it can genotype but not phase indels supplied via an input VCF, and SVs >50 bp are unsupported
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, limiting variant detection in some duplicated regions
- GIAB and Platinum Genomes validation sets were generated with short reads and are biased in favor of short-read technologies, so some Longshot 'false positives' may be truth-set false negatives
- Allele observations with phred-scaled quality below the default threshold (7.0) are discarded, reducing the effective read depth for SMS reads
- High insertion/deletion error rates of SMS reads make accurate short indel calling challenging

## Evidence

- [readme] The directory example_data contains a simulated toy dataset that can be used to test out Longshot: "The directory example_data contains a simulated toy dataset that can be used to test out Longshot"
- [readme] Execution on example data: "Execution should take around 5 to 10 seconds on a typical desktop machine. The output can be compared to ground_truth_variants.vcf for accuracy."
- [discussion] Candidate identification by genotype likelihood: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
- [discussion] Strand bias filter: [evidence span withheld — anchor status: not_found]
- [discussion] GQ cutoffs for comparison: "The GQ cutoff was fixed at 50 for short reads. For Longshot, we used a variable GQ cutoff (matched to the median read coverage)"
- [discussion] Copy-number depth filter: [evidence span withheld — anchor status: not_found]
- [results] Simulation benchmark finding: [evidence span withheld — anchor status: not_found]
- [discussion] Truth set bias caveat: "The GIAB and Platinum Genomes variant sets used to assess variant-calling accuracy in this study were generated using short-read datasets, are therefore biased in favor of short-read technologies"
