---
name: line1-transduction-source-inference
description: Use when nanomonsv has produced insertion calls (nanomonsv.result.txt)
  with full inserted sequences from paired tumor/control long-read data and you need
  to classify them as LINE1/Alu/SVA or identify LINE1 3' transduction source sites.
when_to_use_negative:
- The goal is somatic copy-number or balanced rearrangement detection rather than
  insertion classification — use the nanomonsv Canonical SV or Single breakend SV
  modules instead.
- Insertions are present in only a minority of cells (subclonal) at modest sequencing
  depth — the paper notes detection of such subclonal MEIs remains challenging.
- The event is a long VNTR/microsatellite repeat expansion — the paper states the
  reference-genome-based framework may not appropriately capture long repeat expansion
  events.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: nanomonsv
  role: Provides the canonical SV/insertion calls with full-length inserted sequences
    and the insert_classify subcommand that performs MEI classification and 3' transduction
    source inference
  repo: github.com/friend1ws/nanomonsv
- name: RepeatMasker
  role: Annotates residual inserted sequences against repeat elements; LINE1/Alu/SVA
    assigned only when the element covers at least 80% of the inserted sequence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:5e67db7d1653df624c244e0588e41742ee8282df2982a665ceb3de1fdbcd25d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RepeatMasker
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# LINE1 transduction source inference

## Summary

Classifies somatic mobile element insertions detected by nanomonsv into LINE1, Alu, and SVA classes and traces 3' transduction events (partnered vs orphan) back to putative LINE1 source elements, using full-length inserted sequences recovered from tumor/control paired long-read data.

## When to use

Use when nanomonsv has produced insertion calls (nanomonsv.result.txt) with full inserted sequences from paired tumor/control long-read data and you need to classify them as LINE1/Alu/SVA or identify LINE1 3' transduction source sites.

## When NOT to use

- The goal is somatic copy-number or balanced rearrangement detection rather than insertion classification — use the nanomonsv Canonical SV or Single breakend SV modules instead.
- Insertions are present in only a minority of cells (subclonal) at modest sequencing depth — the paper notes detection of such subclonal MEIs remains challenging.
- The event is a long VNTR/microsatellite repeat expansion — the paper states the reference-genome-based framework may not appropriately capture long repeat expansion events.

## Inputs

- nanomonsv.result.txt canonical somatic SV/insertion result files (e.g. from COLO829, H2009, HCC1954 tumor samples)
- GRCh38 (GDC GRCh38.d1.vd1) reference FASTA
- Gencode basic annotation GTF
- LINE1 database files from resource/LINE1_db (5228 primate-specific reference full-length LINE1 elements, plus 652 1000 Genomes Phase 3 and 2610 gnomAD v2.1 non-reference full-length LINE1 insertions)

## Outputs

- Per-sample MEI classification table (LINE1, Alu, SVA, processed pseudogene) with 5' truncation, internal inversion, polyA/polyT tail, and target site duplication annotations
- 3' transduction event list with partnered/orphan status and inferred source LINE1 elements
- Aggregated summary table of MEI class counts and transduction statistics

## How to apply

Run 'nanomonsv insert_classify' on each nanomonsv.result.txt against the GRCh38 (GDC GRCh38.d1.vd1) reference FASTA, supplying the Gencode basic annotation GTF and the LINE1 database files from resource/LINE1_db. First classify insertions whose sequence significantly matches a transcript as processed pseudogenes; for the remainder, apply RepeatMasker annotation and call LINE1, Alu, or SVA when the corresponding element covers at least 80% of the inserted sequence, recording inversion status, polyA/polyT tails, and target site duplications. Realign residual inserted sequences to the reference genome to detect 3' transductions and separate partnered from orphan events. Infer source LINE1 elements by searching upstream among the 5228 full-length primate-specific reference LINE1 elements plus 652 (1000 Genomes Phase 3) and 2610 (gnomAD v2.1) non-reference full-length LINE1 insertions. Aggregate per-sample outputs into a summary table of MEI class counts and transduction statistics.

## Related tools

- **nanomonsv** (Provides the canonical SV/insertion calls with full-length inserted sequences and the insert_classify subcommand that performs MEI classification and 3' transduction source inference) — github.com/friend1ws/nanomonsv
- **RepeatMasker** (Annotates residual inserted sequences against repeat elements; LINE1/Alu/SVA assigned only when the element covers at least 80% of the inserted sequence)

## Examples

```
nanomonsv insert_classify nanomonsv.result.txt GRCh38.d1.vd1.fa gencode.basic.gtf resource/LINE1_db --prefix H2009
```

## Evaluation signals

- Reproduced class counts match the reported totals: 509 insertions overall (492 from H2009), 323 LINE1 and 15 Alu insertions across the three cell-line pairs.
- Transduction statistics match: 107 somatic 3' transduction events (61 partnered, 46 orphan) attributable to 33 putative LINE1 source elements.
- Each LINE1/Alu/SVA classification is supported by a RepeatMasker element covering at least 80% of the inserted sequence.
- Transduction source candidates map upstream of the insertion among the reference (5228) and non-reference (652 + 2610) full-length LINE1 element databases.
- Processed pseudogene calls are only made for insertions whose sequence significantly matches a Gencode transcript.

## Limitations

- Interpretation of detailed structure and properties of complex SVs is not fully automated; much of the work is done manually.
- Detection of subclonal MEIs present in a minority of cells is still challenging with a modest sequencing depth.
- The reference-genome-based framework may not appropriately capture long repeat expansion events because the reference genome is unreliable at such loci.
- The Single breakend module cannot detect SVs where both breakpoints lie in poorly characterized highly repetitive regions.

## Evidence

- [abstract] we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3': "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3'"
- [intro] the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties: "the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties"
- [readme] nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data"
