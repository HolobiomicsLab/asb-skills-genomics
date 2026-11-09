---
name: size-selection-read-filtering
description: Use when processing INC-Seq consensus reads from ONT MinION 16S amplicon
  data whose alignment proportion or contiguity is low (~73%), and a 1,300–1,450 bp
  full-length 16S size window plus orientation correction is needed before blastn-based
  assessment.
when_to_use_negative:
- Input is raw noisy nanopore reads rather than INC-Seq consensus reads — the article
  applies chopSeq only after INC-Seq consensus generation
- The analysis goal is de novo consensus calling from unclustered reads (use nanoClust
  read partitioning instead)
- Target amplicons are much shorter than 16S rRNA full length, since the 1,300–1,450
  bp selection window would discard most reads
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: chopSeq
  role: Fragments INC-Seq consensus reads, corrects read orientation, and applies
    1,300–1,450 bp size selection (chopSeq_SS)
  repo: github.com/umerijaz/nanopore
- name: blastn
  role: Computes the contiguous aligned proportion of corrected/selected reads against
    reference 16S sequences and the resulting sequence accuracy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:57b902b981f90fa32e3fe6d6f0b1400cfa189e43d2674dc43904cf57f7e36b67
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - blastn
  - chopSeq
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# size-selection-read-filtering

## Summary

Filters and orientation-corrects INC-Seq consensus nanopore 16S rRNA reads with chopSeq, applying a 1,300–1,450 bp size selection so that ~98.4 ± 0.7% of each read aligns contiguously to reference 16S sequences and consensus accuracy reaches ~98 ± 0.23%.

## When to use

Use when processing INC-Seq consensus reads from ONT MinION 16S amplicon data whose alignment proportion or contiguity is low (~73%), and a 1,300–1,450 bp full-length 16S size window plus orientation correction is needed before blastn-based assessment.

## When NOT to use

- Input is raw noisy nanopore reads rather than INC-Seq consensus reads — the article applies chopSeq only after INC-Seq consensus generation
- The analysis goal is de novo consensus calling from unclustered reads (use nanoClust read partitioning instead)
- Target amplicons are much shorter than 16S rRNA full length, since the 1,300–1,450 bp selection window would discard most reads

## Inputs

- INC-Seq consensus reads (nanopore 16S amplicon FASTA/FASTQ) from raw ONT MinION mock-community reads
- Reference 16S rRNA gene sequences (e.g. SILVA-derived references) for blastn evaluation

## Outputs

- chopSeq-corrected, orientation-fixed, 1,300–1,450 bp size-selected read set
- blastn-aligned proportion metrics (expected ~98.4 ± 0.7% after chopSeq_SS)
- consensus sequence accuracy value (expected ~98 ± 0.23%)

## How to apply

Run chopSeq on the INC-Seq consensus reads to perform fragmentation and read orientation correction; without size selection this already raises the blastn contiguous-alignment proportion from ~73 ± 14.6% to ~96 ± 2.3%. Re-run chopSeq with 1,300–1,450 bp size selection (chopSeq_SS mode) to retain full-length 16S rRNA fragments, then recompute the aligned proportion with blastn, expecting ~98.4 ± 0.7%. Finally, compare the size-selected consensus sequences against the reference 16S sequences with blastn to compute sequence accuracy, expecting ~98 ± 0.23%. The rationale is that mis-oriented and non-full-length INC-Seq consensus reads fragment their contiguous alignment to the reference, and a targeted 1,300–1,450 bp window recovers near-full-length 16S reads suitable for downstream de novo clustering (nanoClust).

## Related tools

- **chopSeq** (Fragments INC-Seq consensus reads, corrects read orientation, and applies 1,300–1,450 bp size selection (chopSeq_SS)) — github.com/umerijaz/nanopore
- **blastn** (Computes the contiguous aligned proportion of corrected/selected reads against reference 16S sequences and the resulting sequence accuracy)

## Evaluation signals

- blastn aligned proportion after chopSeq without size selection falls within ~96 ± 2.3%
- blastn aligned proportion after chopSeq_SS falls within ~98.4 ± 0.7%
- Consensus sequence accuracy after chopSeq_SS falls within ~98 ± 0.23% against the reference sequences
- Size-selected reads fall within the 1,300–1,450 bp length window
- All metrics are collected into a metrics.csv table for reproducible comparison against the reported ranges

## Limitations

- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, capping achievable accuracy (~98% after chopSeq_SS; ~99.5% reported for full NanoAmpli-Seq 2D/1D2 consensus)
- Reference-based blastn assessment requires the organism to be present in the reference database; incorrect mapping is likely due to high error rates of raw nanopore reads
- Strict 1,300–1,450 bp size selection discards reads outside the full-length 16S window

## Evidence

- [abstract] chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [results] accuracy increased to 98 ± 0.23% with 98.4 ± 0.7% aligned: [evidence span withheld — anchor status: not_found]
- [abstract] nearly all residual errors originate from homopolymer deletions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [readme] repository scope: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
