---
name: rna-modification-site-counting-per-replicate
description: Use when filtered modkit BED site tables from multiple nanopore direct
  RNA replicates are available and the goal is per-replicate site counts plus cross-replicate
  shared-site counts at a specific gene locus (e.g., Pkm in mouse C2C12 dRNA data).
when_to_use_negative:
- The input is raw POD5 or unfiltered modBAM — site counting requires Dogme's filtered
  modkit BED outputs, not pre-basecalling or pre-extraction files.
- The question is transcript isoform abundance rather than modification-site reproducibility
  — use LR-Kallisto quantification instead.
- The analysis targets whole-transcriptome modification totals rather than a single
  locus — use genome-wide site tables directly.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Dogme
  role: Nextflow pipeline that basecalls POD5 files with Dorado, maps with minimap2,
    and produces the filtered modkit modification site tables counted here
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Extracts and summarizes modified-base calls into bedMethyl/BED site tables
    per modification class; also supports region-based filtering of the site tables
- name: filterbed.py
  role: Dogme helper script that filters modkit BED outputs by params.minCov and params.perMod
    thresholds before site counting
  repo: https://github.com/mortazavilab/dogme
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:708f8a9ed20d09584c9615c49852ca403cd06a59c83edf52307c4317052509d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - filterbed.py
  - modkit
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# rna-modification-site-counting-per-replicate

## Summary

Counts RNA modification sites (m6A, m5C, Nm, pseudouridine, inosine) per replicate at a genomic locus of interest from filtered modkit modification site tables, then intersects site coordinates across replicates to quantify shared (reproducible) sites.

## When to use

Use when filtered modkit BED site tables from multiple nanopore direct RNA replicates are available and the goal is per-replicate site counts plus cross-replicate shared-site counts at a specific gene locus (e.g., Pkm in mouse C2C12 dRNA data).

## When NOT to use

- The input is raw POD5 or unfiltered modBAM — site counting requires Dogme's filtered modkit BED outputs, not pre-basecalling or pre-extraction files.
- The question is transcript isoform abundance rather than modification-site reproducibility — use LR-Kallisto quantification instead.
- The analysis targets whole-transcriptome modification totals rather than a single locus — use genome-wide site tables directly.

## Inputs

- Filtered modkit modification site BED/BED.gz tables (one per replicate) from the Dogme pipeline, with modification class annotations (m6A, m5C, Nm, pseudouridine, inosine)
- Genomic coordinates of the target locus (e.g., mouse Pkm region from a GTF/mm10 reference)

## Outputs

- Per-replicate site counts per modification class at the locus (e.g., pkm_modification_counts.tsv)
- Shared-site counts per modification class from three-replicate coordinate intersection

## How to apply

Load the filtered modkit modification site tables for each replicate produced by the Dogme pipeline (per-mod BED/BED.gz outputs with coverage and per-mod thresholds already applied by scripts/filterbed.py using params.minCov and params.perMod). Restrict each replicate's calls to the target genomic locus (e.g., the Pkm/pyruvate kinase, muscle locus) with coordinate-based subsetting such as bedtools/pybedtools intersect or modkit region filtering. Count sites per modification class (m6A, m5C, Nm, pseudouridine, inosine) per replicate, then intersect site coordinates across all replicates and count the resulting shared sites per modification. Rationale: Dogme's abstract reports reproducible modification profiles across replicates, so shared-site counts are the reproducibility metric; per-replicate counts are validated against reported ranges (21–29 m6A, 26–31 m5C, 28–31 Nm, 27–39 pseudouridine, 18–24 inosine at Pkm) and shared counts against 14 m6A, 23 m5C, 16 Nm, 21 Ψ, and 12 inosine. Write per-replicate and shared-site counts to a TSV (e.g., pkm_modification_counts.tsv).

## Related tools

- **Dogme** (Nextflow pipeline that basecalls POD5 files with Dorado, maps with minimap2, and produces the filtered modkit modification site tables counted here) — https://github.com/mortazavilab/dogme
- **modkit** (Extracts and summarizes modified-base calls into bedMethyl/BED site tables per modification class; also supports region-based filtering of the site tables)
- **filterbed.py** (Dogme helper script that filters modkit BED outputs by params.minCov and params.perMod thresholds before site counting) — https://github.com/mortazavilab/dogme

## Examples

```
python scripts/filterbed.py --in rep1.bed.gz --out rep1.filtered.bed.gz --minCov $params.minCov --perMod $params.perMod
```

## Evaluation signals

- Each per-replicate count falls within the reported Pkm ranges: 21–29 m6A, 26–31 m5C, 28–31 Nm, 27–39 pseudouridine, 18–24 inosine.
- Shared-site counts match reported values: 14 m6A, 23 m5C, 16 Nm, 21 pseudouridine, 12 inosine across all three replicates.
- Shared-site count is ≤ the minimum per-replicate count for each modification class (intersection invariant).
- Output TSV contains one count per modification class per replicate plus a shared count, all restricted to the Pkm locus.
- Site tables have already passed filterbed.py thresholds (params.minCov, params.perMod); counts computed on unfiltered tables will not match.

## Limitations

- Counts depend on Dogme's modkit filtering thresholds (minCov, perMod); changing thresholds changes counts and reproducibility conclusions.
- Site-level intersection requires exact coordinate agreement across replicates; strand handling matters (report_bed_rep.py merges plus/minus strands per sample).
- Published validation numbers are specific to the three C2C12 mouse myoblast direct RNA MinION replicates and the Pkm locus; other loci or datasets need re-derivation.

## Evidence

- [readme] Filtered modkit inputs and thresholds: [evidence span withheld — anchor status: external_doc]
- [abstract] Reproducibility rationale: "The pipeline produced reproducible modification profiles and transcript expression levels across replicates"
- [abstract] Detected modification classes genome-wide: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
- [intro] Modification extraction step: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] Cross-replicate comparison tooling: [evidence span withheld — anchor status: external_doc]
