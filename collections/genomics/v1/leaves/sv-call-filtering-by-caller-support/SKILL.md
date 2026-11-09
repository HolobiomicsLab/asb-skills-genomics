---
name: sv-call-filtering-by-caller-support
description: Use when you have raw SV calls from ≥2 independent callers (e.g. VCFs
  from Delly, Lumpy, Pindel) on the same aligned BAMs and need a high-specificity
  consensus call set by requiring multi-caller support.
when_to_use_negative:
- Sensitivity matters more than specificity — the study explicitly focused on high
  specificity rather than detecting all SVs, so multi-caller consensus will drop real
  events seen by only one caller
- Calls cannot be converted to VCF with consistent coordinates and SV type annotations
  — SURVIVOR's merge operates on VCF-formatted SV calls
- SVs were called by a single algorithm only and no second caller can be run on the
  data
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Delly
  role: Caller producing preliminary SV predictions (deletions, duplications, inversions,
    translocations) from paired-end and split-read signals; one of the ≥2 votes required
- name: Lumpy
  role: Independent caller on the same mapped BAMs contributing discordant read-pair
    and split-read evidence; one of the ≥2 votes required
- name: Pindel
  role: Independent caller on the same BAMs using split-read alignments; one of the
    ≥2 votes required
- name: cn.MOPs
  role: Fourth inference package applied for CNV prediction; its calls feed the initial
    prediction set that is filtered by caller support
- name: SURVIVOR
  role: Merges the caller VCFs into a consensus set, retaining SVs detected by at
    least two callers; also provides filter/stats modules for downstream VCF filtering
    and summary
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:25da88bc9bf82244a4df47681c4e0a6d213cffd9a998d1488707380cda304b12
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sv-call-filtering-by-caller-support

## Summary

Consensus structural-variant calling by merging preliminary SV predictions from multiple callers (Delly, Lumpy, Pindel; cn.MOPs for CNVs) with SURVIVOR, retaining only SVs detected by at least two callers. In the S. pombe strain-panel study this reduced the initial prediction set to 315 preliminary variant calls.

## When to use

Use when you have raw SV calls from ≥2 independent callers (e.g. VCFs from Delly, Lumpy, Pindel) on the same aligned BAMs and need a high-specificity consensus call set by requiring multi-caller support.

## When NOT to use

- Sensitivity matters more than specificity — the study explicitly focused on high specificity rather than detecting all SVs, so multi-caller consensus will drop real events seen by only one caller
- Calls cannot be converted to VCF with consistent coordinates and SV type annotations — SURVIVOR's merge operates on VCF-formatted SV calls
- SVs were called by a single algorithm only and no second caller can be run on the data

## Inputs

- Mapped short-read BAM files for each S. pombe strain
- Raw SV calls in VCF format from each caller (Delly, Lumpy, Pindel), with consistent coordinates and SV type annotations

## Outputs

- Consensus VCF of SVs detected by at least two callers (~315 preliminary variant calls in the reference application)
- Per-SV caller-support annotations used for subsequent filtering and manual inspection

## How to apply

Run each caller independently on the same mapped strain BAM files — Delly, Lumpy and Pindel in this study, using parameters optimized on simulated data — then normalize the outputs to VCF with consistent coordinates and SV-type annotations. Merge the caller VCFs with SURVIVOR's merge module and accept only SVs where the SV type matches across callers and breakpoints agree closely (within 1 kb), i.e. at least two of the three callers support the event. This consensus step prioritizes high specificity (correctness of the inferred SV) over high sensitivity; the study deliberately accepted that not all SVs would be detected. The ~315 consensus calls are preliminary and still require downstream filtering (removal of breakpoints overlapping low-complexity regions or annotated LTRs) and manual vetting of read alignments.

## Related tools

- **Delly** (Caller producing preliminary SV predictions (deletions, duplications, inversions, translocations) from paired-end and split-read signals; one of the ≥2 votes required)
- **Lumpy** (Independent caller on the same mapped BAMs contributing discordant read-pair and split-read evidence; one of the ≥2 votes required)
- **Pindel** (Independent caller on the same BAMs using split-read alignments; one of the ≥2 votes required)
- **cn.MOPs** (Fourth inference package applied for CNV prediction; its calls feed the initial prediction set that is filtered by caller support)
- **SURVIVOR** (Merges the caller VCFs into a consensus set, retaining SVs detected by at least two callers; also provides filter/stats modules for downstream VCF filtering and summary) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
./SURVIVOR merge delly.vcf,lumpy.vcf,pindel.vcf  # see README usage: ./SURVIVOR <command> [options]
```

## Evaluation signals

- Consensus VCF contains approximately 315 preliminary variant calls when filtering initial predictions to SVs detected by at least two callers
- Every retained SV is supported by at least two of the callers (Delly/Lumpy/Pindel/cn.MOPs) with matching SV type and breakpoints within 1 kb
- Merged VCF is valid VCF format with consistent coordinates and SV type annotations across callers
- Downstream curation after this step (low-complexity/LTR filtering plus manual inspection) should be feasible: final curation yielded 113 SVs (23 deletions, 64 duplications, 11 inversions, 15 translocations)
- SURVIVOR runs without error (git clone; cd SURVIVOR/Debug; make; ./SURVIVOR merge ...)

## Limitations

- The approach prioritizes high specificity over high sensitivity, so not all SVs in the panel were detected
- The ≥2-caller consensus output is preliminary: it still requires removal of SVs with breakpoints overlapping low-complexity regions or annotated LTRs, and manual vetting by visual inspection of read alignments
- cn.MOPs calls were not part of the SURVIVOR 3-caller merge in the described workflow — cn.MOPS initial population calls required separate manual curation
- Validation by PCR and BLAST against de novo assemblies only positively verified 76% of rearrangements, so some consensus calls may be incorrect

## Evidence

- [abstract] Filtering initial predictions by caller support: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [abstract] Caller set and parameter optimization: "we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs)24252627 to existing short-read data17, using parameters optimized on simulated data"
- [discussion] Consensus approach with SURVIVOR plus manual inspection: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [readme] SURVIVOR merge module purpose: [evidence span withheld — anchor status: external_doc]
- [intro] SURVIVOR tool scope: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples"
