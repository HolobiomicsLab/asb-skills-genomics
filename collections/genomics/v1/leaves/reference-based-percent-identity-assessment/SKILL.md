---
name: reference-based-percent-identity-assessment
description: Use when you have corrected long reads (e.g., HECIL output FASTA) for
  an organism with a reference genome and need read-level PI / % matched bases to
  verify correction gains.
when_to_use_negative:
- No reference genome exists for the organism, so only k-mer counts (Jellyfish) or
  de novo assembly metrics (Canu + QUAST + MUMmer dnadiff) can be used for evaluation
- You have raw, uncorrected long reads and want correction itself rather than post-correction
  quality assessment (run the HECIL quick/optimization correction workflow first)
- The goal is assembly-level quality evaluation rather than read-level percent identity
  against a reference
edam_operation: http://edamontology.org/operation_0254
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: BLASR
  role: Aligns the corrected long reads to the reference genome so PI and % matched
    bases can be computed
- name: HECIL
  role: Upstream tool that produces the corrected reads (LongRead_Corrected.fasta)
    that are then assessed against the reference
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:2bfe48096b828bfb16a2af1566567f6c2110564e5a544283e412d81a8a16e3c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BLASR
  - BWA-MEM
  - HECIL
  - Jellyfish
  edam_operation: http://edamontology.org/operation_0254
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Reference-based percent identity assessment

## Summary

Assesses the quality of hybrid-corrected long reads by aligning them to a reference genome (e.g., BLASR against the A. funestus Fumoz genome, GenBank GCA_000349085.1) and computing percent identity (PI) and % matched bases per flowcell and per HECIL iteration.

## When to use

Use when you have corrected long reads (e.g., HECIL output FASTA) for an organism with a reference genome and need read-level PI / % matched bases to verify correction gains.

## When NOT to use

- No reference genome exists for the organism, so only k-mer counts (Jellyfish) or de novo assembly metrics (Canu + QUAST + MUMmer dnadiff) can be used for evaluation
- You have raw, uncorrected long reads and want correction itself rather than post-correction quality assessment (run the HECIL quick/optimization correction workflow first)
- The goal is assembly-level quality evaluation rather than read-level percent identity against a reference

## Inputs

- Corrected long reads in FASTA (HECIL output LongRead_Corrected.fasta)
- Reference genome FASTA for the sample organism (e.g., Anopheles funestus strain Fumoz, GenBank GCA_000349085.1)

## Outputs

- Percent identity (PI) per flowcell and per HECIL iteration
- % matched bases per flowcell and per HECIL iteration
- Metrics CSV, e.g., hecil_afunestus_flowcell_metrics.csv

## How to apply

Run HECIL to produce the corrected reads (saved as LongRead_Corrected.fasta per the README), then align the corrected long reads to the reference genome using BLASR — for the A. funestus data this is the strain Fumoz assembly, GenBank GCA_000349085.1. Compute percent identity (PI) and % matched bases per flowcell and per HECIL iteration so that the effect of the iterative learning component can be tracked across iterations 1..n. Judge success by agreement with reported values (e.g., PI rising from 94.5 to 98.5 on flowcell 1; 87.5% matched bases with 3.8% improvement on flowcell 16 at iteration 5) and by PI increasing across HECIL iterations, and write all values to a metrics table such as hecil_afunestus_flowcell_metrics.csv.

## Related tools

- **BLASR** (Aligns the corrected long reads to the reference genome so PI and % matched bases can be computed)
- **HECIL** (Upstream tool that produces the corrected reads (LongRead_Corrected.fasta) that are then assessed against the reference) — https://github.com/NDBL/HECIL

## Evaluation signals

- Reproduced PI and matched-base values agree with reported flowcell results (e.g., PI rising from 94.5 to 98.5 on flowcell 1; 87.5% matched bases with 3.8% improvement on flowcell 16 at iteration 5)
- PI increases monotonically (or non-decreasingly) across successive HECIL iterations until termination
- Metrics are reported separately for each flowcell (e.g., flowcells 1, 4, and 16 of the 44 A. funestus flowcells) and each iteration
- All PI and % matched-base values are written to hecil_afunestus_flowcell_metrics.csv with per-flowcell, per-iteration rows

## Limitations

- The paper's iterative improvement is 'only a conjecture with no mathematical proof', so PI gains must be verified empirically per dataset rather than assumed
- Assessment requires a suitable reference genome; the paper validates against strain-specific assemblies (e.g., Fumoz GCA_000349085.1), so mismatched strain references could bias PI

## Evidence

- [results] Aligning corrected long reads to reference genome with BLASR: "we align corrected long reads to its reference genome using BLASR21"
- [results] Fumoz reference used for validating corrections: "The reference genome of strain Fumoz (GenBank assembly accession: GCA_000349085.1) is used for validating corrections"
- [readme] HECIL output file that feeds the reference-based assessment: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
