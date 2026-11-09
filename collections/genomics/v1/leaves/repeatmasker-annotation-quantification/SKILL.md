---
name: repeatmasker-annotation-quantification
description: Use when tumor/control long-read data has yielded full-length inserted
  sequences (nanomonsv.result.txt) and you must classify each insertion by mobile
  element class and characterize its structural properties (inversion, transduction
  source, TSD).
when_to_use_negative:
- When the inserted sequence significantly matches a Gencode transcript — those insertions
  should be classified as processed pseudogenes, not RepeatMasker element classes
- For long repeat expansion events (VNTR/microsatellite expansions) — the article
  notes the reference-genome-based framework may not capture them reliably
- When only one breakpoint is detectable and the other lies in a highly repetitive
  region — this is the Single breakend SV module's problem, not MEI classification
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0102
tools:
- name: nanomonsv
  role: Provides the canonical somatic SV result files and the 'insert_classify' command
    that orchestrates pseudogene, RepeatMasker-based MEI classification, and 3' transduction
    source inference
  repo: github.com/friend1ws/nanomonsv
- name: RepeatMasker
  role: Supplies repeat element annotation used to label LINE1, Alu, or SVA when the
    element covers at least 80% of the inserted sequence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:20cfdadca2e3c83d86362b083238ba2e7e5bb4c8f73b8688e444a6f084833adb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RepeatMasker
  - nanomonsv
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# repeatmasker-annotation-quantification

## Summary

Classifies somatic mobile element insertions (LINE1, Alu, SVA) from nanomonsv's full-length inserted sequences by applying RepeatMasker annotation plus Gencode transcript matching, and quantifies 5' truncations, internal inversions, polyA/polyT tails, target site duplications, and LINE1 3' transductions.

## When to use

Use when tumor/control long-read data has yielded full-length inserted sequences (nanomonsv.result.txt) and you must classify each insertion by mobile element class and characterize its structural properties (inversion, transduction source, TSD).

## When NOT to use

- When the inserted sequence significantly matches a Gencode transcript — those insertions should be classified as processed pseudogenes, not RepeatMasker element classes
- For long repeat expansion events (VNTR/microsatellite expansions) — the article notes the reference-genome-based framework may not capture them reliably
- When only one breakpoint is detectable and the other lies in a highly repetitive region — this is the Single breakend SV module's problem, not MEI classification

## Inputs

- nanomonsv.result.txt (canonical somatic SV results with full-length inserted sequences, e.g. from COLO829, H2009, HCC1954 tumor/control pairs)
- GRCh38 (GDC GRCh38.d1.vd1) reference genome FASTA
- Gencode basic annotation GTF
- LINE1 database files from nanomonsv resource/LINE1_db
- RepeatMasker annotation of the reference genome

## Outputs

- Per-insertion MEI class labels (LINE1, Alu, SVA, processed pseudogene, other)
- Per-insertion structural annotations: 5' truncation status, internal inversion status, polyA/polyT tail, target site duplication
- 3' transduction calls (partnered vs orphan) and inferred source LINE1 elements
- Aggregate summary table of MEI class counts and transduction statistics

## How to apply

Obtain the nanomonsv canonical SV result files (nanomonsv.result.txt) for the tumor samples (e.g. COLO829, H2009, HCC1954) and run 'nanomonsv insert_classify' against the GRCh38 (GDC GRCh38.d1.vd1) reference FASTA, the Gencode basic annotation GTF, and the LINE1 database files from resource/LINE1_db. First classify insertions whose sequence significantly matches a transcript as processed pseudogenes using the Gencode annotation; only pass the residual insertions to RepeatMasker. Assign LINE1, Alu, or SVA class only when the corresponding element covers at least 80% of the inserted sequence, simultaneously recording inversion status, polyA/polyT tails, and target site duplications. Realign residual inserted sequences to the reference to identify LINE1 3' transductions (partnered vs orphan) and infer source LINE1 elements by searching upstream among the 5228 full-length primate-specific reference LINE1 elements plus the 652 (1000 Genomes Phase 3) and 2610 (gnomAD v2.1) non-reference full-length LINE1 insertions. Finally aggregate per-sample outputs into a summary table of MEI class counts and transduction statistics for comparison with published figures.

## Related tools

- **nanomonsv** (Provides the canonical somatic SV result files and the 'insert_classify' command that orchestrates pseudogene, RepeatMasker-based MEI classification, and 3' transduction source inference) — github.com/friend1ws/nanomonsv
- **RepeatMasker** (Supplies repeat element annotation used to label LINE1, Alu, or SVA when the element covers at least 80% of the inserted sequence)

## Examples

```
nanomonsv insert_classify --result_file nanomonsv.result.txt --reference_GRCh38 GRCh38.d1.vd1.fa --gtf gencode.basic.annotation.gtf --LINE1_db resource/LINE1_db
```

## Evaluation signals

- The 80% coverage rule: each LINE1/Alu/SVA assignment must have the corresponding element covering ≥80% of the inserted sequence; otherwise the insertion remains unclassified
- Reproduced counts should match the reported 323 LINE1 and 15 Alu insertions, 509 total insertions (492 from H2009), and 107 somatic 3' transduction events (61 partnered, 46 orphan) from 33 putative source LINE1 elements
- Every classified insertion should carry recorded inversion status, polyA/polyT tail presence, and target site duplications
- Transduction source search should cover the full LINE1 database: 5228 full-length primate-specific reference elements plus 652 (1000 Genomes Phase 3) and 2610 (gnomAD v2.1) non-reference full-length LINE1 insertions
- Processed pseudogene calls must precede RepeatMasker classification (transcript match takes precedence)

## Limitations

- Detection of MEIs present in a minority of cells (subclones) is still challenging with a modest sequencing depth
- The reference-genome-based framework may not be appropriate to capture long repeat expansion events
- Interpretation of the detailed structure and properties of complex SVs is not fully automated; much of the work is done manually

## Evidence

- [abstract] MEI classification scope: "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions"
- [abstract] Long-read data revealing SV features: "our approaches applied to cancer long-read sequencing data can reveal various features of somatic SVs"
- [readme] nanomonsv input type: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data."
- [results] Expected MEI class counts: [evidence span withheld — anchor status: not_found]
