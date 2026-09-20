---
name: shannon-diversity-index-calculation
description: Use when you have a filtered IsoPops database of full-length PacBio lrCaptureSeq
  isoforms with per-isoform abundances and need to quantify and rank which genes express
  the most diverse isoform portfolios.
when_to_use_negative:
- The goal is kmer-based isoform clustering or dimensionality reduction (use get_kmer_counts
  with kmer_PCA / kmer_tSNE / cluster_isoforms instead of abundance-based diversity
  ranking)
- Input is short-read RNA-seq data rather than a full-length lrCaptureSeq isoform
  database, since reads <200 bp cannot resolve full-length multi-kilobase transcript
  sequences
- Only overall gene-level expression is needed and no isoform-level abundance partition
  is available
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: IsoPops
  role: R package that compiles, filters, and stores the isoform abundance database
    and supplies the jellyfish_plot abundance visualization that the aggregated profiles
    feed into
  repo: https://github.com/kellycochran/IsoPops
- name: vegan
  role: Provides the Shannon diversity index formula applied to each gene's isoform
    relative abundances
- name: dplyr
  role: Data manipulation for aggregating per-isoform counts into per-gene normalized
    abundance profiles and assembling the ranking table
- name: ggplot2
  role: Visualization of the diversity/abundance rankings
- name: R
  role: Execution environment for the IsoPops companion package and the diversity
    computation
  repo: https://github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:d544e0d3bdf84a81ecfdf62b5b869dcfdb642705a3dfb1b68729bb47ee0a4d22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kellycochran/IsoPops
  tools:
  - IsoPops
  - R
  - dplyr
  - ggplot2
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# shannon-diversity-index-calculation

## Summary

Rank-order genes from an lrCaptureSeq/PacBio isoform catalog by Shannon diversity index computed from per-gene isoform relative abundances, so that isoform-diverse targets (e.g., Nrxn3) can be identified and Lorenz-style jellyfish plots contextualized. Uses the IsoPops R package database and the vegan package's diversity metric.

## When to use

Use when you have a filtered IsoPops database of full-length PacBio lrCaptureSeq isoforms with per-isoform abundances and need to quantify and rank which genes express the most diverse isoform portfolios.

## When NOT to use

- The goal is kmer-based isoform clustering or dimensionality reduction (use get_kmer_counts with kmer_PCA / kmer_tSNE / cluster_isoforms instead of abundance-based diversity ranking)
- Input is short-read RNA-seq data rather than a full-length lrCaptureSeq isoform database, since reads <200 bp cannot resolve full-length multi-kilobase transcript sequences
- Only overall gene-level expression is needed and no isoform-level abundance partition is available

## Inputs

- Filtered IsoPops database RDS file (e.g., DB_filter_4ex_95perc, built with exon_min_count = 4, abund_cutoff = 1, truncations removed, 95% abundance cutoff, recalc_abundances = F)
- gene_ID_table mapping gene IDs to names (e.g., gene_IDs_and_names.tsv)

## Outputs

- Combined gene- and isoform-level ranking CSV (e.g., shannon_abundance_ranking.csv) with Shannon diversity index, isoform count per gene, and normalized abundance ranks
- Optionally, jellyfish_plot-style Lorenz diversity visualizations (ggplot2)

## How to apply

Load a saved IsoPops database (built via compile_raw_db and process_db, then filtered with filter_db using exon_min_count = 4 and abund_cutoff = 1, truncations removed via filter_truncations, and a 95% abundance cutoff applied with recalc_abundances = F) from its RDS file with readRDS. Aggregate per-isoform read counts into per-gene normalized relative-abundance profiles, the same representation consumed by IsoPops' jellyfish_plot Lorenz-style visualization. Compute each gene's Shannon diversity index from its isoform relative abundances using the vegan package's diversity formula, then rank genes descending by Shannon index and isoforms descending by normalized abundance, annotating the number of isoforms per gene. Write the combined gene- and isoform-level table (e.g., shannon_abundance_ranking.csv) with dplyr and optionally visualize with ggplot2. Rationale: Shannon index captures how evenly a gene's overall expression is spread across its isoform portfolio, complementing raw isoform counts in the IsoPops catalog of CNS cell-surface molecule isoforms.

## Related tools

- **IsoPops** (R package that compiles, filters, and stores the isoform abundance database and supplies the jellyfish_plot abundance visualization that the aggregated profiles feed into) — https://github.com/kellycochran/IsoPops
- **vegan** (Provides the Shannon diversity index formula applied to each gene's isoform relative abundances)
- **dplyr** (Data manipulation for aggregating per-isoform counts into per-gene normalized abundance profiles and assembling the ranking table)
- **ggplot2** (Visualization of the diversity/abundance rankings)
- **R** (Execution environment for the IsoPops companion package and the diversity computation) — https://github.com/kellycochran/IsoPops

## Examples

```
DB_filter_4ex <- filter_db(process_db(compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename), gene_ID_table), exon_min_count = 4, abund_cutoff = 1); shannon <- vegan::diversity(isoform_abundance_matrix, index = "shannon")
```

## Evaluation signals

- Nrxn3 is the top-ranked gene by Shannon diversity index with roughly 900 isoforms
- Nrcam is the most abundant gene in the ranked output
- Crb1-B is the most abundant isoform in the dataset
- Output file is a well-formed CSV with both gene-level Shannon ranks and isoform-level abundance ranks present
- Each gene's Shannon index falls in [0, ln(number of isoforms for that gene)], consistent with a proper relative-abundance partition

## Limitations

- Some isoforms smaller than 4.5 kb may have evaded detection due to the library-preparation size-selection step, so short isoforms may be absent from the diversity computation
- The catalogs may lack certain short and/or rare transcripts; absence of an isoform cannot be excluded, which can depress Shannon indices for low-abundance diversity
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so cross-gene diversity comparisons are biased toward this pre-selected panel
- It remains undetermined whether the observed isoform diversity is specific to CNS cell-surface molecules or typical of other gene classes and tissues

## Evidence

- [methods] Shannon index from isoform abundances using vegan: "filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [discussion] Companion R package basis: "we provide a companion R software package for analyzing and visualizing isoform catalogs"
- [readme] IsoPops purpose statement: "R package for analysis of isoform-diverse transcript datasets"
- [abstract] Strategy context for diversity analysis: "Here we introduce a strategy to define complete portfolios of full-length isoforms encoded by individual genes."
- [methods] Filtering pipeline step: "DB_all <- process_db(rawDB, gene_ID_table)"
