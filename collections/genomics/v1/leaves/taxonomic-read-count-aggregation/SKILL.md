---
name: taxonomic-read-count-aggregation
description: Use when Kraken2 per-sample report files (.kreport2.txt) must be parsed
  to extract exact read counts for watchlist taxids (e.g., E. coli 562, S. aureus
  1280, B. anthracis 1392, Y. pestis 632, C. botulinum 1491) feeding downstream validation.
when_to_use_negative:
- You need clade-wide abundance percentages — the 'cumul_reads' / '%' columns of the
  Kraken2 report, not this direct-read extraction, are the wrong tool since this skill
  deliberately excludes descendant reads.
- Input is raw Kraken2 classified-read output (kraken2 output lines) rather than report
  files (.kreport2.txt); the parsing assumes the six-column report schema.
- Analysis requires genus- or family-level counting — this skill retains only species-level
  rank codes (S, S1, S2, S3).
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Nanometa Live
  role: Hosts the get_species_read_counts parsing that tracks species of interest
    from Kraken2 output and surfaces watchlist detections
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Kraken2
  role: Produces the per-sample .kreport2.txt report files parsed by this aggregation
    step
  repo: https://github.com/FOI-Bioinformatics/nanometanf
- name: BLAST
  role: Downstream validation applied to the detections surfaced by this species-of-interest
    read extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:f12cee53e776febcfe26217d94740fcdfc704c44b626ff9039e50a611d679a85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - BLAST
  - Kraken2
  - Nanometa Live
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# taxonomic-read-count-aggregation

## Summary

Aggregates per-sample, per-taxon read counts for user-defined species of interest from Kraken2 report files (.kreport2.txt) so Nanometa Live can surface pathogen watchlist detections with direct read assignments rather than clade totals.

## When to use

Use when Kraken2 per-sample report files (.kreport2.txt) must be parsed to extract exact read counts for watchlist taxids (e.g., E. coli 562, S. aureus 1280, B. anthracis 1392, Y. pestis 632, C. botulinum 1491) feeding downstream validation.

## When NOT to use

- You need clade-wide abundance percentages — the 'cumul_reads' / '%' columns of the Kraken2 report, not this direct-read extraction, are the wrong tool since this skill deliberately excludes descendant reads.
- Input is raw Kraken2 classified-read output (kraken2 output lines) rather than report files (.kreport2.txt); the parsing assumes the six-column report schema.
- Analysis requires genus- or family-level counting — this skill retains only species-level rank codes (S, S1, S2, S3).

## Inputs

- Per-sample Kraken2 report files matching the glob results/kraken2/*.kreport2.txt (tab-separated, no header)
- User-defined species-of-interest watchlist of NCBI taxids (e.g., 562, 1280, 1350, 1392, 632, 1491)

## Outputs

- species_of_interest_reads.csv with minimum fields taxid, name, rank, reads, sample
- Per-sample read counts per watchlist taxid used for downstream validation views

## How to apply

Glob every per-sample Kraken2 report matching results/kraken2/*.kreport2.txt and load each with pandas as a headerless tab-separated frame with columns '%', 'cumul_reads', 'reads', 'rank', 'taxid', 'name'. Filter each frame to species-level rank codes ('S', 'S1', 'S2', 'S3') so subspecies entries count as species-level detections, then intersect the retained rows with the species-of-interest watchlist by taxid. Record the direct 'reads' column — never 'cumul_reads' — so descendant reads are not double counted, and write the aggregated per-sample, per-taxon table to species_of_interest_reads.csv with at least taxid, name, rank, reads, and sample fields. This get_species_read_counts-style parsing feeds Nanometa Live's species-of-interest tracking and downstream BLAST validation views.

## Related tools

- **Nanometa Live** (Hosts the get_species_read_counts parsing that tracks species of interest from Kraken2 output and surfaces watchlist detections) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Kraken2** (Produces the per-sample .kreport2.txt report files parsed by this aggregation step) — https://github.com/FOI-Bioinformatics/nanometanf
- **BLAST** (Downstream validation applied to the detections surfaced by this species-of-interest read extraction)

## Examples

```
df = pd.read_csv('results/kraken2/sample.kreport2.txt', sep='\t', header=None, names=['%','cumul_reads','reads','rank','taxid','name'])
```

## Evaluation signals

- species_of_interest_reads.csv exists and contains a taxid field
- Every row's rank value lies within the species-level rank codes {S, S1, S2, S3}
- Extracted taxa match watchlist entries by taxid (e.g., 1392, 632, 1491)
- Read counts equal the direct 'reads' column, not 'cumul_reads', so no descendant reads are double counted
- One row per matched taxon per sample, with the sample identifier populated

## Limitations

- The step has no independent benchmark in the article; correctness is accepted qualitatively as extracted taxa matching the watchlist with direct read assignments rather than clade totals.
- Only species-level rank codes are retained, so detections ranked above species (e.g., genus-level-only matches) are missed.
- No tool versions for Kraken2 or BLAST are specified, so exact report-format behavior across Kraken2 versions is unverified.

## Evidence

- [methods] Load every per-sample Kraken2 report matching the glob results/kraken2/*.kreport2.txt with pandas, reading tab-separated files with no header: [evidence span withheld — anchor status: not_found]
- [methods] Record the direct 'reads' value (not 'cumul_reads') for each matched taxon per sample, so descendant reads are not double counted.: [evidence span withheld — anchor status: not_found]
- [methods] confirm the output file exists, contains a taxid field, and that every row's rank value lies within the species-level rank codes {S, S1, S2, S3}: [evidence span withheld — anchor status: not_found]
- [readme] It monitors taxonomic classification output during sequencing, surfaces detections of pathogens of interest, and provides per-sample: "It monitors taxonomic classification output during sequencing, surfaces detections of pathogens of interest, and provides per-sample quality control and validation views."
