---
name: processed-pseudogene-transcript-matching
description: Use when nanomonsv insert_classify returns somatic insertion sequences
  from tumor/control paired long-read data and some inserted sequences match annotated
  transcripts, requiring separation of processed pseudogene insertions from LINE1/Alu/SVA
  mobile element insertions.
when_to_use_negative:
- Input is germline or single-sample long-read data without a matched control — nanomonsv's
  somatic calling and this classification workflow assume tumor/control paired data
- Detecting subclonal MEIs present in a minority of cells, which the article notes
  remains challenging at modest sequencing depth
- Long repeat expansion events (e.g. VNTR/microsatellite expansions), which the reference-genome-based
  framework may not capture reliably
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: nanomonsv
  role: insert_classify subcommand performs transcript matching against the Gencode
    basic annotation to label processed pseudogenes, then hands residual insertions
    to RepeatMasker-based repeat classification
  repo: github.com/friend1ws/nanomonsv
- name: RepeatMasker
  role: annotates only the residual (non-pseudogene) insertions; an element must cover
    at least 80% of the inserted sequence to be called LINE1, Alu, or SVA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:71d6b62a2c1515bcec92f840273c74ea58c3c505d2f5d2ed16d71b66605037a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RepeatMasker
  - nanomonsv
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# processed-pseudogene-transcript-matching

## Summary

Within the nanomonsv mobile element insertion (MEI) classification workflow, inserted sequences whose sequence significantly matches a Gencode transcript are re-labeled as processed pseudogenes before RepeatMasker-based LINE1/Alu/SVA annotation, so retrotransposition counts are not inflated by pseudogene insertions.

## When to use

Use when nanomonsv insert_classify returns somatic insertion sequences from tumor/control paired long-read data and some inserted sequences match annotated transcripts, requiring separation of processed pseudogene insertions from LINE1/Alu/SVA mobile element insertions.

## When NOT to use

- Input is germline or single-sample long-read data without a matched control — nanomonsv's somatic calling and this classification workflow assume tumor/control paired data
- Detecting subclonal MEIs present in a minority of cells, which the article notes remains challenging at modest sequencing depth
- Long repeat expansion events (e.g. VNTR/microsatellite expansions), which the reference-genome-based framework may not capture reliably

## Inputs

- nanomonsv.result.txt canonical somatic insertion result files from paired tumor/control long-read (ONT/PacBio) data
- GRCh38 (GDC GRCh38.d1.vd1) reference FASTA
- Gencode basic annotation GTF
- LINE1 database files from resource/LINE1_db (including 5228 full-length primate-specific reference LINE1s plus 1000 Genomes Phase 3 and gnomAD v2.1 non-reference full-length LINE1s)

## Outputs

- Per-sample insertion classification labels marking processed pseudogene vs LINE1/Alu/SVA insertions
- Per-insertion properties: inversion status, polyA/polyT tails, target site duplications
- Aggregated summary table of MEI class counts and 3' transduction statistics (partnered vs orphan, source LINE1 elements)

## How to apply

Run `nanomonsv insert_classify` on each nanomonsv.result.txt (COLO829, H2009, HCC1954) using the GRCh38 (GDC GRCh38.d1.vd1) reference FASTA, the Gencode basic annotation GTF, and the LINE1 database files from resource/LINE1_db. For insertions whose full inserted sequence significantly matches a Gencode transcript, first classify them as processed pseudogenes using the Gencode annotation. Only the residual insertions are then passed to RepeatMasker annotation, where an element is called LINE1, Alu, or SVA when it covers at least 80% of the inserted sequence, with inversion status, polyA/polyT tails, and target site duplications recorded. The rationale is that processed pseudogenes are reverse-transcribed mRNA insertions that would otherwise be miscounted as MEIs, so transcript matching must precede repeat-mask classification. Finally, aggregate per-sample outputs into a summary table of MEI class counts and transduction statistics.

## Related tools

- **nanomonsv** (insert_classify subcommand performs transcript matching against the Gencode basic annotation to label processed pseudogenes, then hands residual insertions to RepeatMasker-based repeat classification) — github.com/friend1ws/nanomonsv
- **RepeatMasker** (annotates only the residual (non-pseudogene) insertions; an element must cover at least 80% of the inserted sequence to be called LINE1, Alu, or SVA)

## Examples

```
nanomonsv insert_classify nanomonsv.result.txt GRCh38.d1.vd1.fa gencode.basic.annotation.gtf resource/LINE1_db
```

## Evaluation signals

- Reproduced MEI counts should match Figure 4B: 323 LINE1 and 15 Alu insertions across the three cell-line pairs
- Total insertions should reach 509 (492 from H2009), with 107 somatic 3' transduction events (61 partnered, 46 orphan) from 33 putative LINE1 source elements
- Every insertion labeled LINE1/Alu/SVA must have the corresponding element covering at least 80% of the inserted sequence; anything else should have been transcript-matched first
- Insertions classified as processed pseudogenes must show a significant match to a Gencode transcript and must not be double-counted in LINE1/Alu/SVA classes
- Per-sample outputs should aggregate cleanly into a summary table without insertions assigned to more than one class

## Limitations

- Detection of subclonal MEIs present in a minority of cells is still challenging with modest sequencing depth
- Interpretation of the detailed structure and properties of complex SVs is not fully automated; much of the work is done manually
- The Single breakend SV module cannot detect SVs where both breakpoints lie in poorly characterized, highly repetitive regions

## Evidence

- [abstract] we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3': "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3'"
- [intro] the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties: "the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties"
- [full_text] Run 'nanomonsv insert_classify' on each result file with the GRCh38 (GDC GRCh38.d1.vd1) reference FASTA, the Gencode basic annotation GTF: [evidence span withheld — anchor status: not_found]
- [full_text] classify them as LINE1, Alu, or SVA when the corresponding element covers at least 80% of the inserted sequence: [evidence span withheld — anchor status: not_found]
