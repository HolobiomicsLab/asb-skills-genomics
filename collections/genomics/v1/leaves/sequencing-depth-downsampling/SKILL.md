---
name: sequencing-depth-downsampling
description: Use when benchmarking a long-read ARG profiling or taxonomic classification
  pipeline's sensitivity to sequencing depth, requiring matched subsets of reads spanning
  low (1–2×) to high (32×) coverage.
when_to_use_negative:
- When evaluating a method at a single sequencing depth with no interest in the coverage-performance
  curve — a single run suffices
- When the goal is de novo discovery of ARG genomic contexts in understudied environments
  rather than depth-benchmarked detection (assembly-/binning-based approaches may
  be preferred per the article)
- When taxonomic composition varies between samples, since differences would confound
  the coverage effect being measured
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: NanoSim
  role: Simulates the Nanopore metagenomic samples (v3.1.0, metagenome mode) that
    are subsequently downsampled across coverage points
- name: DIAMOND
  role: Maps the 25 reference genomes to SARG+ (blastx, --evalue 1e-15, --id 90, --subject-cover
    90) to build depth-independent ground-truth ARG abundances for evaluating downsampled
    runs
- name: Argo
  role: Read-based profiler run on every downsampled set ('--plasmid -z 0') to measure
    detection performance versus coverage
  repo: https://github.com/xinehc/argo
- name: metaFlye
  role: Assembles each downsampled set ('--nano-raw --meta') for the assembly-based
    comparison arm
- name: Kraken2
  role: Classifies contigs from each downsampled set in the assembly-based comparison
    arm
- name: SemiBin2
  role: Bins contigs from each downsampled set ('single_easy_bin --sequencing-type=long_read')
    in the binning-based comparison arm
- name: GTDB-Tk
  role: Labels MAGs from each downsampled set ('classify_wf') in the binning-based
    comparison arm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:606ccdae8bfe545bc1a2df21e79f18cf2c23870ecc93bc0e96d75e882e5ccda8
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
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sequencing-depth-downsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically downsamples simulated or real Nanopore metagenomic reads to a ladder of expected coverages (1×, 2×, 4×, 8×, 16×, 32×) so that detection-performance of ARG profiling methods can be compared as a function of sequencing depth.

## When to use

Use when benchmarking a long-read ARG profiling or taxonomic classification pipeline's sensitivity to sequencing depth, requiring matched subsets of reads spanning low (1–2×) to high (32×) coverage.

## When NOT to use

- When evaluating a method at a single sequencing depth with no interest in the coverage-performance curve — a single run suffices
- When the goal is de novo discovery of ARG genomic contexts in understudied environments rather than depth-benchmarked detection (assembly-/binning-based approaches may be preferred per the article)
- When taxonomic composition varies between samples, since differences would confound the coverage effect being measured

## Inputs

- Nanopore metagenomic read sets (FASTA/FASTQ), either simulated with NanoSim or real, from samples of known genome composition
- Reference genomes used to compose the mock community (25 primary/opportunistic pathogens) for computing ground-truth ARG abundances
- Expected coverage ladder (1×, 2×, 4×, 8×, 16×, 32×) and mean read length/quality parameters

## Outputs

- Per-coverage downsampled read subsets with constant taxonomic composition
- Per-coverage profiling outputs from each method (e.g., Argo cpg-normalized ARG abundance tables)
- Metrics table of precision, recall, F1-score, Pearson correlation, and L1/L2 distances at type and subtype levels per coverage point

## How to apply

Generate a deep Nanopore metagenomic sample (e.g., NanoSim v3.1.0 in metagenome mode with calibrated HQ q=19.04/l=5028 or LQ q=13.19/l=10,365 read profiles over 25 even-abundance reference genomes, 1,000,000 reads each). From each full sample, subset reads to expected genome coverages of 1×, 2×, 4×, 8×, 16×, and 32×, keeping the taxonomic composition constant so that only depth varies. Establish ground-truth ARG abundances independently of depth (here, mapping the 25 reference genomes to SARG+ with DIAMOND blastx using --evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0). Run each profiling method (Argo read-based, metaFlye+Kraken2 assembly-based, SemiBin2+GTDB-Tk binning-based) on every downsampled set and evaluate precision, recall, F1-score, Pearson correlation, and L1/L2 distances at type and subtype levels per coverage point. The rationale is that coverage thresholds differ drastically between strategies, so the ladder reveals each method's minimum depth for reliable detection.

## Related tools

- **NanoSim** (Simulates the Nanopore metagenomic samples (v3.1.0, metagenome mode) that are subsequently downsampled across coverage points)
- **DIAMOND** (Maps the 25 reference genomes to SARG+ (blastx, --evalue 1e-15, --id 90, --subject-cover 90) to build depth-independent ground-truth ARG abundances for evaluating downsampled runs)
- **Argo** (Read-based profiler run on every downsampled set ('--plasmid -z 0') to measure detection performance versus coverage) — https://github.com/xinehc/argo
- **metaFlye** (Assembles each downsampled set ('--nano-raw --meta') for the assembly-based comparison arm)
- **Kraken2** (Classifies contigs from each downsampled set in the assembly-based comparison arm)
- **SemiBin2** (Bins contigs from each downsampled set ('single_easy_bin --sequencing-type=long_read') in the binning-based comparison arm)
- **GTDB-Tk** (Labels MAGs from each downsampled set ('classify_wf') in the binning-based comparison arm)

## Evaluation signals

- The metrics table should reproduce the reported pattern: Argo reaches type-level F1 of 0.811–0.935 and precision of 0.977–0.992 at 1–2× coverage, while assembly-based stays at or below 0.5 F1 there and needs ≥8×, binning-based >32×, to exceed 0.9 F1
- Downsampled subsets must contain reads corresponding to all 25 genomes with the original even taxonomic abundance preserved at every coverage point
- Coverage ladder must include the exact points 1×, 2×, 4×, 8×, 16×, 32× for both HQ and LQ samples
- Ground-truth ARG counts from DIAMOND-vs-SARG+ mapping must be computed once from the full genomes, independent of coverage
- Per-coverage metrics (precision, recall, F1, Pearson, L1/L2 distances) should be reported at both type and subtype levels for direct cross-method comparison

## Limitations

- Simulated coverage ladders depend on accurate NanoSim calibration (HQ q=19.04, l=5028; LQ q=13.19, l=10,365); read-quality distributions of real platforms may shift thresholds
- Insufficient coverage leads to unstable abundance normalization and hinders accurate taxonomic classification due to inadequate read overlaps and weaker read clustering
- Species present below 1× genome copy cannot be resolved by default Argo settings unless '-z 0' is used to force classification

## Evidence

- [other] Read-based profiling (Argo) has far lower detection thresholds: [evidence span withheld — anchor status: not_found]
- [other] coverage ladder and sample simulation: [evidence span withheld — anchor status: not_found]
- [readme] Argo run configuration on downsampled sets: [evidence span withheld — anchor status: not_found]
