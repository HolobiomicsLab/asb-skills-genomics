---
name: nanopore-read-simulation-error-modeling
description: Use when benchmarking long-read taxonomic classifiers or ARG host-assignment
  pipelines and realistic simulated ONT metagenomes with controlled error profiles
  (HQ vs LQ) and known ground-truth species composition are needed.
when_to_use_negative:
- When the goal is to characterize real environmental samples rather than create controlled
  benchmarks with known ground truth — simulation cannot capture real strain diversity
  or plasmid-mediated HGT patterns
- When evaluating short-read (Illumina) pipelines, since the NanoSim models here are
  trained on ONT error profiles
- When ARGs of unseen species or novel genomic contexts need assessment — the simulation
  is restricted to the 25 RefSeq-complete reference genomes used to build the classifier
  databases
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: NanoSim
  role: Trains HQ/LQ error models from Zymo mock reads and simulates 1,000,000-read
    metagenomes in metagenome mode
- name: DIAMOND
  role: Frameshift-aware blastx (--evalue 1e-15 --range-culling --frameshift 15) extracts
    ARG-containing reads from the simulated set before host classification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:02f9a02cd6e5c3edd224ea69e18e349f1e3bb4f7adfc812a37ba6d2b44e72ce3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  tools:
  - Centrifuger
  - DIAMOND
  - Kraken2
  - MEGAN-LR
  - MetaMaps
  - NanoSim
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Nanopore Read Simulation Error Modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train platform-specific NanoSim error models from real Zymo mock-community nanopore reads (HQ from Zymo D6331, LQ from Zymo D6300) and use them to simulate large metagenomic read sets for benchmarking taxonomic host-identification strategies on ARG-carrying long reads.

## When to use

Use when benchmarking long-read taxonomic classifiers or ARG host-assignment pipelines and realistic simulated ONT metagenomes with controlled error profiles (HQ vs LQ) and known ground-truth species composition are needed.

## When NOT to use

- When the goal is to characterize real environmental samples rather than create controlled benchmarks with known ground truth — simulation cannot capture real strain diversity or plasmid-mediated HGT patterns
- When evaluating short-read (Illumina) pipelines, since the NanoSim models here are trained on ONT error profiles
- When ARGs of unseen species or novel genomic contexts need assessment — the simulation is restricted to the 25 RefSeq-complete reference genomes used to build the classifier databases

## Inputs

- Real ONT nanopore reads from Zymo D6331 mock community (for HQ error model training)
- Real ONT nanopore reads from Zymo D6300 mock community (for LQ error model training)
- NCBI RefSeq complete genomes (2024-05-10) of the 25 target pathogen species
- NanoSim v3.1.0

## Outputs

- Trained NanoSim HQ and LQ error models (model files from 'read_analysis')
- Simulated metagenomic FASTQ read sets of 1,000,000 reads with even 25-species abundance and ground-truth species labels per read

## How to apply

Train two NanoSim (v3.1.0) error models by running 'read_analysis' in metagenome mode: use Zymo D6331 nanopore reads for the high-quality (HQ) model and Zymo D6300 reads for the low-quality (LQ) model. Then run NanoSim's 'simulator' script in metagenome mode to generate samples of 1,000,000 reads each, drawn at even taxonomic abundance from 25 primary/opportunistic pathogens whose reference genomes are NCBI RefSeq complete genomes (as of 2024-05-10). The simulated FASTQ reads retain known species-of-origin labels, enabling per-species and overall misclassification-rate computation (split into true-positive species errors, false positives outside the 25 species, and unclassified) when the reads are classified by tools such as Kraken2, Centrifuger, MetaMaps, MEGAN-LR, or minimap2-based strategies. This design lets error rate (HQ vs LQ models) be varied independently of taxonomic composition to isolate classifier robustness to sequencing errors.

## Related tools

- **NanoSim** (Trains HQ/LQ error models from Zymo mock reads and simulates 1,000,000-read metagenomes in metagenome mode)
- **DIAMOND** (Frameshift-aware blastx (--evalue 1e-15 --range-culling --frameshift 15) extracts ARG-containing reads from the simulated set before host classification)

## Examples

```
NanoSim read_analysis metagenome --ont --fastq zymo_d6331_reads.fastq; simulator metagenome --model HQ_model --number_of_reads 1000000
```

## Evaluation signals

- The trained models should reproduce the intended quality split: HQ from Zymo D6331 reads and LQ from Zymo D6300 reads, each built via 'read_analysis' in metagenome mode
- Each simulated sample should contain exactly 1,000,000 reads at even taxonomic abundance across the 25 pathogen species
- Downstream classification of simulated reads with minimap2+RO (Argo's read-overlapping strategy) should yield overall misclassification rates of ~0.19% (HQ) and ~0.09% (LQ)
- Kraken2 should show worst-case per-species error up to 16.63% for M. tuberculosis in HQ, and Centrifuger should stay ≤1.32% overall in both samples
- Misclassification accounting should partition errors into misclassified species among the 25, false positives outside the 25, and unclassified reads summing to 100% per species

## Limitations

- Simulated communities assume even abundance and RefSeq-complete references, so classifier performance on uneven or understudied communities is not captured
- Argo's host-identification gains under this benchmark do not extend to unseen species or novel plasmid syntenies, since profiling is inherently reference-based
- Argo considers ARGs of species below 1× genome copy as 'unclassified', and insufficient coverage hinders read overlap and clustering — low-coverage simulated conditions may inflate unclassified rates

## Evidence

- [methods] Train two NanoSim (v3.1.0) error models with 'read_analysis' in metagenome mode, using Zymo D6331 reads for the high-quality (HQ) model and Zymo D6300 reads for the low-quality (LQ) model.: [evidence span withheld — anchor status: not_found]
- [methods] Simulate samples HQ and LQ with NanoSim's 'simulator' script in metagenome mode, each containing 1,000,000 reads from the 25 primary/opportunistic pathogens at even taxonomic abundance: [evidence span withheld — anchor status: not_found]
