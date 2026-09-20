---
name: nanopore-metagenome-read-simulation
description: Use when benchmarking an ARG profiler (e.g., Argo) against assembly/metaFlye+Kraken2
  or binning/SemiBin2+GTDB-Tk pipelines and ground-truth ARG abundances at controlled
  sequencing coverages are needed.
when_to_use_negative:
- When the goal is de novo identification of ARG genomic contexts and hosts in new,
  understudied environments — the article notes assembly- and binning-based methods
  may be better suited there
- When real, externally validated metagenomic samples are required (e.g., the 329-sample
  human and non-human primate fecal validation dataset) rather than simulated ground-truth
  data
- When the input is already sequenced real Nanopore data with no truth set — simulation-based
  benchmarking presupposes controllable coverage and known expected abundances
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: NanoSim
  role: Simulates Nanopore metagenomic reads (metagenome mode, v3.1.0) from 25 reference
    genomes with controlled quality/length profiles
- name: Argo
  role: Read-based ARG profiler benchmarked on every downsampled set; run with '--plasmid
    -z 0' to classify plasmids and species below 1×
  repo: https://github.com/xinehc/argo
- name: DIAMOND
  role: Computes expected ARG abundances by mapping reference genomes to SARG+ (blastx,
    --evalue 1e-15 --id 90 --subject-cover 90)
- name: metaFlye
  role: Assembles downsampled reads into contigs ('--nano-raw --meta') for the assembly-based
    profiling arm
- name: Kraken2
  role: Classifies assembled contigs taxonomically (v2.1.3) in the assembly-based
    arm
- name: SemiBin2
  role: Bins contigs into MAGs ('single_easy_bin --sequencing-type=long_read') in
    the binning-based arm
- name: GTDB-Tk
  role: Labels MAGs taxonomically ('classify_wf', v2.4.0) in the binning-based arm
- name: R
  role: Computes precision, recall, F1-score, Pearson correlation, and L1/L2 distances
    against expected abundances
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:41a0a80c3693e2728f4041022ce1b026b44f0f36e2f415b18e0b185ab3aa4863
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nanopore-metagenome-read-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate Nanopore metagenomic reads with NanoSim and downsample to a graded coverage grid (1×–32×) to benchmark read-based, assembly-based, and binning-based ARG profiling methods against known ground truth.

## When to use

Use when benchmarking an ARG profiler (e.g., Argo) against assembly/metaFlye+Kraken2 or binning/SemiBin2+GTDB-Tk pipelines and ground-truth ARG abundances at controlled sequencing coverages are needed.

## When NOT to use

- When the goal is de novo identification of ARG genomic contexts and hosts in new, understudied environments — the article notes assembly- and binning-based methods may be better suited there
- When real, externally validated metagenomic samples are required (e.g., the 329-sample human and non-human primate fecal validation dataset) rather than simulated ground-truth data
- When the input is already sequenced real Nanopore data with no truth set — simulation-based benchmarking presupposes controllable coverage and known expected abundances

## Inputs

- 25 primary/opportunistic pathogen reference genomes in FASTA (even taxonomic abundance mock design)
- SARG+ curated ARG reference database (sarg.fa, sarg_ref.fa, sarg_ext.fa)
- NanoSim training parameters for Nanopore read quality/length distributions (HQ and LQ)
- Downsampled Nanopore FASTQ/FASTA read sets at 1×, 2×, 4×, 8×, 16×, 32× expected coverage

## Outputs

- Simulated Nanopore metagenomic read sets from NanoSim v3.1.0 (metagenome mode)
- Expected ARG abundance table per type/subtype derived from DIAMOND blastx against SARG+
- Per-method ARG abundance estimates (Argo cpg-normalized; contigs from metaFlye; MAGs from SemiBin2/GTDB-Tk)
- Metrics table: precision, recall, F1-score, Pearson correlation, L1/L2 distances per method per coverage level

## How to apply

Run NanoSim v3.1.0 in metagenome mode to generate 1,000,000 Nanopore reads per sample from 25 primary/opportunistic pathogen reference genomes at even taxonomic abundance, using trained quality/length parameters (HQ: q=19.04, l=5,028; LQ: q=13.19, l=10,365). Downsample both samples to expected coverages of 1×, 2×, 4×, 8×, 16×, and 32×. Compute expected ARG abundances as ground truth by mapping the 25 reference genomes to SARG+ with DIAMOND blastx (--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hps 0 equivalents: --max-hsps 0) and counting ARG occurrences. Run the read-based profiler (Argo v0.1.0, with '--plasmid -z 0' so plasmid carriers and species below 1× genome copy are classified), assemble contigs with metaFlye v2.9.3 ('--nano-raw --meta'), classify with Kraken2 v2.1.3, bin with SemiBin2 v2.1.0 ('single_easy_bin --sequencing-type=long_read'), and assign MAG taxonomy with GTDB-Tk v2.4.0 ('classify_wf'). Evaluate all methods against expected abundances at type and subtype levels using precision, recall, F1-score, Pearson correlation, and L1/L2 distances at each coverage point; the rationale is that simulation provides the only setting where detection thresholds and host-assignment accuracy are known exactly.

## Related tools

- **NanoSim** (Simulates Nanopore metagenomic reads (metagenome mode, v3.1.0) from 25 reference genomes with controlled quality/length profiles)
- **Argo** (Read-based ARG profiler benchmarked on every downsampled set; run with '--plasmid -z 0' to classify plasmids and species below 1×) — https://github.com/xinehc/argo
- **DIAMOND** (Computes expected ARG abundances by mapping reference genomes to SARG+ (blastx, --evalue 1e-15 --id 90 --subject-cover 90))
- **metaFlye** (Assembles downsampled reads into contigs ('--nano-raw --meta') for the assembly-based profiling arm)
- **Kraken2** (Classifies assembled contigs taxonomically (v2.1.3) in the assembly-based arm)
- **SemiBin2** (Bins contigs into MAGs ('single_easy_bin --sequencing-type=long_read') in the binning-based arm)
- **GTDB-Tk** (Labels MAGs taxonomically ('classify_wf', v2.4.0) in the binning-based arm)
- **R** (Computes precision, recall, F1-score, Pearson correlation, and L1/L2 distances against expected abundances)

## Examples

```
argo example.fa.gz -d database -o . --plasmid -z 0
```

## Evaluation signals

- Metrics table reproduces the reported pattern: Argo type-level F1 of 0.811–0.935 and precision of 0.977–0.992 at 1–2× coverage, while assembly- and binning-based methods stay at or below 0.5 F1 at such low coverage
- Assembly-based (metaFlye + Kraken2) profiling reaches F1 > 0.9 only at ≥ 8× coverage; binning-based (SemiBin2 + GTDB-Tk) requires > 32× coverage
- Downsampling grid covers all six expected coverages (1×, 2×, 4×, 8×, 16×, 32×) for both HQ and LQ NanoSim samples
- Expected ARG abundances derive from DIAMOND blastx of the 25 reference genomes against SARG+ under the specified cutoffs (--id 90, --subject-cover 90, --evalue 1e-15); Argo's output table reports cpg-normalized abundances
- Argo demonstrates consistently good performance across the full 1×–32× coverage range in the simulation, as reported in the article

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from the database
- By default Argo considers ARGs of species with less than one genome copy (1×) as 'unclassified'; the '-z 0' override used here relaxes this, and insufficient coverage leads to unstable normalization and weaker read clustering

## Evidence
