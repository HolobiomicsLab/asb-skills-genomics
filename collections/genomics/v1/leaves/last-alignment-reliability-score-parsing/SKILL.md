---
name: last-alignment-reliability-score-parsing
description: 'Use when scoring MinION SNP candidates by local alignment quality: LAST
  alignments and a candidate SNP table (alternate-nucleotide coverage >= 20% of reads,
  >= 30% of max contig coverage) are available and a p-error column must be attached.'
when_to_use_negative:
- Input reads have not yet been aligned with LAST — reliability symbols do not exist
  before the mapping stage of the NanoPipe pipeline
- The candidate table is empty or candidates did not pass the 20% alternate-nucleotide
  / 30% maximum-coverage thresholds, so there is nothing to p-error-score
- The goal is metagenomics analysis — NanoPipe does not yet support it and instead
  targets polymorphism assessment of MinION reads against a defined target
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: LAST
  role: Sequence aligner (version 946) whose output supplies the per-base ASCII reliability
    scores consumed by the p-error calculation
- name: NanoPipe
  role: Pipeline step that collects aligned bases around each SNP candidate, applies
    the p-error formula, and writes calc.nuccounts.1.poly
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:1ff1d5ae0b37d315b04711d1693e2a8721a230def9d516a0f57811acdf3633c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# last-alignment-reliability-score-parsing

## Summary

Extracts LAST per-base reliability scores (ASCII quality symbols) from read-to-target alignments and converts them into a p-error score quantifying local alignment quality around each SNP candidate in the NanoPipe polymorphisms table.

## When to use

Use when scoring MinION SNP candidates by local alignment quality: LAST alignments and a candidate SNP table (alternate-nucleotide coverage >= 20% of reads, >= 30% of max contig coverage) are available and a p-error column must be attached.

## When NOT to use

- Input reads have not yet been aligned with LAST — reliability symbols do not exist before the mapping stage of the NanoPipe pipeline
- The candidate table is empty or candidates did not pass the 20% alternate-nucleotide / 30% maximum-coverage thresholds, so there is nothing to p-error-score
- The goal is metagenomics analysis — NanoPipe does not yet support it and instead targets polymorphism assessment of MinION reads against a defined target

## Inputs

- LAST (v946) alignment output containing per-base ASCII reliability/quality symbols for MinION reads mapped to the target
- Weighted SNP candidate table (calc.nuccounts) after the transition/transversion weighted-probability filter

## Outputs

- calc.nuccounts.1.poly: refined polymorphisms table with a p-error column (values 0–1) for every SNP candidate row

## How to apply

Start from the weighted SNP candidate table produced after the transition/transversion weighted-probability filter (transversions weighted 1, transitions 2; alternate-nucleotide coverage >= 20% of all reads and >= 30% of maximum contig coverage). From the LAST (version 946) alignment output, extract the LAST reliability scores — ASCII quality symbols — for every base pair of reads aligned against the target. For each SNP candidate, collect aligned bases in a region of up to 10 nucleotides before and after the SNP position (excluding the SNP itself) across all mapped reads, giving i = N(nucleotides around SNP) x N(reads mapped to the evaluated region). Compute p-error = (1/i) * sum of 10^(-(ASCII_value - 33)/10) over the collected bases, then attach the p-error as a quality column and write the refined table to calc.nuccounts.1.poly. A higher p-error (maximum 1) indicates lower alignment quality, flagging candidates potentially driven by local sequencing errors.

## Related tools

- **LAST** (Sequence aligner (version 946) whose output supplies the per-base ASCII reliability scores consumed by the p-error calculation)
- **NanoPipe** (Pipeline step that collects aligned bases around each SNP candidate, applies the p-error formula, and writes calc.nuccounts.1.poly) — https://github.com/IOB-Muenster/nanopipe2

## Evaluation signals

- calc.nuccounts.1.poly exists and contains one p-error entry for every SNP candidate row from the weighted candidate table
- Every p-error value lies between 0 and 1, since the per-base probability 10^(-(ASCII-33)/10) is at most 1 given ASCII_value >= 33 in LAST quality symbols
- Scored region per candidate spans up to 10 nucleotides before and after the SNP position, excluding the SNP itself, across all reads mapped to that region
- Per-candidate i equals N(nucleotides around SNP) x N(reads mapped to the evaluated region), i.e., the denominator scales with both region size and read count
- Directionality check: candidates in low-quality alignment regions receive higher p-error values (toward the maximum of 1)

## Limitations

- p-error only reflects local alignment quality (LAST reliability symbols within ~10 nt of the SNP), not the statistical significance of the polymorphism itself
- A high p-error (approaching the maximum of 1) flags possibly sequencing-error-driven candidates but does not automatically remove them
- NanoPipe's SNP detection statistics are acknowledged as not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned future work
- The formula assumes LAST ASCII quality values are >= 33; behavior with other score encodings is not specified

## Evidence

- [methods] p-error alignment quality assessment: "an assessment of the alignment quality around a potential SNP by estimating a p-error"
- [methods] transition/transversion weighted candidate filter precedes p-error scoring: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [readme] NanoPipe operates on Oxford Nanopore MinION reads against any target: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices."
