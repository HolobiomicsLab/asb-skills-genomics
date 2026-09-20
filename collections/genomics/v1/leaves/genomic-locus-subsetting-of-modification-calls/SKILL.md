---
name: genomic-locus-subsetting-of-modification-calls
description: Use when you have per-replicate modkit modification site tables (e.g.
  filtered .bed.gz from Dogme) and need to count or intersect modification sites within
  one gene or region, such as testing reproducibility at the Pkm locus.
when_to_use_negative:
- Input is unfiltered modBAM or raw Dorado basecalls rather than filtered, coordinate-annotated
  modkit site tables
- The analysis goal is genome-wide modification profiling rather than locus-specific
  subsetting and replicate comparison
- Reads have not yet been basecalled and mapped (Dorado basecalling and minimap2 mapping
  are upstream prerequisites)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Dogme
  role: Produces the filtered modkit modification site tables that are subset per
    locus
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Extracts and filters modified-base calls into bedMethyl/BED tables that can
    be region-filtered by coordinate
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:074050d7420fdb21a2b71dd72bd3a9e405e0a210c34fc2b79c37beaf3fd5d88f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - modkit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Genomic locus subsetting of modification calls

## Summary

Restricts filtered modkit modification site tables (m6A, m5C, Nm, pseudouridine, inosine) produced by the Dogme pipeline to a genomic locus of interest, enabling per-locus reproducibility analysis across sequencing replicates.

## When to use

Use when you have per-replicate modkit modification site tables (e.g. filtered .bed.gz from Dogme) and need to count or intersect modification sites within one gene or region, such as testing reproducibility at the Pkm locus.

## When NOT to use

- Input is unfiltered modBAM or raw Dorado basecalls rather than filtered, coordinate-annotated modkit site tables
- The analysis goal is genome-wide modification profiling rather than locus-specific subsetting and replicate comparison
- Reads have not yet been basecalled and mapped (Dorado basecalling and minimap2 mapping are upstream prerequisites)

## Inputs

- Filtered modkit modification site tables (.bed.gz) per replicate from the Dogme pipeline
- Genomic coordinates of the target locus (e.g. Pkm / pyruvate kinase, muscle gene region)
- List of modification classes to subset: m6A, m5C, Nm, pseudouridine (pseU), inosine

## Outputs

- Per-replicate modification site counts per class within the locus
- Counts of sites shared across all three replicates per modification class
- Summary table such as pkm_modification_counts.tsv

## How to apply

Load the filtered modkit modification site tables (compressed .bed.gz output of the Dogme pipeline, produced by modkit extraction and scripts/filterbed.py) for each biological replicate, here the three mouse C2C12 myoblast direct RNA sequencing replicates. Restrict each replicate's calls to the target genomic locus using coordinate-based subsetting, e.g. bedtools/pybedtools intersect or modkit region filtering. Count sites per replicate per modification class, then compute shared sites by intersecting site coordinates across all replicates. For Pkm, validate against the reported ranges (21-29 m6A, 26-31 m5C, 28-31 Nm, 27-39 pseudouridine, 18-24 inosine sites per replicate) and shared-site counts (14 m6A, 23 m5C, 16 Nm, 21 pseU, 12 inosine), writing results to pkm_modification_counts.tsv.

## Related tools

- **Dogme** (Produces the filtered modkit modification site tables that are subset per locus) — https://github.com/mortazavilab/dogme
- **modkit** (Extracts and filters modified-base calls into bedMethyl/BED tables that can be region-filtered by coordinate) — https://github.com/nanoporetech/modkit

## Examples

```
bedtools intersect -a rep1.mods.bed.gz -b Pkm.bed > rep1.pkm.mods.tsv
```

## Evaluation signals

- Each per-replicate count at the Pkm locus falls within its reported range: 21-29 m6A, 26-31 m5C, 28-31 Nm, 27-39 pseudouridine, 18-24 inosine
- Shared-site counts across all three replicates match the reported values: 14 m6A, 23 m5C, 16 Nm, 21 pseU, 12 inosine
- All retained site coordinates after subsetting lie within the target locus boundaries (coordinate-based intersect returns zero out-of-region sites)
- Output file pkm_modification_counts.tsv contains per-replicate and shared-site counts for all five modification classes

## Limitations

- The Dogme pipeline filters modkit BED outputs by minimum coverage and per-mod thresholds (params.minCov and params.perMod via scripts/filterbed.py), so site counts depend on those filter settings
- Subsetting cannot recover modifications filtered out upstream; only sites present in the filtered tables can be counted or intersected
- Reported count ranges are specific to the three C2C12 myoblast direct RNA MinION replicates and will not generalize to other datasets or read types (cDNA, genomic DNA)

## Evidence

- [intro] For RNA and DNA, it will extract modifications using modkit and filter those.: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [abstract] three replicates finding: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
- [readme] filterbed thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] read types: "Dogme supports 3 different read types: direct RNA ("RNA"), cDNA ("CDNA"), and genomic DNA ("DNA")."
