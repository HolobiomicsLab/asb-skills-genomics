---
name: simulation-based-variant-caller-benchmarking
description: Use when comparing SNV calling accuracy between short-read-based (e.g.
  GATK, FreeBayes) and SMS read-based (e.g. Longshot) callers, and no unbiased truth
  set exists — simulation provides ground-truth variants for precision/recall quantification.
when_to_use_negative:
- Input is real (non-simulated) data intended for downstream biological interpretation
  — simulation output is only a benchmark, not usable variant calls
- Assessing indel or structural variant calling — Longshot calls SNVs only (indels
  must be supplied in an input VCF for genotyping, and SVs >50 bp are unsupported)
- A mature, unbiased high-confidence truth set exists (e.g. GIAB) — though note the
  article warns GIAB/Platinum Genomes sets are biased toward short-read technologies
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: SMS-read-based SNV caller being benchmarked; called with longshot --bam ...
    --ref ... --out ... using default parameters
  repo: https://github.com/pjedge/longshot
- name: SimLoRD
  role: Simulator used to generate 30× coverage PacBio reads from the simulated diploid
    genome
- name: FreeBayes
  role: Short-read variant caller used as the benchmarking comparison baseline
- name: GATK HaplotyperCaller
  role: Short-read variant calling tool cited as the developed-for-short-read baseline
    class
- name: HapCUT2
  role: Read-based haplotype phasing algorithm used inside Longshot's iterative optimization
    during calling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:d1c7b3e9dc7bccc9c018240a3354f302a0cbd5fef87fc3cd045797ec18b3e638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - FreeBayes
  - GATK HaplotyperCaller
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
yes_rate: 0.75
n_claims: 12
---

# simulation-based-variant-caller-benchmarking

## Summary

Benchmark SNV calling accuracy by simulating a diploid genome with known heterozygous variants and generating synthetic short-read (Illumina) and long-read (PacBio SMS) datasets, then comparing caller output against ground-truth VCFs at matched coverages (20–60×).

## When to use

Use when comparing SNV calling accuracy between short-read-based (e.g. GATK, FreeBayes) and SMS read-based (e.g. Longshot) callers, and no unbiased truth set exists — simulation provides ground-truth variants for precision/recall quantification.

## When NOT to use

- Input is real (non-simulated) data intended for downstream biological interpretation — simulation output is only a benchmark, not usable variant calls
- Assessing indel or structural variant calling — Longshot calls SNVs only (indels must be supplied in an input VCF for genotyping, and SVs >50 bp are unsupported)
- A mature, unbiased high-confidence truth set exists (e.g. GIAB) — though note the article warns GIAB/Platinum Genomes sets are biased toward short-read technologies

## Inputs

- Simulated diploid reference genome FASTA (e.g. 3 contigs × 200 kb)
- Ground-truth variant VCF (e.g. ground_truth_variants.vcf with 714 true variants)
- Simulated PacBio SMS read alignments (BAM, e.g. pacbio_reads_30x.bam generated with SimLoRD)
- Simulated paired-end Illumina read alignments (BAM, 20–60× coverage)
- Indexed FASTA reference matching each BAM

## Outputs

- Called variant VCF per caller/coverage combination (e.g. longshot_output.vcf)
- Precision/recall (and FDR) tables per caller, coverage, and genomic region (genome-wide and segmental duplications)
- Haplotype-separated BAM from Longshot (--out_bam) for downstream phasing accuracy checks

## How to apply

Simulate a diploid genome from a reference sequence by introducing heterozygous SNVs at a fixed rate (the paper used rate = 0.001), then generate simulated paired-end Illumina reads and PacBio SMS reads with SimLoRD across coverage points (the paper downsampled 60× datasets to 30×, yielding 20–60× comparisons). Run each caller with its documented defaults — Longshot with min_cov=6, max_cov=8000, min_mapq=20, min_alt_count=3, min_alt_frac=0.125, hap_assignment_qual=20.0 — on aligned BAM files, and apply a fixed GQ cutoff of 50 for short-read variant sets versus a variable GQ cutoff (matched to median read coverage) for Longshot. Parse each output VCF and quantify genotype concordance against the ground-truth variant VCF to compute precision and recall genome-wide. For a quick smoke test, the Longshot repository ships example_data: 3 contigs of 200 kb with 30× SimLoRD-simulated PacBio reads and 714 true variants in ground_truth_variants.vcf, runnable in roughly 5–10 seconds. A correct application reproduces the article's headline result: genome-wide SNV calling precision ≥0.9999 for both short-read and SMS read-based calling at all simulated coverages.

## Related tools

- **Longshot** (SMS-read-based SNV caller being benchmarked; called with longshot --bam ... --ref ... --out ... using default parameters) — https://github.com/pjedge/longshot
- **SimLoRD** (Simulator used to generate 30× coverage PacBio reads from the simulated diploid genome)
- **FreeBayes** (Short-read variant caller used as the benchmarking comparison baseline)
- **GATK HaplotyperCaller** (Short-read variant calling tool cited as the developed-for-short-read baseline class)
- **HapCUT2** (Read-based haplotype phasing algorithm used inside Longshot's iterative optimization during calling)

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Output VCF is produced and parses without error (execution on example_data takes ~5–10 seconds on a typical desktop machine)
- Genotype-level comparison against ground_truth_variants.vcf yields genome-wide precision ≥0.9999 for both short-read and SMS-based calling across 20–60× coverage
- Called variant count is comparable to the ground truth (714 true variants in the toy example)
- Coverage series is internally consistent: 60× datasets downsampled to half coverage give the 30× points used for caller comparison
- Longshot calls include phased haplotypes (HP/PS information) as expected from its haplotype-informed genotyping design

## Limitations

- Simulated read models may not fully capture real error profiles; simulation-based precision values may not transfer to real PacBio/ONT data
- Accurate calling of short indels with SMS reads remains challenging due to high insertion/deletion error rate; this skill covers SNVs only
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge even for long reads
- Real-data truth sets (GIAB, Platinum Genomes) are biased in favor of short-read technologies, so some Longshot 'false positives' may be false negatives in GIAB call sets

## Evidence

- [discussion] Simulated diploid genome with matched Illumina/PacBio design: [evidence span withheld — anchor status: not_found]
- [discussion] Downsampling for matched coverages: [evidence span withheld — anchor status: not_found]
- [methods] Longshot default parameters used in benchmarking: "Minimum coverage (of reads passing filters) to consider position as a potential SNV. [default: 6]"
- [readme] Example dataset description: "The directory example_data contains a simulated toy dataset that can be used to test out Longshot:"
- [abstract] High precision at all coverages: "Longshot achieves very high accuracy for SNV detection using whole-genome Pacific Biosciences data"
- [discussion] Truth-set bias caveat: "The GIAB and Platinum Genomes variant sets used to assess variant-calling accuracy in this study were generated using short-read datasets, are therefore biased in favor of short-read technologies"
