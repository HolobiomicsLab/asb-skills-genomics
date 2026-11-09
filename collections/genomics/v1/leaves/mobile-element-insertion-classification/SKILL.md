---
name: mobile-element-insertion-classification
description: Use when full-length somatic insertion sequences from nanomonsv canonical
  SV results (tumor/control paired ONT data) need to be assigned to LINE1/Alu/SVA/pseudogene
  classes and characterized for 3' transductions.
when_to_use_negative:
- The goal is SV detection rather than classification of already-called insertions
  — use the nanomonsv Canonical SV or Single breakend SV modules instead
- SVs of interest are balanced rearrangements, CNVs, or non-mobile-element events
  rather than inserted sequences — the classifier targets full-length inserted sequences
- The reference genome is unreliable at the locus of interest, e.g. suspected long
  VNTR/microsatellite repeat expansion events, which the article notes the reference-genome-based
  framework may not capture
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: Provides the insert_classify subcommand that consumes nanomonsv.result.txt
    files plus reference/Gencode/LINE1-db resources to classify mobile element insertions
    and their properties
  repo: github.com/friend1ws/nanomonsv
- name: RepeatMasker
  role: Supplies repeat annotation used to classify residual insertions as LINE1,
    Alu, or SVA when the element covers at least 80% of the inserted sequence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:2570f2dc50208dccbeec9c40944685189a3fd99935f841274d7c4a5ae56723a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RepeatMasker
  - nanomonsv
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mobile-element-insertion-classification

## Summary

A nanomonsv post-detection workflow (nanomonsv insert_classify) that classifies somatic mobile element insertions from tumor/control paired ONT long-read data into LINE1, Alu, SVA, and processed pseudogene classes, while characterizing 5' truncations, internal inversions, polyA/polyT tails, target site duplications, and source sites of 3' transductions.

## When to use

Use when full-length somatic insertion sequences from nanomonsv canonical SV results (tumor/control paired ONT data) need to be assigned to LINE1/Alu/SVA/pseudogene classes and characterized for 3' transductions.

## When NOT to use

- The goal is SV detection rather than classification of already-called insertions — use the nanomonsv Canonical SV or Single breakend SV modules instead
- SVs of interest are balanced rearrangements, CNVs, or non-mobile-element events rather than inserted sequences — the classifier targets full-length inserted sequences
- The reference genome is unreliable at the locus of interest, e.g. suspected long VNTR/microsatellite repeat expansion events, which the article notes the reference-genome-based framework may not capture

## Inputs

- nanomonsv.result.txt canonical somatic SV results per tumor sample (COLO829, H2009, HCC1954) from nanomonsv get runs
- GRCh38 (GDC GRCh38.d1.vd1) reference genome FASTA
- Gencode basic annotation GTF
- LINE1 database files from nanomonsv resource/LINE1_db
- RepeatMasker annotation
- Full-length LINE1 source catalogs: 5228 primate-specific reference LINE1s, 652 non-reference LINE1s (1000 Genomes Phase 3), 2610 non-reference LINE1s (gnomAD v2.1)

## Outputs

- Per-sample MEI classification of insertions into LINE1, Alu, SVA, and processed pseudogene classes
- Per-insertion properties: 5' truncation status, internal inversion status, polyA/polyT tails, target site duplications
- LINE1 3' transduction calls with partnered/orphan status and inferred source LINE1 elements
- Aggregated summary table of MEI class counts and transduction statistics (e.g. Figure 4B)

## How to apply

Run 'nanomonsv insert_classify' on each nanomonsv.result.txt (COLO829, H2009, HCC1954 tumor samples) using the GRCh38 (GDC GRCh38.d1.vd1) reference FASTA, the Gencode basic annotation GTF, and the LINE1 database files from resource/LINE1_db. Insertions whose sequence significantly matches a transcript are classified as processed pseudogenes via the Gencode annotation. Remaining insertions are annotated with RepeatMasker and classified as LINE1, Alu, or SVA only when the corresponding element covers at least 80% of the inserted sequence, recording inversion status, polyA/polyT tails, and target site duplications. Residual inserted sequences are realigned to the reference genome to identify LINE1 3' transductions (partnered vs orphan) and infer source LINE1 elements by searching upstream among the 5228 full-length primate-specific reference LINE1 elements plus the 652 (1000 Genomes Phase 3) and 2610 (gnomAD v2.1) non-reference full-length LINE1 insertions. Finally, aggregate per-sample outputs into a summary table of MEI class counts and transduction statistics.

## Related tools

- **nanomonsv** (Provides the insert_classify subcommand that consumes nanomonsv.result.txt files plus reference/Gencode/LINE1-db resources to classify mobile element insertions and their properties) — github.com/friend1ws/nanomonsv
- **RepeatMasker** (Supplies repeat annotation used to classify residual insertions as LINE1, Alu, or SVA when the element covers at least 80% of the inserted sequence)

## Examples

```
nanomonsv insert_classify --result nanomonsv.result.txt --reference GRCh38.d1.vd1.fa --gtf gencode.basic.gtf --line1_db resource/LINE1_db
```

## Evaluation signals

- Reproduce the reported counts of 323 LINE1 and 15 Alu insertions across the three cell-line pairs (Figure 4B)
- Recover 509 total insertions (492 from H2009) and 81 5' inversions in the aggregated summary
- Obtain 107 somatic 3' transduction events split as 61 partnered and 46 orphan, traced to 33 putative LINE1 source elements
- Confirm the ≥80% element-coverage rule gates LINE1/Alu/SVA labels and that transcript-matching insertions are labeled processed pseudogenes
- Verify every LINE1/Alu entry carries recorded inversion status, polyA/polyT tail, and target site duplication annotations

## Limitations

- Detection of subclonal MEIs present in a minority of cells is still challenging with a modest sequencing depth
- The reference-genome-based framework may not be appropriate to capture long repeat expansion events (VNTR/microsatellite) because the reference genome is not reliable at such loci
- Interpretation of detailed structure and properties of complex SVs is not fully automated and much of the work is done manually
- Both 3' transduction source inference and coverage-based classification depend on reference LINE1 catalogs and RepeatMasker, so non-reference sources outside the 5228 + 652 + 2610 element sets may be missed

## Evidence

- [abstract] we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3': "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3'"
- [results] The MEI classification workflow identified 323 LINE1 and 15 Alu insertions in the three cell-lines (Figure 4B).: [evidence span withheld — anchor status: not_found]
- [intro] the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties: "the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties"
