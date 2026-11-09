---
name: isoform-splice-junction-comparison
description: Use when a long-read isoform catalog (e.g., PacBio/IsoPops DB .rds) must
  be benchmarked against public annotations (RefSeq, UCSC Genes GTF) by intron-chain
  matching to measure database recovery fractions.
when_to_use_negative:
- The isoform catalog has not been filtered (exon count, abundance, truncation) —
  unfiltered catalogs contain truncated/off-target transcripts that inflate mismatch
  counts
- The goal is novel-isoform detection rather than recovery of annotated isoforms —
  splice-junction database matching only labels full-splice-match status against known
  annotations
- Input is short-read RNA-seq without full-length isoform sequences — intron chains
  of multi-kilobase transcripts cannot be resolved from <200 bp reads
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SQANTI
  role: Provides full-splice-match criteria used to label catalog isoforms against
    RefSeq and UCSC intron chains
- name: IsoPops
  role: Compiles and filters the lrCaptureSeq isoform database (filter_db, filter_truncations)
    prior to annotation comparison
  repo: https://github.com/kellycochran/IsoPops
- name: R
  role: Runs readRDS loading of the catalog database and the splice-junction comparison
    workflow
  repo: https://github.com/kellycochran/IsoPops
- name: dplyr
  role: Data manipulation for per-database match labeling and tallying set intersections
  repo: https://github.com/kellycochran/IsoPops
- name: stringr
  role: String handling when parsing donor/acceptor junctions and intron chains
  repo: https://github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:6f60f9521cd02b5208b1b2b768a302fc14c25123c4a6e408b0dfbd29df544fbc
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# isoform-splice-junction-comparison

## Summary

Classifies a PacBio lrCaptureSeq isoform catalog against RefSeq and UCSC Genes mm10 annotations by comparing intron chains (donor/acceptor splice junctions) under SQANTI full-splice-match criteria, quantifying the fraction of each database's annotated isoforms recovered by the PacBio catalog (53.9% RefSeq, 72.3% UCSC Genes).

## When to use

Use when a long-read isoform catalog (e.g., PacBio/IsoPops DB .rds) must be benchmarked against public annotations (RefSeq, UCSC Genes GTF) by intron-chain matching to measure database recovery fractions.

## When NOT to use

- The isoform catalog has not been filtered (exon count, abundance, truncation) — unfiltered catalogs contain truncated/off-target transcripts that inflate mismatch counts
- The goal is novel-isoform detection rather than recovery of annotated isoforms — splice-junction database matching only labels full-splice-match status against known annotations
- Input is short-read RNA-seq without full-length isoform sequences — intron chains of multi-kilobase transcripts cannot be resolved from <200 bp reads

## Inputs

- Filtered IsoPops catalog database (DB_filter_4ex_95perc.db, RDS; ≥4 exons, non-truncated, 95% abundance cutoff)
- NCBI RefSeq mm10 GTF annotation
- UCSC Genes mm10 GTF annotation

## Outputs

- sqanti_isoform_classification.tsv with per-database (RefSeq, UCSC) match count fields per isoform
- Per-database intersection/recovery fractions (RefSeq 53.9% = 255 isoforms; UCSC Genes 72.3% = 102 isoforms) for UpSet-plot statistics

## How to apply

Load the filtered IsoPops catalog database (DB_filter_4ex_95perc.db; isoforms with ≥4 exons, non-truncated, top 95% abundance) with readRDS. Parse NCBI RefSeq and UCSC Genes mm10 GTF annotations to extract each annotated transcript's intron chain, and extract donor/acceptor splice junctions from every catalog isoform. Compare intron chains against both annotations under SQANTI full-splice-match criteria, assigning per-database match labels (RefSeq, UCSC) to each isoform. Tally per-database set intersections to obtain recovery fractions for UpSet-plot statistics (53.9% of 255 RefSeq isoforms; 72.3% of 102 UCSC Genes isoforms) and write the classification as sqanti_isoform_classification.tsv with per-database match count fields and intersection fractions. Rationale: splice-junction-accurate long reads allow direct intron-chain comparison, so full-splice-match status is the correct yardstick for whether the PacBio catalog recovers annotated isoforms.

## Related tools

- **SQANTI** (Provides full-splice-match criteria used to label catalog isoforms against RefSeq and UCSC intron chains)
- **IsoPops** (Compiles and filters the lrCaptureSeq isoform database (filter_db, filter_truncations) prior to annotation comparison) — https://github.com/kellycochran/IsoPops
- **R** (Runs readRDS loading of the catalog database and the splice-junction comparison workflow) — https://github.com/kellycochran/IsoPops
- **dplyr** (Data manipulation for per-database match labeling and tallying set intersections) — https://github.com/kellycochran/IsoPops
- **stringr** (String handling when parsing donor/acceptor junctions and intron chains) — https://github.com/kellycochran/IsoPops

## Examples

```
DB_filter_4ex_95perc <- readRDS("DB_filter_4ex_95perc.db"); DB_filter_4ex_notrunc <- filter_truncations(filter_db(process_db(rawDB, gene_ID_table), exon_min_count = 4, abund_cutoff = 1))
```

## Evaluation signals

- RefSeq recovery fraction equals ~53.9% (255 NCBI RefSeq isoforms detected) in the UpSet intersection
- UCSC Genes recovery fraction equals ~72.3% (102 UCSC Genes isoforms detected) in the UpSet intersection
- Every catalog isoform in sqanti_isoform_classification.tsv carries per-database match count fields for both RefSeq and UCSC
- Filtered database used for comparison matches the DB_filter_4ex_95perc criteria: ≥4 exons, non-truncated, 95% abundance cutoff
- Match labels are consistent with SQANTI full-splice-match criteria (complete donor/acceptor intron chain identity)

## Limitations

- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation, deflating recovery fractions
- The catalogs may lack certain short and/or rare transcripts, so recovery statistics cannot exclude absence of isoforms
- The 30 targeted genes were selected for evidence of transcript diversity, so recovery of annotated isoforms for these genes may not generalize to average genes

## Evidence

- [results] The lrCaptureSeq catalog detects 53.9% of NCBI RefSeq isoforms (255 isoforms) and 72.3% of UCSC Genes isoforms (102 isoforms) in the UpSet intersection comparison.: [evidence span withheld — anchor status: not_found]
- [methods] Filter database by exon count, abundance, or truncated transcripts: "### Step 3: Optionally, Filter Database By Exon Count, Abundance, or Truncated Transcripts"
- [methods] filter_db with exon and abundance cutoffs: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [intro] Short reads cannot resolve full-length isoform sequences: "because typical RNA-seq read lengths are <200 bp, the method is not able to resolve the full-length sequence of multi-kilobase transcripts"
- [discussion] Catalogs may lack certain short and rare transcripts: "Thus, while the catalogs may lack certain short and/or rare transcripts, we conclude that we have detected most of the isoforms expressed in our targeted tissues."
- [discussion] Some isoforms smaller than 4.5 kb may have evaded detection due to size selection: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
- [readme] IsoPops is an R package for analysis of isoform-diverse transcript datasets: "IsoPops
R package for analysis of isoform-diverse transcript datasets"
