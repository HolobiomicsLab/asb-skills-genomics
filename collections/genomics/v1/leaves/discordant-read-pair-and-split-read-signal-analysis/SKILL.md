---
name: discordant-read-pair-and-split-read-signal-analysis
description: Use when mapped paired-end short-read BAMs must be screened for deletions,
  duplications, inversions or translocations, and a high-specificity consensus callset
  is required rather than maximal sensitivity.
when_to_use_negative:
- When the goal is high sensitivity — exhaustive detection of all SVs — since this
  workflow deliberately prioritizes high specificity and will miss variants not flagged
  by at least two callers
- When genotyping accuracy of translocations and inversions is required — the authors
  found these hard to genotype accurately and excluded them from re-examination
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Delly
  role: SV caller detecting deletions, duplications, inversions and translocations
    from paired-end and split-read signals in mapped BAMs
- name: Lumpy
  role: SV caller using discordant read-pair and split-read evidence on the same BAMs
- name: Pindel
  role: SV caller detecting SVs from split-read alignments
- name: cn.MOPs
  role: Copy-number inference package contributing to the initial SV prediction set
- name: SURVIVOR
  role: Merges caller VCFs into a consensus (merge module) and supports simulation/evaluation
    of SV calling parameters
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:2ef5da26ed5073419551c511427112050539e94f7837fc1bc8a71f888e21e9e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - Delly
  - Lumpy
  - Pindel
  - SURVIVOR
  - cn.MOPs
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Discordant read-pair and split-read signal analysis

## Summary

Detects structural variants in short-read BAM alignments by exploiting discordant read-pair and split-read evidence via multiple SV callers (Delly, Lumpy, Pindel, cn.MOPs), then building a consensus set with SURVIVOR where at least two callers agree on SV type and breakpoints within 1 kb.

## When to use

Use when mapped paired-end short-read BAMs must be screened for deletions, duplications, inversions or translocations, and a high-specificity consensus callset is required rather than maximal sensitivity.

## When NOT to use

- When the goal is high sensitivity — exhaustive detection of all SVs — since this workflow deliberately prioritizes high specificity and will miss variants not flagged by at least two callers
- When genotyping accuracy of translocations and inversions is required — the authors found these hard to genotype accurately and excluded them from re-examination

## Inputs

- Mapped paired-end short-read BAM files (e.g. 161 S. pombe strains aligned to the reference genome)
- Reference genome FASTA
- Simulation-optimized caller parameter sets (from SURVIVOR simSV/eval on simulated data)

## Outputs

- Per-caller raw SV call sets (converted to valid VCF with consistent coordinates and SV type annotations)
- Merged consensus VCF restricted to SVs supported by at least two callers (e.g. 315 calls: 141 deletions, 112 duplications, 26 inversions, 36 translocations)
- Manually vetted curated SV catalogue (e.g. 113 SVs after filtering and PCR/BLAST validation)

## How to apply

Run Delly, Lumpy and Pindel independently on the same mapped strain BAMs, using parameters optimized beforehand on simulated data (e.g. SURVIVOR simSV/eval), capturing paired-end and split-read signals. cn.MOPs is additionally applied for copy-number inference. Convert each caller's raw output to valid VCF with consistent coordinates and SV type annotations. Merge the caller VCFs with SURVIVOR's merge module, retaining only SVs detected by at least two callers with matching SV type and breakpoints within 1 kb, yielding a preliminary consensus set (315 calls in the S. pombe panel). Then filter calls whose breakpoints overlap low complexity regions or annotated LTRs, and manually vet remaining SVs by visual inspection of read alignments; the priority is high specificity over sensitivity.

## Related tools

- **Delly** (SV caller detecting deletions, duplications, inversions and translocations from paired-end and split-read signals in mapped BAMs)
- **Lumpy** (SV caller using discordant read-pair and split-read evidence on the same BAMs)
- **Pindel** (SV caller detecting SVs from split-read alignments)
- **cn.MOPs** (Copy-number inference package contributing to the initial SV prediction set)
- **SURVIVOR** (Merges caller VCFs into a consensus (merge module) and supports simulation/evaluation of SV calling parameters) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
./SURVIVOR merge delly.vcf lumpy.vcf pindel.vcf 1000 2 1 -1 -1 -1
```

## Evaluation signals

- Consensus VCF contains approximately 315 preliminary variant calls when filtered to SVs detected by at least two callers (breakpoints within 1 kb, matching SV type)
- Each retained SV is supported by at least two of the four callers in the merged SURVIVOR output
- Final curated catalogue after removing low-complexity/LTR-overlapping calls and manual vetting is 113 SVs (23 deletions, 64 duplications, 11 inversions, 15 translocations)
- PCR and BLAST validation against de novo assemblies confirms ~76% of rearrangements
- All caller outputs parse as valid VCF with consistent coordinates and SV type annotations before merging

## Limitations

- Not all SVs are detected: the study prioritized high specificity over high sensitivity
- Translocations and inversions are challenging to genotype accurately and were not re-examined
- Manual vetting by visual inspection of read alignments is required and cannot be fully automated
- Rearrangement stability was not examined, despite evidence for their instability

## Evidence

- [abstract] To predict an initial set of SVs, we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs) to existing short-read data, using parameters optimized on simulated data: "To predict an initial set of SVs, we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs)24252627 to existing short-read data17, using parameters optimized on simulated data"
- [abstract] We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [abstract] we further filtered the 315 variants by removing SV calls whose breakpoints overlapped with low complexity regions or any that corresponded to previously annotated long terminal repeats: "we further filtered the 315 variants by removing SV calls whose breakpoints overlapped with low complexity regions or any that corresponded to previously annotated long terminal repeats"
- [discussion] we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [readme] merge: Compare or merge VCF files to generate a consensus or multi sample vcf files.: [evidence span withheld — anchor status: external_doc]
