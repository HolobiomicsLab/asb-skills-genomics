---
name: replicate-modification-site-intersection
description: Use when you have per-replicate modkit-extracted modification BED tables
  from >=2 (typically 3) nanopore dRNA replicates and need to count sites/genes conserved
  across all replicates per modification type.
when_to_use_negative:
- Input is raw POD5 or unfiltered modBAM — the site tables must first be produced
  by the Dogme/modkit extract-and-filter steps (params.minCov, params.perMod thresholds
  via filterbed.py).
- The goal is a statistical differential-modification test between conditions rather
  than strict three-replicate conservation counting of the same genomic site coordinates.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Dogme
  role: Nextflow pipeline that generates the per-replicate filtered modkit BED modification
    calls that feed the intersection analysis; includes report_bed_rep.py for comparing
    per-sample modkit BED outputs across replicate folders
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Extracts and filters modifications from aligned modBAMs into the bedMethyl/BED
    site tables that are intersected across replicates
  repo: https://github.com/nanoporetech/modkit
- name: minimap2
  role: Spliced-maps the Dorado-basecalled direct RNA reads so modification calls
    carry consistent genomic coordinates for cross-replicate intersection
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:9f07e7fb047d9f29d621dca9f095113815203e6c67efe2c29cd5640c3cf8408d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# replicate-modification-site-intersection

## Summary

Quantifies reproducibility of nanopore RNA modification calls by strictly intersecting per-replicate modkit BED site coordinates across three biological replicates (e.g. the three mouse C2C12 dRNA replicates of ENCSR160HKZ) and counting shared sites per modification type (m6A, m5C, inosine, pseudouridine, Nm).

## When to use

Use when you have per-replicate modkit-extracted modification BED tables from >=2 (typically 3) nanopore dRNA replicates and need to count sites/genes conserved across all replicates per modification type.

## When NOT to use

- Input is raw POD5 or unfiltered modBAM — the site tables must first be produced by the Dogme/modkit extract-and-filter steps (params.minCov, params.perMod thresholds via filterbed.py).
- The goal is a statistical differential-modification test between conditions rather than strict three-replicate conservation counting of the same genomic site coordinates.

## Inputs

- Filtered per-replicate modkit BED/bedMethyl modification site tables (BED.gz) for each biological replicate
- Optional gene annotation (GTF) for mapping shared sites to genes
- Optional genomic locus interval (e.g. Pkm coordinates) for region-restricted intersection

## Outputs

- Per-modification counts of sites shared across all three replicates (strict n-way intersection)
- Reproducibility fraction (e.g. shared m6A sites / average per-replicate m6A total)
- Per-modification gene coverage counts from shared sites (e.g. pkm_modification_counts.tsv for a locus-restricted analysis)
- Comparison CSVs of per-sample modkit BED outputs across replicates (e.g. Compare_GRCh38_m6A.csv)

## How to apply

Load the filtered modkit BED modification call sets for each replicate (produced by the Dogme pipeline from Dorado-basecalled, minimap2-mapped direct RNA reads). For each modification class (m6A, m5C, Nm, pseudouridine, inosine), intersect genomic site coordinates across all replicates using a strict n-way intersection (bedtools/pybedtools) so only sites detected in every replicate are retained. Count shared sites per modification and, where a reproducibility fraction is desired, divide the shared m6A count by the average per-replicate total (in the C2C12 data this recovers 96,603 shared m6A sites = 63.2% and 43,476 shared m5C sites). Optionally map shared sites to gene annotations to compute per-modification gene coverage, validating against the reported m6A coverage (12,179 genes, 88.2%) and m5C coverage (8,547 genes, 74.1%). Restricting the same procedure to a locus of interest (e.g. Pkm) yields per-locus shared counts such as 14 m6A, 23 m5C, 16 Nm, 21 pseudouridine, and 12 inosine sites.

## Related tools

- **Dogme** (Nextflow pipeline that generates the per-replicate filtered modkit BED modification calls that feed the intersection analysis; includes report_bed_rep.py for comparing per-sample modkit BED outputs across replicate folders) — https://github.com/mortazavilab/dogme
- **modkit** (Extracts and filters modifications from aligned modBAMs into the bedMethyl/BED site tables that are intersected across replicates) — https://github.com/nanoporetech/modkit
- **minimap2** (Spliced-maps the Dorado-basecalled direct RNA reads so modification calls carry consistent genomic coordinates for cross-replicate intersection)

## Examples

```
python scripts/report_bed_rep.py --input rep1/ rep2/ rep3/  # merge strands per sample and produce per-reference/modification comparison CSVs (e.g. Compare_GRCh38_m6A.csv)
```

## Evaluation signals

- Computed shared counts match the reference integers: 96,603 m6A, 43,476 m5C, 8,829 inosine, 10,055 pseudouridine, 30,320 Nm shared sites across the three C2C12 replicates.
- Shared m6A count divided by the average per-replicate m6A total recovers the reported 63.2% reproducibility figure.
- Gene coverage of shared sites matches reported values: 12,179 genes (88.2%) for m6A and 8,547 genes (74.1%) for m5C.
- Locus-restricted (Pkm) shared-site counts equal 14 m6A, 23 m5C, 16 Nm, 21 pseudouridine, and 12 inosine; per-replicate counts fall within the reported ranges (e.g. 21-29 m6A, 26-31 m5C, 18-24 inosine).
- Intersection strictness check: no shared site is missing from any single replicate's BED coordinate set (three-way, not union or pairwise).

## Limitations

- Strict three-way intersection underestimates true modification conservation when per-replicate detection near thresholds drops borderline sites; counts are conservative by design.
- Results depend on upstream modkit filtering thresholds (params.minCov and params.perMod in filterbed.py), so threshold changes alter shared-site counts.
- Validation is anchored to the ENCSR160HKZ C2C12 myoblast dRNA dataset values; other datasets will not reproduce those exact integers.

## Evidence

- [abstract] We detected 96 603 m6A, 43 476 m5C, 8 829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates: [evidence span withheld — anchor status: not_found]
- [intro] For RNA and DNA, modifications are extracted with modkit and filtered: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] report_bed_rep.py compares per-sample modkit BED outputs across replicate folders: [evidence span withheld — anchor status: external_doc]
- [readme] filterbed.py filters modkit bed outputs by minimum coverage and per-mod thresholds: [evidence span withheld — anchor status: external_doc]
