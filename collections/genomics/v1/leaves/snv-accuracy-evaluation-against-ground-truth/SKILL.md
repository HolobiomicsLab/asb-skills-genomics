---
name: snv-accuracy-evaluation-against-ground-truth
description: Use when you have simulated long-read (PacBio SMS) reads with a known
  variant truth set and need to quantify SNV calling precision/recall for a long-read
  variant caller such as Longshot.
when_to_use_negative:
- Input is real sequencing data without a validated truth set — no ground-truth comparison
  is possible
- Goal is indel or structural variant detection — Longshot only calls SNVs (indels
  are genotyped only if supplied in an input VCF; SVs >50 bp are unsupported)
- Input reads are short Illumina reads — use short-read callers such as FreeBayes
  or GATK HaplotypeCaller instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: SNV caller executed on simulated PacBio reads to produce the VCF that is scored
    against ground truth
  repo: https://github.com/pjedge/longshot
- name: SimLoRD
  role: Simulator used to generate the 30x coverage PacBio reads and the accompanying
    714-variant truth set
  repo: https://bitbucket.org/genomeinformatics/simlord/
- name: FreeBayes
  role: Short-read variant caller used as the accuracy comparison baseline in the
    simulated benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:7b99c62fe84de5241e93069c2320f97c3ebe94700481da789248ad1889f00683
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# snv-accuracy-evaluation-against-ground-truth

## Summary

Benchmark SNV calling accuracy by running Longshot on simulated PacBio SMS reads (e.g., the SimLoRD-generated 30x example dataset: 3 x 200 kb contigs, 714 true SNVs) and comparing the called VCF against ground_truth_variants.vcf to compute precision/recall.

## When to use

Use when you have simulated long-read (PacBio SMS) reads with a known variant truth set and need to quantify SNV calling precision/recall for a long-read variant caller such as Longshot.

## When NOT to use

- Input is real sequencing data without a validated truth set — no ground-truth comparison is possible
- Goal is indel or structural variant detection — Longshot only calls SNVs (indels are genotyped only if supplied in an input VCF; SVs >50 bp are unsupported)
- Input reads are short Illumina reads — use short-read callers such as FreeBayes or GATK HaplotypeCaller instead

## Inputs

- example_data/genome.fa — simulated reference genome (3 contigs x 200 kb)
- example_data/pacbio_reads_30x.bam — 30x coverage simulated PacBio reads (SimLoRD), sorted and indexed
- example_data/ground_truth_variants.vcf — 714 true variants for validation

## Outputs

- example_data/longshot_output.vcf — phased VCF of called SNVs
- Accuracy metrics (precision, recall, FDR) from comparison against ground_truth_variants.vcf

## How to apply

Load the simulated toy dataset in example_data, which contains a reference genome of 3 contigs each of length 200 kb and 30x coverage simulated PacBio reads generated with SimLoRD. Run Longshot with default parameters (min_cov=6, max_cov=8000, min_mapq=20, min_alt_count=3, min_alt_frac=0.125, hap_assignment_qual=20.0) via 'longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf'. Parse the resulting VCF of called SNVs and compare called variants to the 714 true variants in ground_truth_variants.vcf to quantify calling accuracy (precision/recall/FDR). In the article's simulated diploid-genome benchmark across 20-60x coverage, genome-wide SNV calling precision was consistently high (>=0.9999) for both short-read and SMS read-based SNV calling, so results well below this range indicate a pipeline problem. Note that GQ thresholds differ by technology: 50 for short reads, and a variable GQ cutoff matched to median read coverage for Longshot.

## Related tools

- **Longshot** (SNV caller executed on simulated PacBio reads to produce the VCF that is scored against ground truth) — https://github.com/pjedge/longshot
- **SimLoRD** (Simulator used to generate the 30x coverage PacBio reads and the accompanying 714-variant truth set) — https://bitbucket.org/genomeinformatics/simlord/
- **FreeBayes** (Short-read variant caller used as the accuracy comparison baseline in the simulated benchmark)

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- longshot_output.vcf is produced and parses as a valid VCF with called SNV records
- Called variants are directly comparable to the 714 true variants in ground_truth_variants.vcf via coordinate/genotype matching
- Reported genome-wide precision should be near the article's reported value of >=0.9999 for simulated coverages of 20-60x
- Execution completes in roughly 5-10 seconds on a typical desktop machine; markedly slower runs suggest misconfigured parameters or unindexed inputs
- Default Longshot parameters (min_cov=6, max_cov=8000, min_mapq=20) were used, matching the documented run for the toy dataset

## Limitations

- The toy dataset is a small simulation (3 x 200 kb contigs) intended for testing, not a full genome benchmark
- Accurate calling of short indels using SMS reads is challenging due to the high insertion/deletion error rate
- Allele quality filtering (phred-scaled quality score below 7.0 by default) reduces the effective read depth for SMS reads
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge

## Evidence

- [readme] 30x coverage simulated pacbio reads generated using SimLoRD: "30x coverage simulated pacbio reads generated using [SimLoRD](https://bitbucket.org/genomeinformatics/simlord/) (example_data/pacbio_reads_30x.bam)"
- [readme] The 714 true variants for validation: "- The 714 "true" variants for validation (example_data/ground_truth_variants.vcf)"
- [readme] Output compared to ground truth for accuracy: "Execution should take around 5 to 10 seconds on a typical desktop machine. The output can be compared to ground_truth_variants.vcf for accuracy."
- [discussion] Simulated benchmark precision across coverages: [evidence span withheld — anchor status: not_found]
