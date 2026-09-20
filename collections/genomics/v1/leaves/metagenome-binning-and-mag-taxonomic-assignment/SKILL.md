---
name: metagenome-binning-and-mag-taxonomic-assignment
description: Use when long Nanopore (or PacBio) metagenomic reads at moderate-to-high
  coverage (≥8×) should be assembled into contigs and binned into MAGs so ARGs can
  be linked to de novo-reconstructed hosts rather than classified read-by-read.
when_to_use_negative:
- Sample coverage is ≤8× for assembly-based (or ≤32× for binning-based) detection
  accuracy targets — at 1–2× these approaches stay at or below 0.5 F1 while read-based
  Argo already achieves type-level F1 of 0.811–0.935
- Input reads are short second-generation reads — this workflow depends on long reads
  for metaFlye '--nano-raw' assembly and SemiBin2 '--sequencing-type=long_read' binning
- Goal is per-read or per-cluster taxonomic classification without de novo reconstruction
  — Argo's read-overlapping profiler avoids assembly entirely and classifies ARGs
  at species level directly
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: metaFlye
  role: Assembles downsampled Nanopore reads into contigs with '--nano-raw --meta'
    metagenomic mode (v2.9.3)
- name: Kraken2
  role: Classifies assembled contigs taxonomically (v2.1.3) as the assembly-based
    profiling comparator
- name: SemiBin2
  role: Bins contigs into MAGs using long-read sequencing mode via 'single_easy_bin
    --sequencing-type=long_read' (v2.1.0)
- name: GTDB-Tk
  role: Assigns GTDB-based taxonomy to MAGs with 'classify_wf' (v2.4.0) to complete
    host labeling
- name: DIAMOND
  role: Computes expected ARG abundances (ground truth) by mapping reference genomes
    to SARG+ with blastx and stringent cutoffs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:be72f6255607b183c700fbb837083315527f9a191b1d82adf8390b505fb90a1a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  tools:
  - DIAMOND
  - GTDB-Tk
  - Kraken2
  - NanoSim
  - R
  - SemiBin2
  - metaFlye
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metagenome-binning-and-mag-taxonomic-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles ARG-carrying species from long Nanopore metagenomes by assembling reads with metaFlye, classifying contigs with Kraken2, binning contigs into MAGs with SemiBin2, and labeling MAGs with GTDB-Tk — the binning-based alternative benchmarked against Argo's read-based profiling across 1×–32× coverage.

## When to use

Use when long Nanopore (or PacBio) metagenomic reads at moderate-to-high coverage (≥8×) should be assembled into contigs and binned into MAGs so ARGs can be linked to de novo-reconstructed hosts rather than classified read-by-read.

## When NOT to use

- Sample coverage is ≤8× for assembly-based (or ≤32× for binning-based) detection accuracy targets — at 1–2× these approaches stay at or below 0.5 F1 while read-based Argo already achieves type-level F1 of 0.811–0.935
- Input reads are short second-generation reads — this workflow depends on long reads for metaFlye '--nano-raw' assembly and SemiBin2 '--sequencing-type=long_read' binning
- Goal is per-read or per-cluster taxonomic classification without de novo reconstruction — Argo's read-overlapping profiler avoids assembly entirely and classifies ARGs at species level directly

## Inputs

- Quality-controlled long Nanopore reads (FASTQ), e.g., NanoSim v3.1.0 metagenome-mode simulations (HQ q=19.04, l=5028; LQ q=13.19, l=10,365)
- Downsampled read sets at expected coverages of 1×, 2×, 4×, 8×, 16×, and 32×
- SARG+ reference database plus pathogen reference genomes for computing expected ARG abundances (DIAMOND blastx ground truth)

## Outputs

- metaFlye metagenomic assembly (contigs FASTA)
- Kraken2 per-contig taxonomic classifications
- SemiBin2 metagenome-assembled genomes (MAG bins)
- GTDB-Tk species labels per MAG
- Binning-based ARG abundance profile evaluated at type and subtype levels against expected abundances (precision, recall, F1, Pearson r, L1/L2 distances per coverage)

## How to apply

Downsample quality-controlled Nanopore reads to a target expected coverage (1×, 2×, 4×, 8×, 16×, or 32×), then assemble with metaFlye v2.9.3 using '--nano-raw --meta' for metagenomic mode. Classify the resulting contigs with Kraken2 v2.1.3 to obtain preliminary taxonomy, then bin contigs into MAGs with SemiBin2 v2.1.0 via 'single_easy_bin --sequencing-type=long_read', and assign species-level labels to each MAG with GTDB-Tk v2.4.0 'classify_wf'. For evaluation, compute expected ARG abundances independently by mapping the reference genomes (e.g., the 25 pathogens used in the benchmark) to SARG+ with DIAMOND blastx ('--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0'), then score the binning-based profile against these ground truth abundances at type and subtype levels using precision, recall, F1-score, Pearson correlation, and L1/L2 distances per coverage point. The decision point is coverage: assembly-based profiling (metaFlye + Kraken2) needs at least 8× and binning-based profiling (SemiBin2 + GTDB-Tk) more than 32× to reach F1-scores above 0.9, so route low-coverage samples to read-based profiling with Argo instead.

## Related tools

- **metaFlye** (Assembles downsampled Nanopore reads into contigs with '--nano-raw --meta' metagenomic mode (v2.9.3))
- **Kraken2** (Classifies assembled contigs taxonomically (v2.1.3) as the assembly-based profiling comparator)
- **SemiBin2** (Bins contigs into MAGs using long-read sequencing mode via 'single_easy_bin --sequencing-type=long_read' (v2.1.0))
- **GTDB-Tk** (Assigns GTDB-based taxonomy to MAGs with 'classify_wf' (v2.4.0) to complete host labeling)
- **DIAMOND** (Computes expected ARG abundances (ground truth) by mapping reference genomes to SARG+ with blastx and stringent cutoffs)

## Examples

```
SemiBin2 single_easy_bin --sequencing-type=long_read; gtdbtk classify_wf
```

## Evaluation signals

- Metrics table reproduces the reported pattern: binning-based (SemiBin2 + GTDB-Tk) F1 ≤ 0.5 at low coverage and F1 > 0.9 only above 32×; assembly-based (metaFlye + Kraken2) needs ≥ 8× for F1 > 0.9
- Expected ARG abundance table from DIAMOND blastx vs SARG+ is reproducible with the specified cutoffs (--evalue 1e-15 --id 90 --subject-cover 90) and the 25 reference genomes
- F1-score, precision, recall, Pearson correlation, and L1/L2 distances are reported at both type and subtype levels for every coverage point (1×–32×)
- MAG labels are consistent with GTDB taxonomy, and read-poor/low-coverage bins do not silently inflate precision

## Limitations

- Binning-based profiling requires more than 32× coverage and assembly-based profiling at least 8× to exceed 0.9 F1, making both unsuitable for scarce samples
- These methods trade detection sensitivity at low coverage for the ability to de novo identify ARG genomic contexts and hosts in new, understudied environments
- Read-based Argo serves as the appropriate comparator at 1–2×, where it reached type-level F1 of 0.811–0.935 and precision of 0.977–0.992

## Evidence
