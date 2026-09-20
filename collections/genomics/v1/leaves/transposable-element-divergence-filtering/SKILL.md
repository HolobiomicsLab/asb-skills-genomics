---
name: transposable-element-divergence-filtering
description: Use when building a benchmark truth callset of recent, intact TE insertions
  from raw caller output annotated with RepeatMasker, and you need divergence- and
  length-based filtering with per-family count validation.
when_to_use_negative:
- When the goal is to detect old or highly divergent TE insertions — the <15% divergence
  cutoff deliberately excludes them by enriching for recent intact insertions
- When the input is already a curated, filtered truth set (re-applying the filters
  would change per-family counts away from the validated sizes)
- When working from short-read data rather than de novo long-read assemblies, since
  SVIM-asm insertion extraction requires assembly-based calls
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SVIM-asm
  role: Assembly-based insertion caller used on HPRC de novo assemblies to extract
    candidate TE insertion sequences prior to annotation and filtering
- name: RepeatMasker
  role: Aligns extracted insertion sequences against TE consensus sequences and assigns
    TE family labels and divergence values used for filtering
- name: minimap2
  role: Alignment of insertion sequences in the truth-set generation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:87c9a611f036786a53af2a86d51e4105ee9c40790eb85310276a489f970ae84d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RepeatMasker
  - SVIM-asm
  - minimap2
  - python3
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# transposable-element-divergence-filtering

## Summary

A truth-set curation procedure that filters assembly-based transposable element insertion calls (SVIM-asm annotated with RepeatMasker) by sequence divergence (<15% relative to the TE consensus) and TE-family-specific minimum lengths, enriching for recent intact insertions used to benchmark long-read TE callers such as sTELLeR.

## When to use

Use when building a benchmark truth callset of recent, intact TE insertions from raw caller output annotated with RepeatMasker, and you need divergence- and length-based filtering with per-family count validation.

## When NOT to use

- When the goal is to detect old or highly divergent TE insertions — the <15% divergence cutoff deliberately excludes them by enriching for recent intact insertions
- When the input is already a curated, filtered truth set (re-applying the filters would change per-family counts away from the validated sizes)
- When working from short-read data rather than de novo long-read assemblies, since SVIM-asm insertion extraction requires assembly-based calls

## Inputs

- De novo genome assemblies (e.g., HPRC assemblies of HG002 and HG01071)
- SVIM-asm candidate insertion sequences (FASTA)
- TE consensus sequence FASTA for alignment
- RepeatMasker annotation of insertions (TE family labels and divergence values)
- Supporting tooling: minimap2, samtools, python3

## Outputs

- Per-sample truth callset tables of filtered TE insertions, split by family (Alu, L1, HERV, SVA)

## How to apply

First run assembly-based insertion calling with SVIM-asm on HPRC de novo assemblies (e.g., HG002 and HG01071) to extract candidate insertion sequences, then align these sequences against TE consensus sequences and annotate them with RepeatMasker to assign TE family labels. Filter annotated insertions to retain only those with sequence divergence below 15% relative to the TE consensus, which enriches for recent intact insertions. Apply TE-family-specific minimum consecutive length filters: 300 bp for SVA, 1500 bp for HERV, 200 bp for L1, and 100 bp for Alu. Compile the filtered calls into per-sample truth callset tables and validate by confirming the per-family counts match the expected truth set sizes.

## Related tools

- **SVIM-asm** (Assembly-based insertion caller used on HPRC de novo assemblies to extract candidate TE insertion sequences prior to annotation and filtering)
- **RepeatMasker** (Aligns extracted insertion sequences against TE consensus sequences and assigns TE family labels and divergence values used for filtering)
- **minimap2** (Alignment of insertion sequences in the truth-set generation workflow)

## Evaluation signals

- Per-family counts match the reported truth set sizes: HG002 = 2193 Alu, 492 L1, 14 HERV, 205 SVA; HG01071 = 2262 Alu, 439 L1, 11 HERV, 202 SVA
- No retained insertion exceeds 15% sequence divergence relative to its TE consensus
- Every retained insertion meets its family-specific minimum length: SVA ≥ 300 bp, HERV ≥ 1500 bp, L1 ≥ 200 bp, Alu ≥ 100 bp
- Filtered calls compile into complete per-sample truth callset tables for both HG002 and HG01071

## Limitations

- The <15% divergence cutoff enriches for recent intact insertions, so old, fragmented, or highly divergent TE insertions are excluded from the truth set
- Truth set composition depends on the upstream caller (SVIM-asm on HPRC assemblies) and on RepeatMasker family annotation quality
- Family-specific length thresholds differ by an order of magnitude (Alu 100 bp vs HERV 1500 bp), so a single uniform length filter cannot substitute for the per-family cutoffs

## Evidence

- [other] Divergence and length filters define the truth set: [evidence span withheld — anchor status: not_found]
- [other] Divergence rationale: [evidence span withheld — anchor status: not_found]
- [other] Upstream calling and annotation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Downstream benchmarking context: "Particularly, sTELLeR exhibit higher precision and sensitivity for calling of Alu elements than similar tools."
- [readme] Long-read TE caller interface used against such truth sets: "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)"
