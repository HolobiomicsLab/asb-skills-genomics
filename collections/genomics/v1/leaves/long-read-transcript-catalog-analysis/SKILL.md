---
name: long-read-transcript-catalog-analysis
description: Use when you have a filtered long-read isoform database (e.g., IsoPops
  DB with per-isoform read counts) and need to quantify how each gene's expression
  is distributed across its isoforms or rank genes by isoform diversity.
when_to_use_negative:
- 'The input is short-read RNA-seq data alone: typical RNA-seq read lengths are <200
  bp and cannot resolve full-length sequences of multi-kilobase transcripts, so Shannon
  diversity from full-length isoform portfolios cannot be computed.'
- 'The database has not been filtered (rawDB from compile_raw_db without process_db/filter_db/filter_truncations):
  unfiltered off-target and truncated transcripts will distort per-gene abundance
  profiles.'
- 'The goal is cataloging rare or very short (<4.5 kb) isoforms: the size-selection
  step of library preparation may have caused some to evade detection.'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: IsoPops
  role: R package that compiles, filters, and analyzes the lrCaptureSeq isoform database
    and provides jellyfish_plot for Lorenz-style diversity visualization
  repo: github.com/kellycochran/IsoPops
- name: vegan
  role: Provides the diversity formula used to compute the Shannon diversity index
    from per-gene isoform relative abundances
- name: dplyr
  role: Data manipulation for aggregating per-isoform counts into per-gene normalized
    abundance profiles and rank-ordering genes and isoforms
- name: ggplot2
  role: Visualization of the diversity and abundance rankings
- name: R
  role: Language in which the IsoPops companion package and the Shannon diversity
    ranking workflow are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:5c7558dccf4de117f9cfb66a6b5282204a5c24cfef3f3dc9ddd5e6dfebbd3ebe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - dplyr
  - ggplot2
  - vegan
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# long-read-transcript-catalog-analysis

## Summary

Method for quantifying and rank-ordering isoform diversity from a PacBio lrCaptureSeq transcript catalog using the IsoPops R package, including Shannon diversity indexing of per-gene isoform portfolios and abundance ranking of individual isoforms.

## When to use

Use when you have a filtered long-read isoform database (e.g., IsoPops DB with per-isoform read counts) and need to quantify how each gene's expression is distributed across its isoforms or rank genes by isoform diversity.

## When NOT to use

- The input is short-read RNA-seq data alone: typical RNA-seq read lengths are <200 bp and cannot resolve full-length sequences of multi-kilobase transcripts, so Shannon diversity from full-length isoform portfolios cannot be computed.
- The database has not been filtered (rawDB from compile_raw_db without process_db/filter_db/filter_truncations): unfiltered off-target and truncated transcripts will distort per-gene abundance profiles.
- The goal is cataloging rare or very short (<4.5 kb) isoforms: the size-selection step of library preparation may have caused some to evade detection.

## Inputs

- Filtered IsoPops database saved as .rds (e.g., DB_filter_4ex_95perc built with exon_min_count = 4, abund_cutoff = 1, truncations removed, 95% abundance cutoff, recalc_abundances = F)
- gene_IDs_and_names.tsv gene ID table (IsoPops_Data directory)
- Per-isoform abundance counts from lrCaptureSeq (PacBio) targeted sequencing of neural cell-surface molecule genes

## Outputs

- shannon_abundance_ranking.csv — combined gene-level Shannon diversity ranking and isoform-level normalized abundance ranking with isoform counts per gene
- Per-gene normalized isoform abundance profiles suitable for jellyfish_plot Lorenz-style visualization

## How to apply

Load the filtered IsoPops database from its saved RDS file — the reference workflow uses DB_filter_4ex_95perc, built via filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1), removal of truncated transcripts with filter_truncations, and a 95% abundance cutoff. Aggregate per-isoform read counts into per-gene normalized abundance profiles of the kind consumed by IsoPops' jellyfish_plot (Lorenz-style diversity visualization). Compute the Shannon diversity index for every gene from its isoform relative abundances using vegan's diversity formula, then rank genes by Shannon index (descending) and isoforms by normalized abundance (descending), annotating isoform counts per gene. Rationale: typical RNA-seq read lengths (<200 bp) cannot resolve full-length sequences of multi-kilobase transcripts, so Shannon diversity must be computed from full-length PacBio isoform catalogs rather than short-read coverage. Write the combined gene- and isoform-level ranking table to CSV for downstream validation.

## Related tools

- **IsoPops** (R package that compiles, filters, and analyzes the lrCaptureSeq isoform database and provides jellyfish_plot for Lorenz-style diversity visualization) — github.com/kellycochran/IsoPops
- **vegan** (Provides the diversity formula used to compute the Shannon diversity index from per-gene isoform relative abundances)
- **dplyr** (Data manipulation for aggregating per-isoform counts into per-gene normalized abundance profiles and rank-ordering genes and isoforms)
- **ggplot2** (Visualization of the diversity and abundance rankings)
- **R** (Language in which the IsoPops companion package and the Shannon diversity ranking workflow are implemented)

## Examples

```
DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1); DB_filter_4ex_notrunc <- filter_truncations(DB_filter_4ex)
```

## Evaluation signals

- Nrxn3 is the top-ranked gene by Shannon diversity index, with roughly 900 isoforms in the filtered database.
- Nrcam is the most abundant gene in the filtered dataset.
- Crb1-B is the most abundant single isoform in the dataset.
- The output CSV is well-formed and contains both gene-level Shannon ranks (descending) and isoform-level abundance ranks (descending) with per-gene isoform counts.
- Shannon indices were computed from normalized isoform relative abundances (summing to 1 per gene) using vegan's diversity function.

## Limitations

- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation protocol, biasing per-gene diversity estimates.
- The catalogs may lack certain short and/or rare transcripts; absence of an isoform cannot be excluded.
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so diversity rankings should not be generalized to arbitrary genes.
- It remains undetermined whether the observed isoform diversity is specific to CNS cell-surface molecules or typical of other gene classes and tissues.

## Evidence

- [intro] Typical RNA-seq read lengths cannot resolve full-length isoforms: "because typical RNA-seq read lengths are <200 bp, the method is not able to resolve the full-length sequence of multi-kilobase transcripts"
- [methods] Filtering parameters used to build the database: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [methods] IsoPops database compilation step: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [readme] IsoPops is an R package for isoform-diverse transcript datasets: "R package for analysis of isoform-diverse transcript datasets"
- [discussion] Size-selection bias caveat: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
