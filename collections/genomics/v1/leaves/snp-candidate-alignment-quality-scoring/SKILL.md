---
name: snp-candidate-alignment-quality-scoring
description: 'Use when refining a weighted SNP candidate table from Nanopore MinION
  reads: after the 20%/30% coverage and transition/transversion weighted-probability
  filters, to quantify alignment quality around each candidate via LAST reliability
  scores.'
when_to_use_negative:
- Input is the raw read-to-target alignment without a prior filtered SNP candidate
  table — p-error is a refinement step, not a SNP caller
- Reads were not aligned with LAST, so no ASCII reliability symbols (quality values
  >= 33) are available for the per-base error formula
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: NanoPipe
  role: Pipeline that computes the p-error around each SNP candidate and writes the
    refined polymorphisms table calc.nuccounts.1.poly
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Sequence aligner (version 946) whose per-base reliability scores (ASCII quality
    symbols) feed the p-error formula
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:b7dd4ed4f319cb741a38c442d92e142d2b32b0ca2b32bee94481f1b9a83733cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# snp-candidate-alignment-quality-scoring

## Summary

Scores the local alignment quality around each NanoPipe SNP candidate by computing a p-error from LAST per-base reliability (ASCII quality) symbols, and attaches the score as a column in the polymorphisms table so candidates can be down-weighted when local sequencing errors are high.

## When to use

Use when refining a weighted SNP candidate table from Nanopore MinION reads: after the 20%/30% coverage and transition/transversion weighted-probability filters, to quantify alignment quality around each candidate via LAST reliability scores.

## When NOT to use

- Input is the raw read-to-target alignment without a prior filtered SNP candidate table — p-error is a refinement step, not a SNP caller
- Reads were not aligned with LAST, so no ASCII reliability symbols (quality values >= 33) are available for the per-base error formula

## Inputs

- Weighted SNP candidate table from NanoPipe (post coverage-20%/30% and transition/transversion weighted-probability filtering)
- LAST alignment output with per-base reliability (ASCII quality) symbols for reads mapped to the target

## Outputs

- Refined polymorphisms table (calc.nuccounts.1.poly) with a p-error value (0–1) attached to every SNP candidate row

## How to apply

Start from the weighted SNP candidate table produced after the transition/transversion weighted-probability filter (alternate-nucleotide coverage >= 20% of reads and >= 30% of maximum contig coverage). Extract the LAST reliability scores (ASCII quality symbols) for every aligned base pair of the reads against the target. For each SNP candidate, collect aligned bases in a window of up to 10 nucleotides before and after the SNP position (excluding the SNP itself) across all reads mapped to that region, giving i = N(nucleotides around SNP) x N(reads mapped to the evaluated region). Compute p-error = (1/i) * sum of 10^(-(ASCII_value - 33)/10) over the collected bases, so a higher p-error (maximum 1) indicates lower local alignment quality and lower candidate reliability. Attach the p-error as a quality column and write the refined table to calc.nuccounts.1.poly.

## Related tools

- **NanoPipe** (Pipeline that computes the p-error around each SNP candidate and writes the refined polymorphisms table calc.nuccounts.1.poly) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Sequence aligner (version 946) whose per-base reliability scores (ASCII quality symbols) feed the p-error formula)

## Evaluation signals

- calc.nuccounts.1.poly exists and contains a p-error entry for every SNP candidate row
- Every p-error value lies in the range 0–1, since the per-base term 10^(-(ASCII-33)/10) is at most 1 for ASCII_value >= 33
- Each p-error is computed over i = N(nucleotides around SNP) x N(reads mapped to the evaluated region) bases, using a window of up to 10 nucleotides on each side excluding the SNP itself
- Candidates in low-quality regions show higher p-error values (closer to 1), consistent with lower alignment quality and lower reliability
- Pre-existing columns (coverage fractions, transition/transversion weighting) are preserved unchanged in the refined table

## Limitations

- p-error reflects only local alignment quality around the candidate (up to 10 nt each side); it does not by itself prove a variant is real
- NanoPipe's SNP detection statistics are deliberately simple; more sophisticated statistics for SNP detection and evaluation are planned future work
- A high p-error can only flag lower reliability — it may stem from local sequencing errors rather than distinguishing true variants from artifacts

## Evidence

- [methods] p-error formula: "an assessment of the alignment quality around a potential SNP by estimating a p-error"
- [methods] filtering thresholds context: "a SNP candidate should have read coverage of at least 30% from the maximum coverage in the respective contig"
- [methods] weighting scheme context: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [methods] LAST aligner source of reliability scores: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [readme] NanoPipe scope: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
