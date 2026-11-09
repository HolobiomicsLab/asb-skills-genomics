---
name: repeatmasker-satellite-annotation
description: Use when nanomonsv's Single breakend SV module has produced PASS single
  breakend calls and each breakpoint's distal side must be characterized as satellite-
  or simple-repeat-bound (e.g., centromeric rearrangements).
when_to_use_negative:
- Both breakpoints of the SV are already resolved by the Canonical SV module — RepeatMasker
  contig annotation is only needed for single breakends where one side is unresolvable
- Both breakpoints lie in poorly characterized highly repetitive reference regions
  — the Single breakend SV module itself cannot detect such events
- The question is long repeat expansion detection (VNTR/microsatellite expansions)
  — the reference-genome-based framework is not appropriate for those
edam_operation: http://edamontology.org/operation_0365
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: Produces single breakend SV calls (nanomonsv parse/get) whose PASS events
    are annotated; use without --no_single_bnd
  repo: https://github.com/friend1ws/nanomonsv
- name: RepeatMasker
  role: Annotates the realigned contig sequence after the breakpoint to classify satellite/simple
    repeat binding
- name: minimap2
  role: Aligns contigs to GRCh38.d1.vd1 reference; also aligns ONT reads with '-ax
    map-ont' upstream
  repo: https://github.com/lh3/minimap2
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform providing tumor/control reads where single breakends
    are detectable
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:b0ef9a1af72a34bf28e2a53d7cf5146a734f9c1598bef5b2277ecc1a354ad3d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - Oxford Nanopore Technologies (ONT)
  - RepeatMasker
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_0365
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# RepeatMasker satellite annotation of single breakend SVs

## Summary

Classifies nanomonsv single breakend SVs by realigning the assembled contig sequence after the breakpoint to the reference genome and annotating it with RepeatMasker to label events bound to alpha satellite, human satellite, or simple repeat sequences.

## When to use

Use when nanomonsv's Single breakend SV module has produced PASS single breakend calls and each breakpoint's distal side must be characterized as satellite- or simple-repeat-bound (e.g., centromeric rearrangements).

## When NOT to use

- Both breakpoints of the SV are already resolved by the Canonical SV module — RepeatMasker contig annotation is only needed for single breakends where one side is unresolvable
- Both breakpoints lie in poorly characterized highly repetitive reference regions — the Single breakend SV module itself cannot detect such events
- The question is long repeat expansion detection (VNTR/microsatellite expansions) — the reference-genome-based framework is not appropriate for those

## Inputs

- {tumor_prefix}.nanomonsv.sbnd.result.txt single breakend results with PASS calls from nanomonsv get
- Assembled contig sequences spanning each single breakend breakpoint
- GRCh38.d1.vd1 reference genome (FASTA) for contig realignment
- GRCh38 simple-repeat BED (used with --simple_repeat_bed in nanomonsv get)

## Outputs

- Per-event repeat classification: alpha satellite (e.g., 23 alpha satellite in the validation set), human satellite, or simple repeat bound
- Annotated single breakend table distinguishing repeat-bound events (32 of 91) from unbound events

## How to apply

For each PASS single breakend in {tumor_prefix}.nanomonsv.sbnd.result.txt, take the assembled contig sequence after the breakpoint and realign it to the GRCh38.d1.vd1 reference genome, then annotate the realigned sequence with RepeatMasker to classify the event as bound to alpha satellite, human satellite, or simple repeat sequence. Rationale: single breakend SVs have only one resolvable breakpoint because the other side typically falls in repetitive regions (centromeric, satellite, LINE1-mediated), so contig-based realignment plus RepeatMasker is needed to assign the unknown distal material. Apply this after running 'nanomonsv get' with single breakend detection enabled (no --no_single_bnd) on tumor/control ONT pairs aligned with minimap2 '-ax map-ont'. Expected counts on COLO829/H2009/HCC1954 (PRJDB10898) are 3/38/50 single breakends (91 total), of which 23 are alpha-satellite-bound, 5 human-satellite-bound, and 4 simple-repeat-bound (32 total).

## Related tools

- **nanomonsv** (Produces single breakend SV calls (nanomonsv parse/get) whose PASS events are annotated; use without --no_single_bnd) — https://github.com/friend1ws/nanomonsv
- **RepeatMasker** (Annotates the realigned contig sequence after the breakpoint to classify satellite/simple repeat binding)
- **minimap2** (Aligns contigs to GRCh38.d1.vd1 reference; also aligns ONT reads with '-ax map-ont' upstream) — https://github.com/lh3/minimap2
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform providing tumor/control reads where single breakends are detectable)

## Examples

```
nanomonsv get --prefix tumor --control_prefix control --simple_repeat_bed simpleRepeat.bed  # leaves single breakend detection enabled (no --no_single_bnd)
```

## Evaluation signals

- Per-cell-line PASS single breakend counts reproduce 3 (COLO829), 38 (H2009), 50 (HCC1954), totaling 91
- Repeat classification breakdown matches 23 alpha satellite, 5 human satellite, 4 simple repeat (32 repeat-bound of 91)
- Each classified event has a realigned contig with a defensible alignment past the breakpoint before RepeatMasker annotation
- Events in {tumor_prefix}.nanomonsv.sbnd.result.txt lacking a classifiable distal contig alignment are reported as unbound rather than dropped

## Limitations

- Cannot detect SVs where both breakpoints are in poorly characterized highly repetitive reference regions
- Reference-genome-based framework is unreliable for capturing long repeat expansion events (VNTR/microsatellite expansions)
- Interpretation of detailed structure of complex SVs is not fully automated and much work is done manually

## Evidence

- [abstract] Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences"
- [supplementary] Workflow realignment and RepeatMasker annotation: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation counts: [evidence span withheld — anchor status: not_found]
- [discussion] Module limitation: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
