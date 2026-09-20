---
name: long-read-sequencing-read-simulation
description: Use when benchmarking third-generation SV/deletion callers needs ground-truth
  data at multiple coverage depths (e.g. 90x/65x/45x/20x) and specified variant length
  ranges, so recall/precision/F1 can be computed against a known truth set.
when_to_use_negative:
- When the evaluation target is real patient or GIAB samples (e.g. Ashkenazim Trio
  HG002/HG003/HG004) where variant truth comes from a benchmark set like NIST Reference
  Material 8392 rather than simulation
- When the SV type of interest is not deletions — this simulation design was scoped
  to deletion variants only (insertion calling is stated as future work)
- When short-read (Illumina) data must be simulated; the parameters here (20k bp average
  read length, 15% error rate) are specific to PacBio SMRT long reads
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: SURVIVOR
  role: Generates the simulation parameter file specifying the type, quantity and
    length of simulated deletion variants (50-1000 bp and 1000-5000 bp, homozygous
    and heterozygous)
- name: PaSS
  role: Simulates PacBio SMRT long reads from the parameterized variant set at 90x/65x/45x/20x
    coverage, 20k bp average read length, 15% average calling error rate, outputting
    FASTQ
- name: NGMLR
  role: Aligns the simulated FASTQ reads to the reference genome to obtain BAM files
    consumed by the SV callers
- name: DASV
  role: Downstream deletion caller benchmarked on the simulated datasets; source code
    repository referenced by the article
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:e823aadd9575528d97cff43d9d00c4f1b437129eb6d712462ef44bda7242fe69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - NGMLR
  - PBSV
  - PaSS
  - SURVIVOR
  - SVIM
  - Sniffles
  - cuteSV
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# long-read-sequencing-read-simulation

## Summary

Generates simulated PacBio SMRT long-read datasets at controlled coverage depths for benchmarking deletion SV callers, using SURVIVOR to define the variant truth set and PaSS to simulate reads. It grounds the DASV-vs-baseline comparison on Sim_A/Sim_B data where ground truth is known by construction.

## When to use

Use when benchmarking third-generation SV/deletion callers needs ground-truth data at multiple coverage depths (e.g. 90x/65x/45x/20x) and specified variant length ranges, so recall/precision/F1 can be computed against a known truth set.

## When NOT to use

- When the evaluation target is real patient or GIAB samples (e.g. Ashkenazim Trio HG002/HG003/HG004) where variant truth comes from a benchmark set like NIST Reference Material 8392 rather than simulation
- When the SV type of interest is not deletions — this simulation design was scoped to deletion variants only (insertion calling is stated as future work)
- When short-read (Illumina) data must be simulated; the parameters here (20k bp average read length, 15% error rate) are specific to PacBio SMRT long reads

## Inputs

- Reference genome (FASTA) to simulate reads against
- SURVIVOR simulation parameter file specifying deletion type, quantity and length distribution (50-1000 bp and 1000-5000 bp; homozygous and heterozygous groups)
- Simulation settings: coverage depths 90x, 65x, 45x, 20x; average read length 20k bp; average calling error rate 15%

## Outputs

- Eight simulated PacBio SMRT long-read FASTQ files: Sim_A{90X,65X,45X,20X} (homozygous) and Sim_B{90X,65X,45X,20X} (heterozygous)
- Simulated truth set of deletion variants (known ground truth per dataset) for computing precision, recall and F1

## How to apply

First write a SURVIVOR simulation parameter file specifying the type, quantity and length of deletion variants — covering deletion lengths between 50-1000 bp and 1000-5000 bp for both homozygous and heterozygous groups. Then simulate PacBio SMRT long reads with PaSS at coverage depths of 90x, 65x, 45x and 20x for each group, using an average read length of 20k bp and an average calling error rate of 15%, yielding eight FASTQ datasets (Sim_A{90X,65X,45X,20X} and Sim_B{90X,65X,45X,20X}, Sim_A being homozygous and Sim_B heterozygous). Retain the simulated truth VCF so downstream callers (DASV, cuteSV, SVIM, Sniffles, PBSV) can be scored per coverage depth and deletion length, mirroring Supplementary Tables 7(a&b) and 8(a&b). The rationale is that only simulated data provide exact deletion coordinates, enabling per-tool recall checks such as the >=82.16% recall DASV reports at 20x.

## Related tools

- **SURVIVOR** (Generates the simulation parameter file specifying the type, quantity and length of simulated deletion variants (50-1000 bp and 1000-5000 bp, homozygous and heterozygous))
- **PaSS** (Simulates PacBio SMRT long reads from the parameterized variant set at 90x/65x/45x/20x coverage, 20k bp average read length, 15% average calling error rate, outputting FASTQ)
- **NGMLR** (Aligns the simulated FASTQ reads to the reference genome to obtain BAM files consumed by the SV callers)
- **DASV** (Downstream deletion caller benchmarked on the simulated datasets; source code repository referenced by the article) — https://github.com/deconvolution-w/DASV

## Evaluation signals

- Eight FASTQ files are produced: Sim_A and Sim_B each at 90x, 65x, 45x and 20x coverage (homozygous vs heterozygous variant groups)
- Simulated deletions span both the 50-1000 bp and 1000-5000 bp length ranges in each group
- Per-tool precision, recall and F1 computed against the simulated truth set match the structure of Supplementary Tables 7(a&b) and 8(a&b)
- DASV attains the highest F1 of all five tools across coverage depths, including recall >= 82.16% at the 20x low-coverage condition
- Read simulation settings reproduce the stated profile: ~20k bp average read length and ~15% average calling error rate

## Limitations

- Simulation covers deletion variants only; insertion and base substitution detection is stated as future work, so results do not transfer to other SV classes
- Realistic error profile depends on PaSS parameters (15% calling error, 20k bp reads); deviations may change caller rankings
- Low-coverage (20x) conditions are the hardest; conclusions are strongest for high-coverage data where DASV showed its most notable advantages

## Evidence

- [methods] Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups: "Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups"
- [intro] The results showed that DASV has notable advantages for data with high coverage depth: "The results showed that DASV has notable advantages for data with high coverage depth"
- [discussion] This study currently focuses exclusively on experiments related to deletion variants calling: "This study currently focuses exclusively on experiments related to deletion variants calling"
- [discussion] our proposed method, DASV, consistently delivered impressive results across various sequencing depths: "our proposed method, DASV, consistently delivered impressive results across various sequencing depths"
