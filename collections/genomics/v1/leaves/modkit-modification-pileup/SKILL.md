---
name: modkit-modification-pileup
description: Use when you have Dorado-basecalled, minimap2-mapped nanopore modBAMs
  with MM/ML tags and need per-position modification pileups (e.g., m6A, m5C) as BED/bedMethyl
  tables or open-chromatin regions.
when_to_use_negative:
- Input is raw POD5 without basecalling/alignment — Dorado basecalling and minimap2
  mapping must run first.
- Read type is direct RNA or cDNA where the goal is transcript quantification — LR-Kallisto,
  not modkit pileup, is the relevant stage (modkit still handles the modification
  track).
- Input BAM lacks MM/ML modification tags, since modkit pileup has no modification
  calls to summarize.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: modkit
  role: Computes pileups of modification signals from modBAMs; v0.5+ used for m6A
    open-chromatin calls and bedMethyl generation
  repo: https://github.com/nanoporetech/modkit
- name: Dogme
  role: Nextflow pipeline that orchestrates modkit pileup, custom filtering (minCov/perMod),
    and per-genome consolidation of open-chromatin intervals
  repo: https://github.com/mortazavilab/dogme
- name: Dorado
  role: Upstream basecaller producing the modBAMs (with modification tags) that modkit
    pileup consumes
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Maps Dorado basecalls to the reference to create the aligned modBAM input
    for pileup
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:584f8cf36a4a36db6d70d604f417b108b9629c0a0ce4cc7b2066ec5fd777b336
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - Dogme
  - Dorado
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# modkit-modification-pileup

## Summary

Run modkit (v0.5+) pileup on aligned nanopore modBAMs to extract per-site modification counts/frequencies, then filter and consolidate them into bedMethyl/BED and BedGraph outputs for modification calling (including m6A-based open-chromatin calls in Dogme).

## When to use

Use when you have Dorado-basecalled, minimap2-mapped nanopore modBAMs with MM/ML tags and need per-position modification pileups (e.g., m6A, m5C) as BED/bedMethyl tables or open-chromatin regions.

## When NOT to use

- Input is raw POD5 without basecalling/alignment — Dorado basecalling and minimap2 mapping must run first.
- Read type is direct RNA or cDNA where the goal is transcript quantification — LR-Kallisto, not modkit pileup, is the relevant stage (modkit still handles the modification track).
- Input BAM lacks MM/ML modification tags, since modkit pileup has no modification calls to summarize.

## Inputs

- Aligned modBAM from Dorado basecalling + minimap2 mapping, with MM/ML base-modification tags
- Exogenous-m6A-labeled genomic DNA modBAM (for the open-chromatin workflow)

## Outputs

- Per-site modification pileup (bedMethyl / bed-like table with read-depth and modification-frequency columns)
- filtered_modifications.bed after the >=3 reads and >=5% frequency filter
- Consolidated open_chromatin.bed.gz per-genome intervals (gDNA m6A workflow)
- Per-genome BedGraph tracks for genome-browser visualization

## How to apply

Load the aligned modBAM produced upstream by Dorado basecalling and minimap2 mapping, then run modkit pileup to convert the MM/ML base-modification tags into per-reference-position counts and modification frequencies. In Dogme, apply the custom filter retaining only sites with at least 3 supporting reads (params.minCov) and at least 5% modification frequency (params.perMod) to reduce false positives while keeping sensitivity; scripts/filterbed.py performs this on compressed .bed.gz files. For genomic DNA labeled with exogenous m6A, consolidate the filtered per-chromosome pileup one chromosome at a time (limiting memory) into a per-genome open-chromatin interval set, exported as open_chromatin.bed.gz plus per-genome BedGraph tracks. Validation checks that the BED exists, is valid compressed BED, contains read-depth and frequency columns, and that all retained rows satisfy read count >= 3 and frequency >= 0.05.

## Related tools

- **modkit** (Computes pileups of modification signals from modBAMs; v0.5+ used for m6A open-chromatin calls and bedMethyl generation) — https://github.com/nanoporetech/modkit
- **Dogme** (Nextflow pipeline that orchestrates modkit pileup, custom filtering (minCov/perMod), and per-genome consolidation of open-chromatin intervals) — https://github.com/mortazavilab/dogme
- **Dorado** (Upstream basecaller producing the modBAMs (with modification tags) that modkit pileup consumes) — https://github.com/nanoporetech/dorado
- **minimap2** (Maps Dorado basecalls to the reference to create the aligned modBAM input for pileup)

## Examples

```
modkit pileup aligned.mod.bam --out pileup.bed.gz && python scripts/filterbed.py --input pileup.bed.gz --output filtered_modifications.bed.gz --minCov 3 --perMod 0.05
```

## Evaluation signals

- Output BED file exists and is valid compressed BED (.bed.gz) format.
- Output contains read-count and modification-frequency columns.
- All retained rows satisfy read count >= 3 and modification frequency >= 0.05.
- Consolidated per-genome intervals correspond to high-confidence m6A-modified positions from the modkit pileup, derived one chromosome at a time.
- BedGraph tracks are present per genome and render expected open-chromatin signal in a genome browser.

## Limitations

- Open-chromatin calling requires modkit 0.5 or higher and mapped BAMs carrying m6A modifications.
- Per-chromosome processing is used to reduce annotation memory use; whole-genome one-shot processing may exhaust memory.
- The >=3 reads / >=5% frequency thresholds are adjustable; overly strict settings will drop genuine low-coverage modification sites.

## Evidence

- [intro] Modkit pileup with custom filter: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] Open-chromatin via modkit 0.5+: [evidence span withheld — anchor status: external_doc]
- [readme] filterbed.py thresholds: [evidence span withheld — anchor status: external_doc]
- [discussion] Per-chromosome memory reduction: "Reduced annotation memory use by processing one chromosome at a time."
- [methods] Filter criteria from source card: [evidence span withheld — anchor status: not_found]
