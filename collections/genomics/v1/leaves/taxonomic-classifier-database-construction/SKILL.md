---
name: taxonomic-classifier-database-construction
description: Use when benchmarking multiple taxonomic classifiers (e.g., Kraken2 vs
  Centrifuger vs minimap2-based strategies) on the same sample and you need reference-matched,
  version-pinned databases to make misclassification rates comparable.
when_to_use_negative:
- The analysis goal is ARG detection itself rather than host-identification benchmarking
  — use Argo with its SARG+/GTDB Zenodo database instead of rebuilding classifier
  databases from RefSeq
- The classifiers being compared intentionally use different reference databases (e.g.,
  GTDB vs RefSeq), where reference-matched construction would erase the effect under
  study
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Kraken2
  role: k-mer-based taxonomic classifier whose database is built via kraken2-build
    from the frozen RefSeq complete-genome set (v2.1.3)
- name: Centrifuger
  role: compressive classifier whose index is built with centrifuger-download/build
    from the same RefSeq set (v1.0.1)
- name: MetaMaps
  role: long-read metagenomic classifier whose database is constructed with buildDB.pl
    (v0.1); run with --maxmemory 20 at classification time
- name: minimap2
  role: provides the shared map-ont base-level alignment index (-x map-ont -d) used
    by minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR (v2.28)
- name: NanoSim
  role: trains read-simulation error models (read_analysis, metagenome mode) from
    Zymo D6331/D6300 reads so benchmark samples match ONT error profiles (v3.1.0)
- name: Argo
  role: the read-overlapping profiler whose minimap2+RO strategy is the top-performing
    strategy being benchmarked against per-read classifiers
  repo: github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:25ff5d7df9750466c18fa42fd089a2d650df577bdad9688954e6dcd57fa73bfa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
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
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# taxonomic-classifier-database-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs taxonomic classifier databases for benchmarking host identification of ARG-carrying long reads, rebuilding Kraken2, Centrifuger, MetaMaps, and a shared minimap2 map-ont index exclusively from NCBI RefSeq complete genomes (2024-05-10) so all methods are compared on identical reference content.

## When to use

Use when benchmarking multiple taxonomic classifiers (e.g., Kraken2 vs Centrifuger vs minimap2-based strategies) on the same sample and you need reference-matched, version-pinned databases to make misclassification rates comparable.

## When NOT to use

- The analysis goal is ARG detection itself rather than host-identification benchmarking — use Argo with its SARG+/GTDB Zenodo database instead of rebuilding classifier databases from RefSeq
- The classifiers being compared intentionally use different reference databases (e.g., GTDB vs RefSeq), where reference-matched construction would erase the effect under study

## Inputs

- NCBI RefSeq complete genome assemblies (snapshot dated 2024-05-10) for the 25 target primary/opportunistic pathogen species
- Real Nanopore reads from Zymo D6331 (high-quality) and Zymo D6300 (low-quality) mock communities for NanoSim model training
- Classifier tool versions: Kraken2 v2.1.3, Centrifuger v1.0.1, MetaMaps v0.1, minimap2 v2.28, MEGAN-LR v6.25.9

## Outputs

- Kraken2 database (kraken2-build)
- Centrifuger index (centrifuger-download/build)
- MetaMaps database (buildDB.pl)
- Shared minimap2 .mmi index built with -x map-ont -d
- NanoSim v3.1.0 error models for HQ (D6331) and LQ (D6300) metagenome simulation

## How to apply

First freeze the reference set — here NCBI RefSeq complete genomes downloaded as of 2024-05-10 — so every classifier is built from identical genomic content and version differences cannot confound the comparison. Build each database with the classifier's native tool: Kraken2 v2.1.3 via kraken2-build, Centrifuger v1.0.1 via centrifuger-download/build, MetaMaps v0.1 via buildDB.pl, and a single minimap2 v2.28 index built with '-x map-ont -d' that is shared by minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR (v6.25.9) read-overlapping strategies. In parallel, train NanoSim v3.1.0 error models in metagenome mode using real Zymo D6331 (HQ) and Zymo D6300 (LQ) reads so simulated benchmark samples reflect platform-specific error profiles. The rationale is that database scope and read-mapping index parameters directly drive false-positive host calls, so differences in misclassification (e.g., minimap2+RO at 0.19%/0.09% vs Kraken2 up to 16.63% for M. tuberculosis) can be attributed to assignment strategy rather than reference drift.

## Related tools

- **Kraken2** (k-mer-based taxonomic classifier whose database is built via kraken2-build from the frozen RefSeq complete-genome set (v2.1.3))
- **Centrifuger** (compressive classifier whose index is built with centrifuger-download/build from the same RefSeq set (v1.0.1))
- **MetaMaps** (long-read metagenomic classifier whose database is constructed with buildDB.pl (v0.1); run with --maxmemory 20 at classification time)
- **minimap2** (provides the shared map-ont base-level alignment index (-x map-ont -d) used by minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR (v2.28))
- **NanoSim** (trains read-simulation error models (read_analysis, metagenome mode) from Zymo D6331/D6300 reads so benchmark samples match ONT error profiles (v3.1.0))
- **Argo** (the read-overlapping profiler whose minimap2+RO strategy is the top-performing strategy being benchmarked against per-read classifiers) — github.com/xinehc/argo

## Examples

```
minimap2 -x map-ont -d database/genomes.mmi database/genomes.fa
```

## Evaluation signals

- After benchmarking, minimap2+RO should achieve overall misclassification rates of ~0.19% on the HQ sample and ~0.09% on the LQ sample of 1,000,000 even-abundance reads from 25 pathogens
- Kraken2 should show up to 16.63% misclassification for Mycobacterium tuberculosis in HQ, and Centrifuger should stay ≤1.32% overall in both samples
- All classifier databases must derive from the identical RefSeq complete-genome snapshot (2024-05-10); any version drift invalidates the comparison
- Misclassification errors should be decomposable into true-positive (misclassified among the 25 species), false-positive (outside the 25), and unclassified (no species call) categories summing to the total

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Insufficient coverage hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering; Argo marks species below 1× genome copy as unclassified
- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from the database

## Evidence

- [methods] Rebuild classifier databases exclusively from RefSeq complete genomes (2024-05-10): Kraken2 (v2.1.3, kraken2-build), Centrifuger (v1.0.1, centrifuger-download/build), MetaMaps (v0.1, buildDB.pl): [evidence span withheld — anchor status: not_found]
- [methods] a shared minimap2 (v2.28) index built with '-x map-ont -d' for minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR: [evidence span withheld — anchor status: not_found]
