---
name: gff3-file-manipulation
description: Use when a filtered or unfiltered IsoPops transcript Database must be
  exported to (or manipulated as) GFF3 genome-annotation format for isoform catalog
  delivery or downstream visualization.
when_to_use_negative:
- The source transcript structures have not yet been compiled/processed into an IsoPops
  Database (compile_raw_db/process_db must run first)
- The target downstream tool requires a different annotation format (e.g. GTF, BED)
  rather than GFF3
- The analysis goal is isoform discovery or quantification rather than exporting existing
  annotation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: IsoPops
  role: R package providing write_gff_data and filter_top_gff to serialize (optionally
    top-N-subsetted) filtered isoform Databases to GFF3
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Execution environment in which the IsoPops GFF3 export pipeline is scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:19b01c4b7cac9f8ac0e4a33d39ac3942d45d81cfdd6d6b95c78286569f1ae355
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - dplyr
  - stringr
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gff3-file-manipulation

## Summary

Writing and manipulating GFF3 isoform annotation files from IsoPops Database objects, converting filtered lrCaptureSeq isoform catalogs into the GFF3 genome-annotation format for downstream reuse.

## When to use

Use when a filtered or unfiltered IsoPops transcript Database must be exported to (or manipulated as) GFF3 genome-annotation format for isoform catalog delivery or downstream visualization.

## When NOT to use

- The source transcript structures have not yet been compiled/processed into an IsoPops Database (compile_raw_db/process_db must run first)
- The target downstream tool requires a different annotation format (e.g. GTF, BED) rather than GFF3
- The analysis goal is isoform discovery or quantification rather than exporting existing annotation

## Inputs

- IsoPops Database object (from compile_raw_db + process_db, optionally filtered with filter_db and filter_truncations)
- Gene ID/name table (e.g. gene_IDs_and_names.tsv)
- Top-N per-gene abundance cutoff (integer argument to filter_top_gff, e.g. 10)
- Output GFF3 file path (e.g. ~/IsoPops_Data/fullfilter_top10.gff)

## Outputs

- GFF3 annotation file of curated isoforms (e.g. fullfilter_top10.gff), ~4116 isoform rows across 30 targeted genes when fully filtered
- Top-N-per-gene GFF3 subset (e.g. top 10 most abundant isoforms per gene)

## How to apply

Load the IsoPops Database (compiled from transcript FASTA, abundance table, GFF, and ORF FASTA via compile_raw_db, then processed against the gene ID/name table with process_db) and apply the isoform filters — filter_db(exon_min_count = 4, abund_cutoff = 1), filter_truncations, then filter_db(DB, 4, 0.95, recalc_abundances = F) to drop the least abundant 5% per gene. Export the result with write_gff_data to produce a GFF3 file; optionally subset first with filter_top_gff to keep only the top N most abundant isoforms per gene before writing. Validation is schema- and count-based: the script must run without error, the output file must exist in GFF3 format, and the isoform row count must match expectations (approximately 4116 isoforms across 30 targeted genes, from 8287 pre-filter isoforms).

## Related tools

- **IsoPops** (R package providing write_gff_data and filter_top_gff to serialize (optionally top-N-subsetted) filtered isoform Databases to GFF3) — github.com/kellycochran/IsoPops
- **R** (Execution environment in which the IsoPops GFF3 export pipeline is scripted)

## Examples

```
write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")
```

## Evaluation signals

- The R script runs without error and the output GFF3 file exists at the specified path
- The output file parses as valid GFF3 format
- Isoform row count of the fully filtered output is approximately 4116 (down from 8287 pre-filter isoforms)
- Each retained isoform has at least exon_min_count = 4 non-contiguous genomic segments and abundance at or above the abundance floor
- For filter_top_gff outputs, at most the requested N (e.g. 10) most abundant isoforms per gene appear in the GFF3

## Limitations

- GFF3 export reflects the curated catalog only; the authors note some short and/or rare transcripts may be missing, and isoforms smaller than 4.5 kb may have evaded detection due to the library-preparation size selection step
- Abundance values are written as-is because recalc_abundances = F is used in the final filter_db call

## Evidence

- [methods] write_gff_data exports a filtered Database to GFF3 with an optional top-N abundance subset: "write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")"
- [methods] IsoPops filtering functions are chained before GFF3 export: "### Step 3: Optionally, Filter Database By Exon Count, Abundance, or Truncated Transcripts"
- [full_text] The pipeline writes the fully filtered database to a GFF3 file and validates the row count: [evidence span withheld — anchor status: not_found]
- [readme] IsoPops is the R package underlying this GFF3 workflow: "R package for analysis of isoform-diverse transcript datasets"
