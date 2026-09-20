---
name: isoform-catalog-filtering
description: Use when a raw PacBio Iso-Seq isoform annotation (transcript FASTA, abundance
  table, GFF, ORF FASTA) must be reduced to full-length, high-confidence isoforms
  via exon-count, truncation, and abundance filters.
when_to_use_negative:
- The catalog is already filtered (e.g. it is the 4116-isoform fullfilter.gff); re-running
  the abundance cutoff would remove valid low-rank isoforms
- The data are short-read RNA-seq assemblies, not full-length PacBio isoform annotations
  — IsoPops filtering assumes full-length transcript structures with GFF exon boundaries
- The goal is quantifying CAGE/short-read support for existing TSSs rather than curating
  the isoform catalog itself (use read-coverage validation instead)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: IsoPops
  role: R package providing compile_raw_db, process_db, filter_db, filter_truncations,
    filter_top_gff, and write_gff_data for the three-step catalog curation
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Runtime environment in which the IsoPops filtering pipeline is executed and
    databases are saved/loaded with saveRDS/readRDS
- name: dplyr
  role: Imported dependency used by IsoPops for tabular manipulation of isoform metadata
    during filtering
- name: stringr
  role: Imported dependency used by IsoPops for string processing of isoform/transcript
    identifiers
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:797a035e9c50fa6cefb1fbd12fd82cd0399fdb8f949fd83475104d025483353d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BedTools
  - HISAT2
  - IsoPops
  - R
  - dplyr
  - stringr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# isoform-catalog-filtering

## Summary

Curates a raw PacBio lrCaptureSeq isoform database into a high-confidence catalog using the IsoPops three-step filtering rules (minimum exon count, truncation removal, top-abundance cutoff), yielding the paper's final catalog of 4116 isoforms across 30 neural cell-surface genes.

## When to use

Use when a raw PacBio Iso-Seq isoform annotation (transcript FASTA, abundance table, GFF, ORF FASTA) must be reduced to full-length, high-confidence isoforms via exon-count, truncation, and abundance filters.

## When NOT to use

- The catalog is already filtered (e.g. it is the 4116-isoform fullfilter.gff); re-running the abundance cutoff would remove valid low-rank isoforms
- The data are short-read RNA-seq assemblies, not full-length PacBio isoform annotations — IsoPops filtering assumes full-length transcript structures with GFF exon boundaries
- The goal is quantifying CAGE/short-read support for existing TSSs rather than curating the isoform catalog itself (use read-coverage validation instead)

## Inputs

- Transcript sequence FASTA from PacBio Iso-Seq (lrCaptureSeq)
- Per-isoform abundance table
- Isoform annotation GFF (pre-filter, 8287 isoforms)
- Predicted ORF FASTA
- Two-column gene ID/name table (e.g. gene_IDs_and_names.tsv)

## Outputs

- Fully filtered IsoPops Database (saveRDS .rds) with 4116 isoforms across 30 targeted genes
- GFF3 file of the filtered catalog (fullfilter.gff-style output)
- Optional per-gene top-N GFF via filter_top_gff (e.g. fullfilter_top10.gff)

## How to apply

Compile the raw database in R with IsoPops::compile_raw_db from the transcript FASTA, abundance table, GFF, and ORF FASTA, then attach gene identity with process_db using a 2-column gene ID/name table. Apply filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1) to drop isoforms with fewer than 4 non-contiguous genomic segments (exons). Apply filter_truncations to remove truncated transcripts from contiguous exon-boundary subsets, then filter_db(DB, 4, 0.95, recalc_abundances = F) to drop the least abundant 5% of isoforms per gene. Write the fully filtered catalog to GFF3 with write_gff_data (optionally filter_top_gff for top-N per gene). The rationale is that short internal-exon artifacts and truncated amplicons inflate isoform counts, so requiring 4+ exons, removing truncations, and a 95% per-gene abundance floor reduces 8287 pre-filter isoforms to 4116 curated isoforms.

## Related tools

- **IsoPops** (R package providing compile_raw_db, process_db, filter_db, filter_truncations, filter_top_gff, and write_gff_data for the three-step catalog curation) — github.com/kellycochran/IsoPops
- **R** (Runtime environment in which the IsoPops filtering pipeline is executed and databases are saved/loaded with saveRDS/readRDS)
- **dplyr** (Imported dependency used by IsoPops for tabular manipulation of isoform metadata during filtering)
- **stringr** (Imported dependency used by IsoPops for string processing of isoform/transcript identifiers)

## Examples

```
DB_all <- process_db(compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename), gene_ID_table); DB_f <- filter_db(filter_truncations(filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)), 4, 0.95, recalc_abundances = F)
```

## Evaluation signals

- Script runs without error and the output GFF3 file exists in valid GFF3 format
- Isoform row count of the filtered GFF3 is approximately 4116 (from 8287 pre-filter isoforms), within ±2% tolerance
- Every retained isoform has at least 4 non-contiguous genomic segments (exons) per filter_db(exon_min_count = 4)
- Truncated transcripts from contiguous exon-boundary subsets are absent after filter_truncations
- Per-gene isoform retention is consistent with a 95% abundance cutoff applied with recalc_abundances = FALSE

## Limitations

- Some isoforms smaller than 4.5 kb may have evaded detection due to the library-preparation size-selection step, so the catalog cannot be assumed complete
- The catalogs may lack certain short and/or rare transcripts; absence from the catalog cannot be excluded
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so filter yields are not representative of typical genes

## Evidence

- [methods] rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename): "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1): "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [methods] write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff"): "write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")"
- [readme] IsoPops: R package for analysis of isoform-diverse transcript datasets: "R package for analysis of isoform-diverse transcript datasets"
- [discussion] some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step"
