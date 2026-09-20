---
name: misclassification-rate-quantification
description: Use when comparing host-identification accuracy of taxonomic classifiers
  (e.g., Kraken2, Centrifuger, MetaMaps, MEGAN-LR, minimap2 variants) on ARG-containing
  long reads in simulated metagenomes with known ground truth.
when_to_use_negative:
- Reads have no simulated or known ground-truth host labels — misclassification rate
  cannot be computed without a species-level truth set
- The goal is de novo identification of ARG genomic contexts and hosts in understudied
  environments, where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2
  + GTDB-Tk) methods are better suited
- Evaluating short-read classifiers — this benchmark is specific to ARG-carrying long
  (ONT-style) reads and NanoSim error models
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0798
tools:
- name: NanoSim
  role: Simulates HQ and LQ metagenomes with known species composition to serve as
    ground truth for misclassification scoring
- name: DIAMOND
  role: Frameshift-aware blastx extraction of ARG-containing reads against SARG+ prior
    to classification
- name: Kraken2
  role: Per-read taxonomic classifier benchmarked as a baseline host-identification
    strategy
- name: Centrifuger
  role: Per-read taxonomic classifier benchmarked as a baseline host-identification
    strategy
- name: MetaMaps
  role: Long-read taxonomic classifier benchmarked as a baseline host-identification
    strategy
- name: MEGAN-LR
  role: LCA-based long-read classifier benchmarked on the shared minimap2 index
- name: minimap2
  role: Provides base-level ONT alignments against RefSeq genomes for the minimap2+BH,
    minimap2+EM, minimap2+RO, and MEGAN-LR strategies
- name: Argo
  role: Read-overlapping strategy (minimap2+RO) whose misclassification rates are
    the reference target of the benchmark
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:c9fe2af414afffdcaa9d19de729fed7f0e6e58d0dbd35f880e7d4c3aaea04323
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - Centrifuger
  - DIAMOND
  - Kraken2
  - MEGAN-LR
  - MetaMaps
  - NanoSim
  - minimap2
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# misclassification-rate-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-species and overall misclassification rates of taxonomic assignment strategies applied to ARG-carrying long reads, as used in the Argo benchmark where minimap2+RO achieved 0.19% (HQ) and 0.09% (LQ) overall misclassification versus up to 16.63% for Kraken2 (M. tuberculosis, HQ).

## When to use

Use when comparing host-identification accuracy of taxonomic classifiers (e.g., Kraken2, Centrifuger, MetaMaps, MEGAN-LR, minimap2 variants) on ARG-containing long reads in simulated metagenomes with known ground truth.

## When NOT to use

- Reads have no simulated or known ground-truth host labels — misclassification rate cannot be computed without a species-level truth set
- The goal is de novo identification of ARG genomic contexts and hosts in understudied environments, where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods are better suited
- Evaluating short-read classifiers — this benchmark is specific to ARG-carrying long (ONT-style) reads and NanoSim error models

## Inputs

- Simulated ONT long-read metagenomes (NanoSim HQ and LQ, 1,000,000 reads each from 25 RefSeq complete pathogen genomes)
- Reference genomes: NCBI RefSeq complete genomes as of 2024-05-10
- ARG reference database: SARG+ (sarg.fa)
- Classifier databases built from the same RefSeq genomes (Kraken2, Centrifuger, MetaMaps, minimap2 .mmi index)
- List of the 25 known ground-truth species

## Outputs

- Per-species misclassification rates for each taxonomic strategy
- Overall misclassification rate per strategy per sample (HQ and LQ)
- Error decomposition: true-positive errors (within the 25 species), false-positive errors (outside the 25), and unclassified fractions

## How to apply

Simulate ground-truth metagenomes with NanoSim (v3.1.0) in metagenome mode — an HQ model from Zymo D6331 reads and an LQ model from Zymo D6300 reads — each with 1,000,000 reads from 25 primary/opportunistic pathogens at even abundance, using NCBI RefSeq complete genomes (2024-05-10). Extract ARG-containing reads via DIAMOND frameshift-aware blastx against SARG+ with '--evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hps 0' equivalents, then classify them with each method built exclusively from the same RefSeq complete genomes. Compute misclassification rate per species and overall, splitting errors into true-positive (misassigned to one of the 25 species), false-positive (assigned outside the 25), and unclassified (no species-level call). Validate that minimap2+RO reaches ~0.19% (HQ) and ~0.09% (LQ) overall, Kraken2 shows up to 16.63% for M. tuberculosis in HQ, and Centrifuger stays ≤1.32% overall in both samples.

## Related tools

- **NanoSim** (Simulates HQ and LQ metagenomes with known species composition to serve as ground truth for misclassification scoring)
- **DIAMOND** (Frameshift-aware blastx extraction of ARG-containing reads against SARG+ prior to classification)
- **Kraken2** (Per-read taxonomic classifier benchmarked as a baseline host-identification strategy)
- **Centrifuger** (Per-read taxonomic classifier benchmarked as a baseline host-identification strategy)
- **MetaMaps** (Long-read taxonomic classifier benchmarked as a baseline host-identification strategy)
- **MEGAN-LR** (LCA-based long-read classifier benchmarked on the shared minimap2 index)
- **minimap2** (Provides base-level ONT alignments against RefSeq genomes for the minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR strategies)
- **Argo** (Read-overlapping strategy (minimap2+RO) whose misclassification rates are the reference target of the benchmark) — https://github.com/xinehc/argo

## Examples

```
diamond blastx --db database/sarg --query arg_reads.fa --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0
```

## Evaluation signals

- Overall misclassification rate for minimap2+RO is approximately 0.19% in the HQ sample and 0.09% in the LQ sample
- Kraken2 misclassification reaches up to 16.63% for M. tuberculosis in the HQ sample
- Centrifuger overall misclassification stays below 1.32% in both HQ and LQ samples
- Per-strategy errors decompose cleanly into true-positive (species among the 25), false-positive (outside the 25), and unclassified categories that sum to 100% of ARG-containing reads
- All classifier databases are rebuilt from the identical RefSeq complete genome snapshot (2024-05-10) so differences reflect strategy, not reference composition

## Limitations

- Results are contingent on the simulated NanoSim error models and even-abundance 25-species community; real metagenomes with uneven abundances or unseen species may shift rates
- Reference-based classification can misclassify novel plasmid syntenies or species absent from the reference database
- Unclassified ARG reads (no species-level call) count as errors in this metric, penalizing conservative classifiers

## Evidence
