---
name: per-read-taxonomic-benchmarking
description: Use when comparing taxonomic classifiers for host identification of ARG-carrying
  long reads, or when quantifying misclassification rates (true-positive, false-positive,
  unclassified) of metagenomic profilers against known ground truth.
when_to_use_negative:
- The goal is de novo identification of ARG genomic contexts and hosts in novel, understudied
  environments where reference-based classification is unreliable (Argo is inherently
  reference-based and may misclassify unseen species or novel plasmid syntenies)
- Reads contain point-mutation-derived ARGs that cannot be distinguished from sequencing
  errors, since these are excluded from the SARG+ database
- Only one classifier is being evaluated without a shared reference catalog — the
  misclassification benchmark requires a like-for-like comparison across strategies
  on identical simulated ground truth
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: NanoSim
  role: Trains HQ/LQ error models and simulates metagenome-mode long-read samples
    with known ground-truth species composition
- name: Kraken2
  role: Per-read k-mer taxonomic classifier benchmarked on the simulated ARG-carrying
    reads
- name: Centrifuger
  role: Per-read taxonomic classifier benchmarked on the simulated ARG-carrying reads
- name: MetaMaps
  role: Long-read taxonomic classifier benchmarked on the simulated ARG-carrying reads
- name: MEGAN-LR
  role: LCA-based long-read taxonomic classifier benchmarked using the shared minimap2
    alignment index
- name: minimap2
  role: Builds the shared '-x map-ont -d' alignment index used by minimap2+BH, minimap2+EM,
    minimap2+RO, and MEGAN-LR strategies
- name: DIAMOND
  role: Frameshift-aware blastx extraction of ARG-containing reads against SARG+ prior
    to classification
- name: Argo
  role: Read-overlapping strategy (minimap2+RO) whose cluster-level species assignment
    is benchmarked against per-read classifiers
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:7549c46cc631ae8dda8112e97ab28744326992fe431273780f7abd7adc457f54
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# per-read-taxonomic-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A simulation-based benchmark that quantifies how accurately different taxonomic assignment strategies (e.g., minimap2+RO, Kraken2, Centrifuger, MetaMaps, MEGAN-LR) identify the true species hosts of ARG-carrying long reads, using per-species and overall misclassification rates as the metric.

## When to use

Use when comparing taxonomic classifiers for host identification of ARG-carrying long reads, or when quantifying misclassification rates (true-positive, false-positive, unclassified) of metagenomic profilers against known ground truth.

## When NOT to use

- The goal is de novo identification of ARG genomic contexts and hosts in novel, understudied environments where reference-based classification is unreliable (Argo is inherently reference-based and may misclassify unseen species or novel plasmid syntenies)
- Reads contain point-mutation-derived ARGs that cannot be distinguished from sequencing errors, since these are excluded from the SARG+ database
- Only one classifier is being evaluated without a shared reference catalog — the misclassification benchmark requires a like-for-like comparison across strategies on identical simulated ground truth

## Inputs

- Zymo D6331 (HQ) and Zymo D6300 (LQ) Nanopore read sets for NanoSim error-model training
- NCBI RefSeq complete genomes for 25 primary/opportunistic pathogens (downloaded 2024-05-10)
- SARG+ ARG reference database (sarg.fa)
- Raw simulated long-read metagenome samples (1,000,000 reads each, HQ and LQ)

## Outputs

- Per-species and overall misclassification rates for each taxonomic assignment strategy
- Error decomposition per strategy: true-positive (misclassified among the 25), false-positive (outside the 25), and unclassified (no species-level call)
- Ranked comparison table of strategies (minimap2+RO, Kraken2, Centrifuger, MetaMaps, MEGAN-LR, minimap2+BH, minimap2+EM)

## How to apply

Train two NanoSim (v3.1.0) error models in metagenome mode — Zymo D6331 reads for the high-quality (HQ) model and Zymo D6300 reads for the low-quality (LQ) model — then simulate 1,000,000 reads from 25 primary/opportunistic pathogens at even taxonomic abundance using NanoSim's 'simulator' script, with references from NCBI RefSeq complete genomes (2024-05-10). Rebuild all classifier databases exclusively from the same RefSeq complete genomes so the comparison is fair: Kraken2 (v2.1.3, kraken2-build), Centrifuger (v1.0.1, centrifuger-download/build), MetaMaps (v0.1, buildDB.pl), and a shared minimap2 (v2.28) index built with '-x map-ont -d' for minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR (v6.25.9). Extract ARG-containing reads with DIAMOND frameshift-aware blastx against SARG+ ('--evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0') and classify those reads with each method (run MetaMaps with '--maxmemory 20'). Finally, compute misclassification rate per species and overall for each strategy, splitting errors into true-positive (misclassified among the 25 species), false-positive (outside the 25), and unclassified (no species-level call) categories. The rationale is that per-read classifiers mislabel reads at short lengths, whereas Argo's read-overlapping strategy assigns labels at the cluster level, so only a like-for-like simulation with a single reference catalog exposes the true error profiles.

## Related tools

- **NanoSim** (Trains HQ/LQ error models and simulates metagenome-mode long-read samples with known ground-truth species composition)
- **Kraken2** (Per-read k-mer taxonomic classifier benchmarked on the simulated ARG-carrying reads)
- **Centrifuger** (Per-read taxonomic classifier benchmarked on the simulated ARG-carrying reads)
- **MetaMaps** (Long-read taxonomic classifier benchmarked on the simulated ARG-carrying reads)
- **MEGAN-LR** (LCA-based long-read taxonomic classifier benchmarked using the shared minimap2 alignment index)
- **minimap2** (Builds the shared '-x map-ont -d' alignment index used by minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR strategies)
- **DIAMOND** (Frameshift-aware blastx extraction of ARG-containing reads against SARG+ prior to classification)
- **Argo** (Read-overlapping strategy (minimap2+RO) whose cluster-level species assignment is benchmarked against per-read classifiers) — https://github.com/xinehc/argo

## Examples

```
diamond blastx --db sarg.fa -q sim_reads.fq --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0 -o arg_reads.tsv
```

## Evaluation signals

- minimap2+RO should reproduce overall misclassification rates of approximately 0.19% (HQ) and 0.09% (LQ)
- Kraken2 should show a worst-case per-species misclassification of up to 16.63% for M. tuberculosis in the HQ sample
- Centrifuger overall misclassification should remain ≤1.32% in both HQ and LQ samples
- Every strategy's error counts must sum to the classified reads, partitioned into true-positive, false-positive, and unclassified categories
- All classifier databases must be built from the identical RefSeq complete genome catalog (2024-05-10); any version skew invalidates the comparison

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- ARGs prone to horizontal gene transfer may appear in multiple genetic locations (e.g., plasmids or chromosomes) across different species, making per-read classification difficult
- Results are grounded in simulated even-abundance communities with RefSeq-complete references; real environmental samples with uneven abundance and incomplete references may behave differently

## Evidence
