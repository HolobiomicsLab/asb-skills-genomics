---
name: upset-intersection-statistics
description: Use when a long-read isoform catalog must be benchmarked against reference
  annotations (RefSeq, UCSC Genes mm10 GTF) to compute per-database isoform recovery
  fractions and intersection counts for UpSet plots.
when_to_use_negative:
- The goal is discovery of novel isoforms rather than benchmarking catalog completeness
  against existing annotations
- Transcripts are shorter than ~4.5 kb or rare, since the size selection and library
  preparation may have caused them to be absent from the catalog, biasing recovery
  estimates
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SQANTI
  role: Provides the full-splice-match criteria used to classify isoforms against
    RefSeq and UCSC intron chains
- name: IsoPops
  role: R package that compiles and filters the isoform database (filter_db with exon_min_count
    = 4, filter_truncations, 95% abundance cutoff) that serves as the classification
    input
  repo: https://github.com/kellycochran/IsoPops
- name: R
  role: Language in which the database is loaded with readRDS, intron chains compared,
    and intersection fractions tallied
- name: dplyr
  role: Data manipulation for per-database match-label tallying
- name: stringr
  role: String handling when parsing splice junctions and intron chains
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:902502114477a92e8f15edaecd780b070ad6cd3f374c59102cffcf489530ed1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kellycochran/IsoPops
  tools:
  - IsoPops
  - R
  - SQANTI
  - dplyr
  - stringr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# upset-intersection-statistics

## Summary

Quantify how completely the lrCaptureSeq PacBio isoform catalog recovers annotated isoforms from NCBI RefSeq and UCSC Genes by assigning per-database full-splice-match labels and tallying set intersections for UpSet-plot statistics.

## When to use

Use when a long-read isoform catalog must be benchmarked against reference annotations (RefSeq, UCSC Genes mm10 GTF) to compute per-database isoform recovery fractions and intersection counts for UpSet plots.

## When NOT to use

- The goal is discovery of novel isoforms rather than benchmarking catalog completeness against existing annotations
- Transcripts are shorter than ~4.5 kb or rare, since the size selection and library preparation may have caused them to be absent from the catalog, biasing recovery estimates

## Inputs

- Filtered IsoPops catalog database (DB_filter_4ex_95perc.db; ≥4 exons, non-truncated, 95% abundance cutoff), loaded with readRDS
- NCBI RefSeq mm10 GTF annotation
- UCSC Genes mm10 GTF annotation

## Outputs

- sqanti_isoform_classification.tsv with per-database (RefSeq, UCSC) match count fields
- Intersection fractions per database (e.g. 53.9% RefSeq / 255 isoforms; 72.3% UCSC / 102 isoforms)
- UpSet-plot intersection statistics

## How to apply

Load the filtered IsoPops catalog database (DB_filter_4ex_95perc.db, filtered to ≥4 exons, non-truncated, 95% abundance cutoff) with readRDS. Parse each annotated transcript's intron chain from NCBI RefSeq and UCSC Genes mm10 GTF annotations, and extract donor/acceptor splice junctions from every catalog isoform. Classify isoforms against both annotations under SQANTI full-splice-match criteria, assigning per-database match labels (RefSeq, UCSC) to each isoform. Tally the set intersections for UpSet-plot statistics and write the classification table as sqanti_isoform_classification.tsv with per-database match count fields and intersection fractions. In this study the catalog recovered 53.9% of NCBI RefSeq isoforms (255 isoforms) and 72.3% of UCSC Genes isoforms (102 isoforms).

## Related tools

- **SQANTI** (Provides the full-splice-match criteria used to classify isoforms against RefSeq and UCSC intron chains)
- **IsoPops** (R package that compiles and filters the isoform database (filter_db with exon_min_count = 4, filter_truncations, 95% abundance cutoff) that serves as the classification input) — https://github.com/kellycochran/IsoPops
- **R** (Language in which the database is loaded with readRDS, intron chains compared, and intersection fractions tallied)
- **dplyr** (Data manipulation for per-database match-label tallying)
- **stringr** (String handling when parsing splice junctions and intron chains)

## Examples

```
DB_filter_4ex_95perc <- readRDS("DB_filter_4ex_95perc.db")
```

## Evaluation signals

- sqanti_isoform_classification.tsv exists and contains per-database match count fields for RefSeq and UCSC
- Reported recovery fractions fall in the expected ranges from this study: 53.9% of NCBI RefSeq isoforms (255) and 72.3% of UCSC Genes isoforms (102)
- Every catalog isoform in the filtered database (≥4 exons, non-truncated, 95% abundance) received a match label for each of the two annotation databases
- Intron-chain comparisons were performed under SQANTI full-splice-match criteria using donor/acceptor splice junctions extracted from every isoform
- Intersection counts reproduce the UpSet-plot statistics of the UpSet comparison

## Limitations

- Some transcripts may be missing from the catalogs entirely; absence of an annotated isoform cannot be excluded, so recovery fractions are lower bounds
- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation protocol
- The catalogs may lack certain short and/or rare transcripts, biasing the intersection tallies

## Evidence

- [intro] Annotation-comparison rationale: "information about the number and the identity of CNS mRNA isoforms remains surprisingly scarce—even within the major transcriptome annotation databases"
- [methods] Filtered-database construction: "### Step 3: Optionally, Filter Database By Exon Count, Abundance, or Truncated Transcripts"
- [methods] Saving/loading filtered database: "### Saving and Loading Databases"
- [discussion] Catalog completeness caveat: "Thus, while the catalogs may lack certain short and/or rare transcripts, we conclude that we have detected most of the isoforms expressed in our targeted tissues."
- [readme] Tool provenance: "## IsoPops
R package for analysis of isoform-diverse transcript datasets"
