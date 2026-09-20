---
name: arg-abundance-benchmark-evaluation
description: Use when you need to quantify how detection coverage thresholds differ
  between read-based, assembly-based, and binning-based ARG profiling pipelines on
  simulated long-read metagenomes of known composition.
when_to_use_negative:
- The input is a real environmental or clinical metagenome with unknown true ARG composition
  — no ground truth exists to compute precision/recall/F1 against
- The goal is de novo identification of ARG genomic contexts or hosts in understudied
  environments, where assembly- and binning-based approaches are the article's recommended
  choice
- The question concerns point-mutation-derived ARGs (e.g., SNP-driven resistance),
  which Argo's reference-based database explicitly excludes
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_0091
tools:
- name: Argo
  role: Read-based ARG profiler evaluated at low coverage using '--plasmid -z 0';
    produces cpg-normalized ARG abundance estimates per species
  repo: https://github.com/xinehc/argo
- name: metaFlye
  role: 'Assembly-based branch: assembles each downsampled set into contigs with ''--nano-raw
    --meta'' before Kraken2 classification'
- name: Kraken2
  role: Classifies metaFlye contigs for assembly-based ARG host assignment
- name: SemiBin2
  role: 'Binning branch: bins contigs into MAGs with ''single_easy_bin --sequencing-type=long_read'''
- name: GTDB-Tk
  role: Labels MAGs with 'classify_wf' for binning-based host assignment
- name: DIAMOND
  role: Builds the ground truth by blastx mapping reference genomes to SARG+ with
    stringent cutoffs (e-value 1e-15, id 90, subject-cover 90)
- name: NanoSim
  role: Simulates Nanopore metagenomic reads (HQ/LQ profiles) and enables downsampling
    to target coverages
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:9218a9a99a7be447052965fc07d2c4e8a1d600c3567278677b69e9a0ca94a569
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
  - DIAMOND
  - GTDB-Tk
  - Kraken2
  - NanoSim
  - R
  - SemiBin2
  - metaFlye
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# arg-abundance-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for comparing read-based (Argo), assembly-based (metaFlye + Kraken2), and binning-based (SemiBin2 + GTDB-Tk) ARG profiling pipelines against a ground-truth expectation derived by DIAMOND mapping of reference genomes to SARG+, scoring type- and subtype-level accuracy with precision, recall, F1, Pearson correlation, and L1/L2 distances across simulated coverages from 1× to 32×.

## When to use

Use when you need to quantify how detection coverage thresholds differ between read-based, assembly-based, and binning-based ARG profiling pipelines on simulated long-read metagenomes of known composition.

## When NOT to use

- The input is a real environmental or clinical metagenome with unknown true ARG composition — no ground truth exists to compute precision/recall/F1 against
- The goal is de novo identification of ARG genomic contexts or hosts in understudied environments, where assembly- and binning-based approaches are the article's recommended choice
- The question concerns point-mutation-derived ARGs (e.g., SNP-driven resistance), which Argo's reference-based database explicitly excludes

## Inputs

- Simulated Nanopore metagenomic FASTQ/FASTA samples (NanoSim v3.1.0 metagenome mode, HQ and LQ error profiles)
- Downsampled read sets at 1×, 2×, 4×, 8×, 16×, and 32× expected coverage
- 25 reference pathogen genomes (FASTA) for ground-truth ARG expectation
- SARG+ ARG reference database (sarg.fa from xinehc/sarg-curation)
- GTDB reference database (for GTDB-Tk and Argo species assignment)

## Outputs

- Expected ARG abundance table (ARG occurrences per genome from DIAMOND blastx mapping)
- Per-method, per-coverage predicted ARG abundance tables (Argo output as cpg-normalized copies per genome)
- Evaluation metrics table: precision, recall, F1-score, Pearson correlation, and L1/L2 distances at type and subtype levels per coverage point

## How to apply

Simulate Nanopore metagenomic samples with NanoSim v3.1.0 in metagenome mode (e.g., HQ q=19.04, l=5028 and LQ q=13.19, l=10,365, 1,000,000 reads each from 25 primary/opportunistic pathogens at even abundance), then downsample to expected coverages of 1×, 2×, 4×, 8×, 16×, and 32×. Build the ground truth by mapping the 25 reference genomes to SARG+ with DIAMOND blastx (--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0) and counting ARG occurrences. Run each profiling method on every downsampled set: read-based profiling with Argo v0.1.0 using '--plasmid -z 0' (so plasmids and species below 1× genome copy are still classified), assembly-based profiling via metaFlye v2.9.3 ('--nano-raw --meta') plus Kraken2 v2.1.3 contig classification, and binning-based profiling via SemiBin2 v2.1.0 ('single_easy_bin --sequencing-type=long_read') plus GTDB-Tk v2.4.0 ('classify_wf'). Evaluate all three against expected ARG abundances at type and subtype levels using precision, recall, F1-score, Pearson correlation coefficient, and L1/L2 distances per coverage point. The validated expected outcome: Argo reaches type-level F1 of 0.811–0.935 with precision 0.977–0.992 at 1–2×, while assembly-based profiling needs ≥8× and binning-based >32× coverage to exceed 0.9 F1.

## Related tools

- **Argo** (Read-based ARG profiler evaluated at low coverage using '--plasmid -z 0'; produces cpg-normalized ARG abundance estimates per species) — https://github.com/xinehc/argo
- **metaFlye** (Assembly-based branch: assembles each downsampled set into contigs with '--nano-raw --meta' before Kraken2 classification)
- **Kraken2** (Classifies metaFlye contigs for assembly-based ARG host assignment)
- **SemiBin2** (Binning branch: bins contigs into MAGs with 'single_easy_bin --sequencing-type=long_read')
- **GTDB-Tk** (Labels MAGs with 'classify_wf' for binning-based host assignment)
- **DIAMOND** (Builds the ground truth by blastx mapping reference genomes to SARG+ with stringent cutoffs (e-value 1e-15, id 90, subject-cover 90))
- **NanoSim** (Simulates Nanopore metagenomic reads (HQ/LQ profiles) and enables downsampling to target coverages)

## Examples

```
diamond blastx --db database/sarg --query ref_genomes.fa --evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0
```

## Evaluation signals

- The metrics table must reproduce the reported pattern: Argo type-level F1 of 0.811–0.935 and precision of 0.977–0.992 at 1–2× coverage
- Assembly-based (metaFlye + Kraken2) F1 must stay at or below 0.5 at 1–2× and only exceed 0.9 F1 at ≥8× coverage
- Binning-based (SemiBin2 + GTDB-Tk) F1 must stay at or below 0.5 at 1–2× and only exceed 0.9 F1 above 32× coverage
- Ground-truth ARG counts from DIAMOND must be reproducible under the fixed blastx settings (--evalue 1e-15 --id 90 --subject-cover 90)
- Argo's reported output invariants: log reports genome copies, ARG types/subtype counts, and ARG-carrying species counts (e.g., 'ARG copies per genome: 24.478 | ARG types: 14 | ARG subtypes: 138 | ARG-carrying species: 8')

## Limitations

- Argo is inherently reference-based and may misclassify unseen species or novel plasmid syntenies
- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from the SARG+ database
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering resolution for ORF-based annotation or detailed sequence typing
- By default Argo marks ARGs of species below 1× genome copy as 'unclassified' — the benchmark overrides this with '-z 0' (and '--plasmid' to force carrier splitting)
- Insufficient coverage leads to unstable normalization and hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering
- Plasmid reads can have multiple hosts; chimeric reads and uncharacterized plasmids may interfere with carrier identification when '--plasmid' is enabled

## Evidence

- [methods] Argo type-level F1 at low coverage: [evidence span withheld — anchor status: not_found]
- [methods] coverage thresholds of competing methods: [evidence span withheld — anchor status: not_found]
- [readme] SARG+ database description: [evidence span withheld — anchor status: not_found]
