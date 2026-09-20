---
name: long-read-insertion-sequence-characterization
description: Use when tumor/control paired long-read (ONT or PacBio) data contain
  somatic insertion calls whose full-length inserted sequences must be classified
  by mobile element type and characterized for truncation, inversion, transduction,
  and source-element properties.
when_to_use_negative:
- Detecting somatic VNTR/microsatellite repeat expansion events — the article notes
  the reference-genome-based framework may not capture long repeat expansion events
  because the reference is unreliable at susceptible loci
- SVs where both breakpoints lie in poorly characterized highly repetitive regions
  such as centromeres — the Single breakend module cannot detect these
- Subclonal MEIs present in a minority of cells — detection is challenging at modest
  sequencing depth
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: nanomonsv
  role: Runs 'insert_classify' on canonical SV results to classify insertions using
    the GRCh38 reference, Gencode GTF, and LINE1_db resources
  repo: https://github.com/friend1ws/nanomonsv
- name: RepeatMasker
  role: Provides repeat annotation used to label residual insertions as LINE1, Alu,
    or SVA when the element covers at least 80% of the inserted sequence
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform generating the reads from which full-length
    inserted sequences are recovered
- name: PacBio
  role: Long-read sequencing platform generating the reads from which full-length
    inserted sequences are recovered
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:51e96ffe79414537830aed403d9bd61bff5f664f72746f7512aa204bac05136b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - Oxford Nanopore Technologies (ONT)
  - PacBio
  - RepeatMasker
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# long-read-insertion-sequence-characterization

## Summary

A nanomonsv-based workflow that classifies somatic mobile element insertions (LINE1, Alu, SVA, processed pseudogenes) from full-length inserted sequences recovered by paired tumor/control long-read sequencing, and annotates properties such as 5' truncations, internal inversions, polyA tails, target site duplications, and LINE1 3' transduction source sites.

## When to use

Use when tumor/control paired long-read (ONT or PacBio) data contain somatic insertion calls whose full-length inserted sequences must be classified by mobile element type and characterized for truncation, inversion, transduction, and source-element properties.

## When NOT to use

- Detecting somatic VNTR/microsatellite repeat expansion events — the article notes the reference-genome-based framework may not capture long repeat expansion events because the reference is unreliable at susceptible loci
- SVs where both breakpoints lie in poorly characterized highly repetitive regions such as centromeres — the Single breakend module cannot detect these
- Subclonal MEIs present in a minority of cells — detection is challenging at modest sequencing depth

## Inputs

- nanomonsv.result.txt canonical SV result files from tumor samples (e.g. COLO829, H2009, HCC1954)
- GRCh38 (GDC GRCh38.d1.vd1) reference genome FASTA
- Gencode basic annotation GTF
- LINE1 database files from resource/LINE1_db
- RepeatMasker annotation tracks

## Outputs

- Per-sample MEI classification output (LINE1, Alu, SVA, processed pseudogene calls)
- Insertion property annotations: 5' truncations, internal inversions, polyA/polyT tails, target site duplications
- 3' transduction events annotated as partnered or orphan with inferred source LINE1 elements
- Summary table of MEI class counts and transduction statistics

## How to apply

Start from nanomonsv canonical SV result files (nanomonsv.result.txt) from tumor/control paired long-read data, and run 'nanomonsv insert_classify' with the GRCh38 (GDC GRCh38.d1.vd1) reference FASTA, Gencode basic annotation GTF, and the LINE1 database files in resource/LINE1_db. Insertions whose sequence significantly matches a transcript are first classified as processed pseudogenes using the Gencode annotation; the remainder are compared against RepeatMasker annotation and labeled LINE1, Alu, or SVA when the element covers at least 80% of the inserted sequence, recording inversion status, polyA/polyT tails, and target site duplications. Residual inserted sequences are realigned to the reference genome to identify LINE1 3' transductions (partnered vs orphan), and source LINE1 elements are inferred by searching upstream among 5228 full-length primate-specific reference LINE1s plus 652 (1000 Genomes Phase 3) and 2610 (gnomAD v2.1) non-reference full-length LINE1 insertions. Finally aggregate per-sample outputs into a summary table of MEI class counts and transduction statistics.

## Related tools

- **nanomonsv** (Runs 'insert_classify' on canonical SV results to classify insertions using the GRCh38 reference, Gencode GTF, and LINE1_db resources) — https://github.com/friend1ws/nanomonsv
- **RepeatMasker** (Provides repeat annotation used to label residual insertions as LINE1, Alu, or SVA when the element covers at least 80% of the inserted sequence)
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform generating the reads from which full-length inserted sequences are recovered)
- **PacBio** (Long-read sequencing platform generating the reads from which full-length inserted sequences are recovered)

## Examples

```
nanomonsv insert_classify --result_file nanomonsv.result.txt --reference_GRCh38.fa --line1_db resource/LINE1_db --gencode_gtf gencode.basic.annotation.gtf
```

## Evaluation signals

- Reproduced class counts match the reported figures: 323 LINE1 and 15 Alu insertions across the three cell-line pairs (Figure 4B)
- Total insertions reproduce the reported 509, of which 492 are from H2009
- 3' transduction statistics reproduce 107 somatic events (61 partnered, 46 orphan) from 33 putative LINE1 source elements
- Each classified LINE1/Alu/SVA insertion satisfies the ≥80% element-coverage rule over the inserted sequence
- Transduction source inference accounts against the full panel: 5228 reference plus 652 (1000G) plus 2610 (gnomAD) full-length LINE1 elements

## Limitations

- Detection of subclonal MEIs present in a minority of cells is still challenging with a modest sequencing depth
- The current reference-genome-based framework may not be appropriate to capture long repeat expansion events because the reference genome is not reliable at susceptible locations
- Interpretation of the detailed structure and properties of complex SVs is not fully automated, and much of the work is done manually

## Evidence

- [abstract] A workflow classifies mobile element insertions and elucidates properties such as 5' truncations, internal inversions, and source sites for 3' transductions.: "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3'"
- [results] The MEI workflow identified 323 LINE1 and 15 Alu insertions in the three cell-lines (Figure 4B).: [evidence span withheld — anchor status: not_found]
- [methods] RepeatMasker-based classification requires ≥80% element coverage of the inserted sequence.: [evidence span withheld — anchor status: not_found]
- [intro] Full-length inserted sequences from nanomonsv enable characterization of insertion genetic properties.: "the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties"
- [discussion] Reference-based framework limitations for long repeat expansions.: "the current framework based on the reference genome may not be appropriate to capture long repeat expansion events because the reference genome is not reliable at the location susceptible to these"
