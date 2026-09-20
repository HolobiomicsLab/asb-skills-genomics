---
name: blast-evalue-annotation
description: Use when a long-read (MinION/PacBio) mapping pipeline needs per-alignment
  statistical confidence (E-value, MAPQ) written into a SAM file so alignments can
  be ranked, filtered, or compared against BLAST-like sensitivity benchmarks.
when_to_use_negative:
- Input is a short-read (Illumina) mapping task where standard mapper MAPQ conventions
  suffice and GraphMap's ~14x reference-size hash index (~42 GB peak for human genome)
  is impractical
- You need a published, calibrated P-value/E-value interpretation for variant calling
  rather than a mapper-internal BLAST-style heuristic score
- The alignment step has already been performed by BLAST itself or another tool that
  natively reports E-values
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: GraphMap
  role: Mapper whose five-stage pipeline produces the alignments that are annotated
    with E-value and mapping-quality tags in the output SAM
  repo: https://github.com/isovic/graphmap
- name: BLAST
  role: Reference standard whose E-value statistic and sensitivity GraphMap's annotation
    emulates for ranking alignments
- name: samscripts
  role: Companion scripts for processing/validating the annotated SAM output
  repo: https://github.com/isovic/samscripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:f8c84d2bd33b97832abb827afab858438cd98b69bc0ac1d7d27d8fd9a439dd31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - BLAST
  - GCC
  - GraphMap
  - make
  - samscripts
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# blast-evalue-annotation

## Summary

Annotate GraphMap read alignments with a BLAST-style E-value and a meaningful SAM mapping-quality so downstream users can rank and filter nanopore mapping results. GraphMap emits these as custom SAM tags alongside the final alignment.

## When to use

Use when a long-read (MinION/PacBio) mapping pipeline needs per-alignment statistical confidence (E-value, MAPQ) written into a SAM file so alignments can be ranked, filtered, or compared against BLAST-like sensitivity benchmarks.

## When NOT to use

- Input is a short-read (Illumina) mapping task where standard mapper MAPQ conventions suffice and GraphMap's ~14x reference-size hash index (~42 GB peak for human genome) is impractical
- You need a published, calibrated P-value/E-value interpretation for variant calling rather than a mapper-internal BLAST-style heuristic score
- The alignment step has already been performed by BLAST itself or another tool that natively reports E-values

## Inputs

- Final gapped alignments of long error-prone reads (e.g. Oxford Nanopore MinION 1d/2d, PacBio) against a reference genome FASTA
- GraphMap candidate alignment scores (region quality scoring, Eqs. 4-7 of the article)
- Unmapped reads / alignments without a significant match

## Outputs

- graphmap_alignments.sam with per-alignment E-value and mapping-quality annotations (custom tags such as ZE:f)
- Valid-float E-value annotations for unmapped records (instead of Inf)

## How to apply

After GraphMap produces its final gapped alignments (Myers bit-vector refinement of the top-ranked candidates from the LCSk-chaining and L1-regression stages), compute a BLAST-style E-value for each alignment and a mapping quality, and emit them as custom SAM tags in the output file (e.g. the ZE:f tag for E-value). For unmapped reads, the ZE:f E-value must be a valid float (std::numeric_limits<float>::max()) rather than Inf so the SAM file parses correctly. Validate that the emitted SAM parses and that precision of retained alignments stays above the reported >95% mapping precision.

## Related tools

- **GraphMap** (Mapper whose five-stage pipeline produces the alignments that are annotated with E-value and mapping-quality tags in the output SAM) — https://github.com/isovic/graphmap
- **BLAST** (Reference standard whose E-value statistic and sensitivity GraphMap's annotation emulates for ranking alignments)
- **samscripts** (Companion scripts for processing/validating the annotated SAM output) — https://github.com/isovic/samscripts

## Examples

```
./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o graphmap_alignments.sam
```

## Evaluation signals

- The output SAM file parses correctly with pysam/samtools and all records carry a numeric ZE:f E-value tag
- Unmapped records report ZE:f as a large finite float (std::numeric_limits<float>::max()), never Inf or NaN
- Mapping quality values are meaningful (discriminative, not constant), consistent with GraphMap's 'meaningful mapping quality' feature claim
- Alignments ranked by E-value/MAPQ reproduce the reported >95% mapping precision and >98% typical location precision on the processed nanopore reads
- Secondary alignments are emitted without malformed extra new lines, so downstream parsers see one record per alignment line

## Limitations

- GraphMap's E-value and MAPQ are mapper-internal annotations (custom SAM tags), not interchangeable with BLAST E-values computed on the same query/reference pair
- GraphMap requires large-memory machines (~100 GB for the human genome; ~42 GB peak index construction), and FM-index memory reduction was still 'currently under implementation'
- Pre-0.5.2 versions emitted Inf for unmapped-record E-values (ZE:f), breaking strict SAM float validation; use version 0.5.2 or later

## Evidence

- [discussion] GraphMap was the only mapper that exhibited BLAST-like sensitivity, while being orders of magnitude faster than BLAST: "GraphMap was the only mapper that exhibited BLAST-like sensitivity, while being orders of magnitude faster than BLAST"
- [readme] Changed the unmapped / Inf E-value (the ZE:f: tag) to be a valid float value: "Changed the unmapped / Inf E-value (the ZE:f: tag) to be a valid float value (`std::numeric_limits<float>::max()` instead of `Inf`)."
- [readme] Meaningful mapping quality.: [evidence span withheld — anchor status: external_doc]
- [abstract] progressively refines candidate alignments to robustly handle potentially high-error rates: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
