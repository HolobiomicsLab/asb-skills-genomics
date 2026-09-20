---
name: genome-copy-estimation-from-metagenomes
description: Use when long-read (Nanopore/PacBio) fecal or environmental metagenomes
  need per-sample genome-copy normalization, e.g. to convert ARG read counts into
  cpg within the Argo profiling pipeline.
when_to_use_negative:
- Samples with insufficient coverage — normalization is unstable and read clustering
  for taxonomic classification is hindered
- Extremely rare species whose abundance falls below 1x genome copy — Argo will mark
  their ARGs 'unclassified' rather than estimate them
- Short-read metagenomes or de novo discovery of ARG genomic contexts in new, understudied
  environments — Argo is reference-based and long-read-only; assembly-based (metaFlye
  + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches are better suited
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Argo
  role: Runs the read-overlapping ARG profiling workflow and combines Melon genome-copy
    estimates with SARG+ assignments to report cpg abundances
  repo: https://github.com/xinehc/argo
- name: Melon
  role: Estimates per-sample genome copies from quality-controlled long reads
- name: Kraken2
  role: PlusPF pre-filtering of non-prokaryotic reads before genome-copy estimation
- name: nanoq
  role: Quality control of reads (--min-qual 10 --min-len 1000) prior to estimation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:7365ad28352261c5a2e2847a4b0b44a8132de0def5258a747145df881fe97cc6
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
  - Kraken2
  - Melon
  - Porechop
  - R
  - ggplot2
  - nanoq
  - vegan
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# genome-copy-estimation-from-metagenomes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates per-sample prokaryotic genome copies (cells) from quality-controlled long-read metagenomes — via Melon with Kraken2 PlusPF pre-filtering of non-prokaryotic reads — so that Argo can express ARG counts as copies per genome (cpg).

## When to use

Use when long-read (Nanopore/PacBio) fecal or environmental metagenomes need per-sample genome-copy normalization, e.g. to convert ARG read counts into cpg within the Argo profiling pipeline.

## When NOT to use

- Samples with insufficient coverage — normalization is unstable and read clustering for taxonomic classification is hindered
- Extremely rare species whose abundance falls below 1x genome copy — Argo will mark their ARGs 'unclassified' rather than estimate them
- Short-read metagenomes or de novo discovery of ARG genomic contexts in new, understudied environments — Argo is reference-based and long-read-only; assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches are better suited

## Inputs

- Quality-controlled long reads (FASTQ, Nanopore or PacBio)
- Raw ONT fecal metagenome FASTQ files (e.g. 317 human + 12 chimpanzee/bonobo samples)
- Kraken2 PlusPF database for non-prokaryotic read pre-filtering
- Argo database bundle (SARG+ and ARG-tailored GTDB) from Zenodo record 20451539

## Outputs

- Per-sample estimated genome copies (prokaryotic cell equivalents)
- Species-resolved ARG abundance table in ARG copies per genome (cpg)
- Table of predicted ARG types/subtypes with potential host species

## How to apply

First quality-control reads with Porechop (--discard_middle) followed by nanoq (--min-qual 10 --min-len 1000), since Argo requires quality-controlled long reads as input. Pre-filter non-prokaryotic reads with Kraken2 using the PlusPF database, then estimate genome copies per sample with Melon. Argo combines these estimates with SARG+ ARG assignments (read-overlap graph clustering with MCL inflation=2, expansion=2 and greedy weighted set cover) to divide ARG copy counts by estimated genome copies, yielding cpg; this is equivalent to copies per cell (cpc) under the assumption that each cell contains a single genome. Treat species with less than 1x genome copy as 'unclassified' (Argo's default threshold) and check read depth before trusting the normalization, because insufficient coverage makes cpg estimates unstable.

## Related tools

- **Argo** (Runs the read-overlapping ARG profiling workflow and combines Melon genome-copy estimates with SARG+ assignments to report cpg abundances) — https://github.com/xinehc/argo
- **Melon** (Estimates per-sample genome copies from quality-controlled long reads)
- **Kraken2** (PlusPF pre-filtering of non-prokaryotic reads before genome-copy estimation)
- **nanoq** (Quality control of reads (--min-qual 10 --min-len 1000) prior to estimation)

## Examples

```
nanoq --min-qual 10 --min-len 1000 -i sample.fastq -o sample.qc.fastq
```

## Evaluation signals

- Output abundance table reports cpg values that equal copies per cell under the single-genome-per-cell assumption
- Species estimated below 1x genome copy appear as 'unclassified' hosts rather than being force-assigned
- Reproducing the 329-sample fecal comparison yields total-ARG cpg Kruskal–Wallis χ2 = 98.4 with adjusted p < 2e-16 and a decreasing CN→KR→SG→DE→US→NH trend
- Sanity anchors: E. coli sul2 ≈ 0.705 cpg in the CN group and all sul/mcr genes absent from NH primate samples
- QC invariant: no reads shorter than 1000 bp or below Q10 survive into the genome-copy estimation step

## Limitations

- Insufficient coverage leads to unstable normalization and hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering
- Argo considers ARGs of species with less than one genome copy (1x) as 'unclassified' by default, making it less suitable for extremely rare species
- cpg assumes each cell contains a single genome; polyploid or multi-copy contexts violate this equivalence
- Argo is reference-based and may misclassify unseen species or novel plasmid syntenies

## Evidence
