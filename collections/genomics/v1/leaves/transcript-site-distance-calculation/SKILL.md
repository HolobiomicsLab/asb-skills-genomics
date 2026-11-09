---
name: transcript-site-distance-calculation
description: Use when transcriptomic modification site coordinates (e.g., CHEUI m6A/m5C
  predictions) need isoform-aware annotation and relative-distance computation, such
  as selecting site pairs 1–15 nt apart for co-occurrence testing.
when_to_use_negative:
- Input is raw nanopore FAST5/nanopolish eventalign output that has not yet been basecalled,
  aligned with minimap2, and processed by CHEUI — no site coordinates exist yet to
  annotate
- Detection of the modifications themselves is the goal (use CHEUI/SWARM prediction
  rather than R2Dtool distance annotation)
- Differential methylation testing between two conditions (CHEUI-diff) is the objective
  rather than positional/distance annotation
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3512
tools:
- name: R2Dtool
  role: Annotates transcript-specific metatranscript coordinates and absolute and
    relative distances to annotated transcript landmarks in an isoform-specific manner;
    provides cheui_to_bed.sh, R2_annotate, and R2_lift used on CHEUI calls
  repo: https://github.com/comprna/R2Dtool
- name: CHEUI
  role: Produces the single-read and transcript-site m6A/m5C predictions (Model 1/Model
    2 probabilities) whose site coordinates are converted and distance-annotated
  repo: github.com/comprna/CHEUI-public
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:c8266a0a1ec9aa4de2635cdd06f12d5cfac7e0e058da54a3ffc5db0cc2cd4620
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/R2Dtool
  tools:
  - CHEUI
  - R2Dtool
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# transcript-site-distance-calculation

## Summary

Computes isoform-aware metatranscript coordinates and absolute/relative distances between transcriptomic RNA modification sites (e.g., CHEUI m6A and m5C calls) using R2Dtool, enabling distance-resolved co-occurrence analyses on nanopore direct RNA sequencing data.

## When to use

Use when transcriptomic modification site coordinates (e.g., CHEUI m6A/m5C predictions) need isoform-aware annotation and relative-distance computation, such as selecting site pairs 1–15 nt apart for co-occurrence testing.

## When NOT to use

- Input is raw nanopore FAST5/nanopolish eventalign output that has not yet been basecalled, aligned with minimap2, and processed by CHEUI — no site coordinates exist yet to annotate
- Detection of the modifications themselves is the goal (use CHEUI/SWARM prediction rather than R2Dtool distance annotation)
- Differential methylation testing between two conditions (CHEUI-diff) is the objective rather than positional/distance annotation

## Inputs

- CHEUI methylation call files (CHEUI-solo Model 1/Model 2 outputs for m6A and m5C)
- Transcriptome annotation GTF (e.g., Gencode v38 for human, Ensembl v104 for mouse)
- BED-like methylation site files produced by cheui_to_bed.sh

## Outputs

- Bed-like CHEUI methylation calls converted by cheui_to_bed.sh
- Isoform-aware site annotations with metatranscript coordinates and absolute/relative distances from R2_annotate
- Lifted site coordinates (genomic) from R2_lift
- Pairs of modified sites at 1–15 nt relative distance for downstream co-occurrence statistics

## How to apply

First convert CHEUI methylation calls to a bed-like format with the R2Dtool script cheui_to_bed.sh (default parameters). Then run the R2Dtool R2_annotate command with default parameters against the relevant GTF annotation (Gencode v38 on GRCh38 for human HEK293 data; Ensembl v104 on GRCm39 for mouse cortex data) to assign isoform-specific metatranscript coordinates and compute absolute and relative distances to transcript landmarks. Use R2_lift with default parameters when annotated methylation calls must be transposed (e.g., to genomic coordinates). In the co-occurrence workflow, the resulting distances are used to select transcripts containing a pair of significantly modified sites (CHEUI-solo Model 2 probability >0.9999) at a relative distance of 1–15 nt, which defines the candidate pairs for read-level co-occurrence analysis. Rationale: distances must be isoform-aware because site positions and landmark relationships differ between transcript isoforms.

## Related tools

- **R2Dtool** (Annotates transcript-specific metatranscript coordinates and absolute and relative distances to annotated transcript landmarks in an isoform-specific manner; provides cheui_to_bed.sh, R2_annotate, and R2_lift used on CHEUI calls) — https://github.com/comprna/R2Dtool
- **CHEUI** (Produces the single-read and transcript-site m6A/m5C predictions (Model 1/Model 2 probabilities) whose site coordinates are converted and distance-annotated) — github.com/comprna/CHEUI-public

## Evaluation signals

- Sites annotated by R2_annotate carry isoform-specific metatranscript coordinates and distance values consistent with the input GTF (Gencode v38 / Ensembl v104)
- Selected site pairs satisfy the co-occurrence workflow criteria: both sites Model 2 probability >0.9999 and relative distance within 1–15 nt
- cheui_to_bed.sh output is a valid bed-like file with default parameters and is accepted downstream by R2_annotate
- R2_lift output correctly transposes annotated methylation calls without changing site identities or modification states
- CCDC102A transcript pair co-occurrence statistic computed on annotated pairs lands near the reported ~78%

## Limitations

- Strictly distant-site selection at <5 nt can be confounded: 'Signal interference at distances <5 nt would be exacerbated in highly and densely modified ribosomal RNA molecules', so very short-distance pairs should be interpreted cautiously (rRNA reads/sites are excluded)
- R2Dtool requires Rust/Cargo and R with tidyverse, binom, and ragg; annotation files must match the reference (GRCh38/Gencode v38 for human, GRCm39/Ensembl v104 for mouse)
- Distances are isoform-specific, so results depend on correct transcript isoform assignment of each site

## Evidence

- [readme] R2Dtool annotates transcript-specific metatranscript coordinates and absolute and relative distances to annotated transcript landmarks, in an isoform-specific manner.: [evidence span withheld — anchor status: external_doc]
- [methods] cheui_to_bed.sh converts CHEUI methylation calls to bed-like format: "the R2Dtool script cheui_to_bed.sh was run with default parameters to convert the CHEUI methylation calls to a bed-like format"
- [methods] R2_annotate used with default parameters and relevant GTF: "the R2Dtool R2_annotate command was used with default parameters and the relevant GTF annotation"
- [methods] R2_lift transposes annotated methylation calls: "the R2Dtool R2_lift command was run with default parameters to transpose the annotated methylation calls"
- [methods] Reference annotations used were GRCh38/Gencode v38 and GRCm39/Ensembl v104: [evidence span withheld — anchor status: not_found]
