---
name: structural-variant-simulation-parameter-configuration
description: Use when benchmarking a deletion SV caller needs known-truth simulated
  long-read data across homozygous/heterozygous groups and multiple coverage depths,
  requiring SURVIVOR parameter files plus PaSS PacBio read simulation and NGMLR alignment.
when_to_use_negative:
- The goal is real-sample validation rather than a controlled truth-set benchmark
  — the article's real data (GIAB Ashkenazim Trio HG002/HG003/HG004, NIST RM 8392,
  Arabidopsis thaliana F1 hybrid) do not require simulated parameter files
- The SV caller under test must handle insertions or base substitutions — the article's
  simulation setup covers deletion variants only
- Only a single coverage depth or a single zygosity condition is needed, in which
  case the 4-depth x 2-group factorial simulation design is over-specified
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SURVIVOR
  role: Generates the simulation parameter file specifying type, quantity and length
    of simulated deletion variants
- name: PaSS
  role: Simulates PacBio SMRT long reads at 90x/65x/45x/20x coverage with 20k bp average
    read length and 15% error rate
- name: NGMLR
  role: Aligns the simulated FASTQ reads to the reference genome to obtain BAM files
- name: DASV
  role: Caller under test, benchmarked on the simulated BAM files for deletion calling
    performance
  repo: https://github.com/deconvolution-w/DASV
- name: cuteSV
  role: Baseline deletion caller run on the same simulated BAM files under identical
    conditions
- name: SVIM
  role: Baseline deletion caller run on the same simulated BAM files under identical
    conditions
- name: Sniffles
  role: Baseline deletion caller run on the same simulated BAM files under identical
    conditions
- name: PBSV
  role: Baseline deletion caller run on the same simulated BAM files under identical
    conditions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:7b81e4cd88c2771babc50d60f251a11c3a21cea3a35dafea9e356b5a18ee6297
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
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# structural-variant-simulation-parameter-configuration

## Summary

Configure and run a long-read structural-variant simulation benchmark for deletion calling: generate SURVIVOR simulation parameter files specifying homozygous/heterozygous deletion lengths (50-1000 bp and 1000-5000 bp), simulate PacBio SMRT reads with PaSS at four coverage depths (90x, 65x, 45x, 20x), and align them with NGMLR to produce BAM inputs for benchmarking DASV against cuteSV, SVIM, Sniffles and PBSV.

## When to use

Use when benchmarking a deletion SV caller needs known-truth simulated long-read data across homozygous/heterozygous groups and multiple coverage depths, requiring SURVIVOR parameter files plus PaSS PacBio read simulation and NGMLR alignment.

## When NOT to use

- The goal is real-sample validation rather than a controlled truth-set benchmark — the article's real data (GIAB Ashkenazim Trio HG002/HG003/HG004, NIST RM 8392, Arabidopsis thaliana F1 hybrid) do not require simulated parameter files
- The SV caller under test must handle insertions or base substitutions — the article's simulation setup covers deletion variants only
- Only a single coverage depth or a single zygosity condition is needed, in which case the 4-depth x 2-group factorial simulation design is over-specified

## Inputs

- Reference genome FASTA for read simulation and alignment
- SURVIVOR simulation parameter file specifying deletion type, quantity and length distribution (50-1000 bp and 1000-5000 bp, homozygous and heterozygous groups)
- PaSS simulation settings: average read length 20k bp, average calling error rate 15%, coverage depths 90x/65x/45x/20x

## Outputs

- Eight simulated FASTQ read sets: Sim_A{90X,65X,45X,20X} and Sim_B{90X,65X,45X,20X}
- NGMLR-aligned BAM files for each simulated dataset
- Known-truth deletion variant call sets (from the SURVIVOR simulation parameters)
- Per-tool precision, recall and F1 metrics per coverage depth and deletion length (e.g. simulated_deletion_calling_results.csv)

## How to apply

First, generate a simulation parameter file with SURVIVOR that specifies the type, quantity and length of simulated deletion variants, spanning deletion lengths between 50-1000 bp and 1000-5000 bp for both homozygous and heterozygous groups. Second, simulate PacBio SMRT long reads with PaSS at coverage depths of 90x, 65x, 45x and 20x per group, using an average read length of 20k bp and an average calling error rate of 15%, yielding the eight datasets Sim_A{90X,65X,45X,20X} (homozygous) and Sim_B{90X,65X,45X,20X} (heterozygous) as FASTQ files. Third, align the simulated FASTQ reads to the reference genome with NGMLR to obtain BAM files for each dataset, providing identical inputs to the caller under test and the baselines. Finally, compare each tool's deletion calls against the known simulated truth set, computing precision, recall and F1 score per coverage depth and deletion length to judge whether the benchmark reproduces the reported advantage of DASV on high-coverage data (high recall and F1, with at least 82.16% recall on the 20x low-coverage condition).

## Related tools

- **SURVIVOR** (Generates the simulation parameter file specifying type, quantity and length of simulated deletion variants)
- **PaSS** (Simulates PacBio SMRT long reads at 90x/65x/45x/20x coverage with 20k bp average read length and 15% error rate)
- **NGMLR** (Aligns the simulated FASTQ reads to the reference genome to obtain BAM files)
- **DASV** (Caller under test, benchmarked on the simulated BAM files for deletion calling performance) — https://github.com/deconvolution-w/DASV
- **cuteSV** (Baseline deletion caller run on the same simulated BAM files under identical conditions)
- **SVIM** (Baseline deletion caller run on the same simulated BAM files under identical conditions)
- **Sniffles** (Baseline deletion caller run on the same simulated BAM files under identical conditions)
- **PBSV** (Baseline deletion caller run on the same simulated BAM files under identical conditions)

## Evaluation signals

- Eight distinct simulated FASTQ datasets are produced (Sim_A and Sim_B at 90x, 65x, 45x and 20x), covering homozygous and heterozygous groups
- Simulated deletion lengths span 50-1000 bp and 1000-5000 bp; metrics must be reported per length bin
- All five callers (DASV, cuteSV, SVIM, Sniffles, PBSV) run on identical BAM files derived from the same simulated reads
- Precision, recall and F1 per tool per coverage depth match the expected trend: DASV attains the highest F1 across depths and lengths, with recall of at least 82.16% on the 20x low-coverage condition
- Simulation settings reproduce the article's parameters: 20k bp average read length and 15% average calling error rate

## Limitations

- Simulation covers deletion variants only; the article states it 'currently focuses exclusively on experiments related to deletion variants calling'
- Simulated reads carry a fixed 15% average calling error rate and 20k bp average read length, so results may not transfer to other error profiles or read lengths
- No versions are stated for the compared tools (cuteSV, SVIM, Sniffles, PBSV), which may affect reproducibility of the baseline comparison

## Evidence

- [methods] Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups: "Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups"
- [intro] we compared DASV with the advanced third-generation variation detection tools: "we compared DASV with the advanced third-generation variation detection tools, SVIM, Sniffles, PBSV and cuteSV. The results showed that DASV has notable advantages for data with high coverage depth"
- [methods] Sequenced fragments with an alignment quality number <20 were removed: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [readme] Our method achieved the best results on all datasets.: "Our method achieved the best results on all datasets. And we have submitted the manuscript to [briefings in bioinformatics](https://academic.oup.com/bib/)."
