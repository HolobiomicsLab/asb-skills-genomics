---
name: refseq-annotation-matching
description: Use when a PacBio long-read isoform catalog (e.g., IsoPops-filtered .db)
  must be benchmarked against RefSeq or UCSC Genes GTF annotations to report per-database
  isoform recovery fractions like 53.9% (RefSeq) or 72.3% (UCSC).
when_to_use_negative:
- The transcript evidence is short-read RNA-seq only — typical read lengths <200 bp
  cannot resolve full-length sequences of multi-kilobase transcripts, so full-splice-match
  classification of complete isoforms is not meaningful.
- The isoform catalog has not yet been filtered (e.g., with filter_db/filer_truncations
  for exon count ≥4 and truncation removal) — matching unfiltered or truncated transcripts
  inflates apparent recovery statistics.
- The goal is novel isoform discovery rather than annotation benchmarking — use catalog
  novelty classification instead of database recovery comparison.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SQANTI
  role: Defines full-splice-match criteria for comparing catalog isoform intron chains
    against RefSeq and UCSC Genes annotations
- name: IsoPops
  role: Provides the filtered catalog database (compile_raw_db, process_db, filter_db,
    filter_truncations) that is matched against the annotations
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Runtime environment for loading the catalog with readRDS and running the classification
    workflow
  repo: github.com/kellycochran/IsoPops
- name: dplyr
  role: Data manipulation for tallying per-database match labels and intersection
    statistics
  repo: github.com/kellycochran/IsoPops
- name: stringr
  role: String handling for parsing splice junction coordinates from isoform identifiers
  repo: github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:dd4e8100d30e16340fd4632fe533e728cfb1743a9355ac084320a9938faa1afb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - SQANTI
  - dplyr
  - stringr
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# refseq-annotation-matching

## Summary

Classifies a PacBio lrCaptureSeq isoform catalog against public RefSeq and UCSC Genes mm10 annotations by intron-chain matching under SQANTI full-splice-match criteria, quantifying what fraction of each database's annotated isoforms the targeted long-read catalog recovers.

## When to use

Use when a PacBio long-read isoform catalog (e.g., IsoPops-filtered .db) must be benchmarked against RefSeq or UCSC Genes GTF annotations to report per-database isoform recovery fractions like 53.9% (RefSeq) or 72.3% (UCSC).

## When NOT to use

- The transcript evidence is short-read RNA-seq only — typical read lengths <200 bp cannot resolve full-length sequences of multi-kilobase transcripts, so full-splice-match classification of complete isoforms is not meaningful.
- The isoform catalog has not yet been filtered (e.g., with filter_db/filer_truncations for exon count ≥4 and truncation removal) — matching unfiltered or truncated transcripts inflates apparent recovery statistics.
- The goal is novel isoform discovery rather than annotation benchmarking — use catalog novelty classification instead of database recovery comparison.

## Inputs

- IsoPops-filtered catalog database RDS file (DB_filter_4ex_95perc.db; ≥4 exons, non-truncated, 95% abundance cutoff)
- NCBI RefSeq mm10 GTF annotation
- UCSC Genes mm10 GTF annotation
- IsoPops gene ID table (gene_IDs_and_names.tsv)

## Outputs

- sqanti_isoform_classification.tsv with per-database match labels and match counts for each catalog isoform
- Per-database recovery fractions (RefSeq and UCSC Genes isoform intersection percentages)
- UpSet-plot intersection statistics (matched isoform counts per annotation database)

## How to apply

Load the filtered IsoPops catalog database (DB_filter_4ex_95perc.db, filtered to ≥4 exons, non-truncated transcripts, and a 95% abundance cutoff) with readRDS. Parse the NCBI RefSeq and UCSC Genes mm10 GTF files to extract each annotated transcript's intron chain, and extract donor/acceptor splice junctions from every catalog isoform. Compare intron chains against both annotations under SQANTI full-splice-match criteria (complete donor/acceptor junction agreement), assigning a per-database match label (RefSeq, UCSC) to each isoform. Tally set intersections per database for UpSet-plot statistics and write the classification table as sqanti_isoform_classification.tsv with per-database match counts and intersection fractions. In this study the lrCaptureSeq catalog recovered 53.9% of NCBI RefSeq isoforms (255 isoforms) and 72.3% of UCSC Genes isoforms (102 isoforms).

## Related tools

- **SQANTI** (Defines full-splice-match criteria for comparing catalog isoform intron chains against RefSeq and UCSC Genes annotations)
- **IsoPops** (Provides the filtered catalog database (compile_raw_db, process_db, filter_db, filter_truncations) that is matched against the annotations) — github.com/kellycochran/IsoPops
- **R** (Runtime environment for loading the catalog with readRDS and running the classification workflow) — github.com/kellycochran/IsoPops
- **dplyr** (Data manipulation for tallying per-database match labels and intersection statistics) — github.com/kellycochran/IsoPops
- **stringr** (String handling for parsing splice junction coordinates from isoform identifiers) — github.com/kellycochran/IsoPops

## Examples

```
DB_filter_4ex_95perc <- readRDS("~/IsoPops_Data/DB_filter_4ex_95perc.db")
```

## Evaluation signals

- Every catalog isoform in sqanti_isoform_classification.tsv carries a match label for each annotation database (RefSeq and/or UCSC).
- Matched isoforms share complete intron chains (identical donor/acceptor junction pairs) with annotated transcripts, consistent with SQANTI full-splice-match criteria.
- Recovered counts match expected scale: 255 RefSeq isoforms (53.9%) and 102 UCSC Genes isoforms (72.3%) in the UpSet intersection.
- The classification table includes both per-database match count fields and the computed intersection fractions.
- The input database equals DB_filter_4ex_95perc.db (≥4 exons, non-truncated, 95% abundance cutoff), verified against the IsoPops filtering history.

## Limitations

- We cannot exclude the possibility that certain transcripts are missing from the catalogs, so recovery fractions are lower bounds on annotation coverage of the targeted transcriptome.
- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation protocol, biasing recovery statistics against short annotated isoforms.
- The catalogs may lack certain short and/or rare transcripts, which reduces apparent RefSeq/UCSC recovery even when those isoforms are real.
- Recovery fractions are specific to the 30 targeted genes, which were selected for evidence of transcript diversity and probably have more isoforms than the average gene.

## Evidence

- [abstract] Applying this approach to neural cell-surface molecules, we identify thousands of unannotated isoforms expressed in retina and brain.: "Applying this approach to neural cell-surface molecules, we identify thousands of unannotated isoforms expressed in retina and brain."
- [methods] IsoPops database compilation from transcript, abundance, GFF, and ORF inputs.: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] IsoPops filtering by exon count and abundance.: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [intro] Short-read RNA-seq cannot resolve full-length isoform sequences, motivating long-read annotation matching.: "because typical RNA-seq read lengths are <200 bp, the method is not able to resolve the full-length sequence of multi-kilobase transcripts"
- [discussion] Catalog completeness caveat affecting recovery interpretation.: "However, we cannot exclude the possibility that certain transcripts are missing from the catalogs."
- [readme] README confirming IsoPops package purpose.: "R package for analysis of isoform-diverse transcript datasets"
