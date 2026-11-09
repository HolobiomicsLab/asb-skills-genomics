---
name: exon-annotation-interval-matching
description: Use when a published exon list must be intersected with a genome annotation
  to verify exon categories (e.g. strictly second-first vs 'diverse') before computing
  upfi/dofi ratios or Fupfi distributions.
when_to_use_negative:
- The input is already a per-transcript exon-type table derived from the annotation
  — re-matching exons would be redundant.
- The analysis targets novel unannotated exons or de novo splicing junctions absent
  from the GTF, which cannot be typed by interval matching against Ensembl.
- The goal is quantification of splicing order itself (upfi/dofi counts), for which
  Insplico — not annotation matching — is the tool.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: rtracklayer
  role: Imports the Ensembl GTF as a GRanges object for interval-based exon matching
  repo: https://github.com/liniguez/Insplico_simulations
- name: plyranges
  role: Filters and overlaps GRanges exon intervals by transcript_id and exon_number
  repo: https://github.com/liniguez/Insplico_simulations
- name: dplyr
  role: Joins exon annotations to Insplico count tables and filters exons by upfi+dofi
    coverage >= 10
- name: Insplico
  role: Produces the upfi/dofi counts that are intersected with the exon-type annotations
  repo: https://gitlab.com/aghr/insplico
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:029ba2821d37e4a602ecb8f92c774702e9cb56d1313197a916863e4549b83972
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liniguez/Insplico_simulations
  tools:
  - Insplico
  - R
  - STAR
  - cutadapt
  - dplyr
  - ggplot2
  - plyranges
  - rtracklayer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# exon-annotation-interval-matching

## Summary

Match user-supplied or study-defined exon sets (e.g. Kim et al.'s 17,273 second-first exons) against an Ensembl GTF annotation to classify each exon by its exon type across transcripts (second-first, second-last, internal, or 'diverse'), which is essential for reconciling AISO profiles between Insplico quantification and published re-analyses.

## When to use

Use when a published exon list must be intersected with a genome annotation to verify exon categories (e.g. strictly second-first vs 'diverse') before computing upfi/dofi ratios or Fupfi distributions.

## When NOT to use

- The input is already a per-transcript exon-type table derived from the annotation — re-matching exons would be redundant.
- The analysis targets novel unannotated exons or de novo splicing junctions absent from the GTF, which cannot be typed by interval matching against Ensembl.
- The goal is quantification of splicing order itself (upfi/dofi counts), for which Insplico — not annotation matching — is the tool.

## Inputs

- Ensembl GTF annotation (e.g. Homo_sapiens.GRCh38.85.gtf, optionally filtered to transcript_support_level 1)
- Chromosome FASTA files chr1-chrY (GRCh38.p7, NCBI assembly GCA_000001405.22) in a 'chroms' folder
- BED/tabular list of exon coordinates or exon identifiers from a published study (e.g. Kim et al. second-first exons)
- Insplico output table of per-exon upfi, dofi, bus, bos counts

## Outputs

- Per-exon annotation of exon type (second-first, second-last, internal, 'diverse') across all Ensembl transcripts
- Filtered subset of exons that are strictly second-first in all transcripts
- Upfi/dofi ratio table and Fupfi histograms (ggplot2) for matched exons meeting the coverage threshold

## How to apply

Import the exon annotation (e.g. Ensembl v75 / GRCh38 GTF filtered to transcript_support_level 1) with rtracklayer::import() in R, converting it to a GRanges object. For each exon of interest, retrieve all transcripts containing it via interval overlap and inspect the exon_number / transcript context to assign an exon type; exons appearing as second-first in only a subset of transcripts fall into the 'diverse' category. Restrict downstream AISO comparison (upfi+dofi >= 10 coverage filter, per-exon upfi/dofi ratio) to strictly annotated second-first exons — this restriction recovered an excess of upfi AISO (ratio upfi/dofi = 1.21), resolving the discrepancy with Kim et al.'s second-first exon profile. Coordinate matching should respect the chr-prefixed sequence names produced by the README's perl renaming step, and strand-aware GRanges operations are used throughout.

## Related tools

- **rtracklayer** (Imports the Ensembl GTF as a GRanges object for interval-based exon matching) — https://github.com/liniguez/Insplico_simulations
- **plyranges** (Filters and overlaps GRanges exon intervals by transcript_id and exon_number) — https://github.com/liniguez/Insplico_simulations
- **dplyr** (Joins exon annotations to Insplico count tables and filters exons by upfi+dofi coverage >= 10)
- **Insplico** (Produces the upfi/dofi counts that are intersected with the exon-type annotations) — https://gitlab.com/aghr/insplico

## Examples

```
Rscript -e 'library(rtracklayer); exons <- import("Hsa38_only_transcript_support_level_1.gtf"); kim_exons <- rtracklayer::import("kim_second_first_exons.bed"); hits <- findOverlaps(kim_exons, exons)'
```

## Evaluation signals

- Every exon in the published list receives a non-missing exon-type assignment after GTF overlap (strict second-first, second-last, internal, or diverse).
- Restricting to strictly annotated second-first exons and filtering upfi+dofi >= 10 yields a median upfi/dofi ratio consistent with ~1.21 (excess upfi), matching Insplico's own results.
- Sequence names match the chr-prefixed convention (chr1-chrY) produced by the README's GTF/FASTA preprocessing, so no dropped intersections due to '1' vs 'chr1' mismatches.
- Fupfi histograms rendered with ggplot2 cover all exons meeting the coverage threshold, and the resulting AISO pattern recapitulates the previously reported profiles.

## Limitations

- A published 'second-first exon' set may be heterogeneous: the majority of Kim et al.'s 17,272 exons fell in the 'diverse' category rather than being strictly second-first in all transcripts, which biases aggregate AISO profiles.
- Insplico quantification with short reads is limited by read/fragment length and provides little splicing-order information beyond the introns directly flanking the exon of interest.
- A coordinate-name mismatch (Ensembl-style '1' vs UCSC-style 'chr1') silently drops overlaps unless the GTF/FASTA names are normalized first.

## Evidence

- [other] Match the 17,273 Kim second-first exons to Ensembl exon types to show that 'diverse'-type exons explain the discrepancy between the profiles.: [evidence span withheld — anchor status: not_found]
- [readme] grep '^chr.+\texon\t' -P Hsa38_only_transcript_support_level_1.gtf | awk '{ print $10,$14,$20;}'| sed 's/[,\;]//g' | sort | uniq -c: "grep '^chr.+\texon\t' -P Hsa38_only_transcript_support_level_1.gtf | awk '{ print $10,$14,$20;}'| sed 's/[,\;]//g' | sort | uniq -c"
- [methods] my_obj <- import(gtf_file): "my_obj <- import(gtf_file)"
- [readme] transcript_ofI <- my_obj %>% filter(transcript_id %in% rownames(numexo_rand_subset) & type %in% c("exon")) %>% split(f = .$transcript_id): "transcript_ofI<-my_obj %>%
      filter(transcript_id %in% rownames(numexo_rand_subset) & type %in% c("exon")) %>%
      split(f = .$transcript_id)"
