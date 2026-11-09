---
name: sv-caller-vcf-conversion-and-normalization
description: Use when raw SV calls from independent tools (e.g. DELLY, Lumpy, Pindel)
  are in heterogeneous formats and must be unified as VCF records with consistent
  coordinates and SV type annotations before consensus merging.
when_to_use_negative:
- When a single high-sensitivity caller is sufficient and no cross-caller consensus
  vote is intended — the study explicitly prioritized specificity over sensitivity
  and therefore did not detect all SVs
- When inputs are long-read assemblies or smap/bedpe outputs needing different conversion
  — SURVIVOR offers smaptovcf/bedpetovcf/convertAssemblytics for those formats instead
  of caller-output normalization
- When the goal is copy-number genotyping (CNVs from cn.MOPs) rather than breakpoint-level
  SV consensus merging
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Delly
  role: Produces paired-end/split-read SV calls on strain BAMs that must be converted
    to VCF before merging
- name: Lumpy
  role: Produces discordant read-pair/split-read SV calls converted to VCF with consistent
    coordinates and type annotations
- name: Pindel
  role: Produces split-read SV calls converted to VCF prior to the two-caller consensus
    vote
- name: SURVIVOR
  role: Consumes the normalized VCFs via its merge module to generate the consensus
    SV VCF
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: cn.MOPS
  role: Fourth inference package applied to short-read data for the initial SV prediction
    set
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:68799ac19c24cc58a2883afbe768dcfd2ce2f9f2b2b94a773324b65dacef3d60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - Delly
  - Lumpy
  - Pindel
  - SURVIVOR
  - cn.MOPS
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sv-caller-vcf-conversion-and-normalization

## Summary

Converts raw structural variant outputs from multiple callers (Delly, Lumpy, Pindel) into valid, coordinate- and type-consistent VCF format so they can be merged into a consensus with SURVIVOR. Normalization is the prerequisite for the >=2-caller vote that produced the S. pombe SV catalogue.

## When to use

Use when raw SV calls from independent tools (e.g. DELLY, Lumpy, Pindel) are in heterogeneous formats and must be unified as VCF records with consistent coordinates and SV type annotations before consensus merging.

## When NOT to use

- When a single high-sensitivity caller is sufficient and no cross-caller consensus vote is intended — the study explicitly prioritized specificity over sensitivity and therefore did not detect all SVs
- When inputs are long-read assemblies or smap/bedpe outputs needing different conversion — SURVIVOR offers smaptovcf/bedpetovcf/convertAssemblytics for those formats instead of caller-output normalization
- When the goal is copy-number genotyping (CNVs from cn.MOPs) rather than breakpoint-level SV consensus merging

## Inputs

- Mapped short-read BAM files per S. pombe strain (161-strain panel)
- Raw SV call outputs from Delly, Lumpy and Pindel (caller-native formats)
- Simulated SV datasets (via SURVIVOR simSV) used for parameter optimization

## Outputs

- Per-caller normalized VCF files with consistent coordinates and SV type annotations (DEL, DUP, INV, BND/TRA)
- Merged consensus VCF (SURVIVOR merge) restricted to SVs detected by at least two callers, yielding ~315 variant calls (141 deletions, 112 duplications, 26 inversions, 36 translocations)

## How to apply

Run each caller (Delly, Lumpy, Pindel) independently on the same mapped strain BAM files; parameters should first be optimized on simulated data (SURVIVOR's simSV/eval modules support this). Convert each caller's raw output to a valid VCF with consistent coordinates and explicit SV type annotations (deletion, duplication, inversion, translocation), since callers use differing coordinate conventions and event labels. Only after normalization, merge the VCFs with SURVIVOR's merge module, accepting an SV as consensus when the SV type matches across callers and breakpoints lie within 1 kb and at least two callers agree. Rationale: the study prioritized high specificity over sensitivity, and mismatched coordinates or type labels would cause identical SVs to fail the two-caller vote. If caller outputs contain copy-number-style calls (e.g. cn.MOPs), keep them separate — the paper's consensus filtering applied to the SV-type callers feeding SURVIVOR.

## Related tools

- **Delly** (Produces paired-end/split-read SV calls on strain BAMs that must be converted to VCF before merging)
- **Lumpy** (Produces discordant read-pair/split-read SV calls converted to VCF with consistent coordinates and type annotations)
- **Pindel** (Produces split-read SV calls converted to VCF prior to the two-caller consensus vote)
- **SURVIVOR** (Consumes the normalized VCFs via its merge module to generate the consensus SV VCF) — https://github.com/fritzsedlazeck/SURVIVOR
- **cn.MOPS** (Fourth inference package applied to short-read data for the initial SV prediction set)

## Examples

```
./SURVIVOR merge callers.vcf delly.vcf lumpy.vcf pindel.vcf 1000 1 1 1 0 0
```

## Evaluation signals

- Normalized per-caller VCFs parse as valid VCF and carry consistent SV type labels (deletion, duplication, inversion, translocation) across all callers
- After SURVIVOR merge with the >=2-caller rule, the consensus VCF contains approximately 315 preliminary variant calls
- Downstream curation of the 315 calls yields the final catalogue of 113 SVs (23 deletions, 64 duplications, 11 inversions, 15 translocations), confirming breakpoint consistency
- Consensus records satisfy matching SV types with breakpoints within the 1 kb merge window (SURVIVOR merge default distance parameter)
- Calls retained after filtering show high validation rate (76% of rearrangements verified by PCR and BLAST against de novo assemblies)

## Limitations

- The two-caller filter prioritizes specificity over sensitivity, so not all SVs in the panel are detected
- Translocations and inversions are harder to genotype accurately; the authors did not re-examine these variants during genotyping
- Callers applied to existing short-read data limit resolution; remaining candidates required manual vetting by visual inspection of read alignments
- cn.MOPS produces copy-number calls that do not feed the breakpoint-level SURVIVOR merge; its calls required separate manual curation

## Evidence

- [abstract] Four callers applied before consensus: "To predict an initial set of SVs, we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs)24252627 to existing short-read data17, using parameters optimized on simulated data"
- [abstract] Two-caller filter yielding 315 calls: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [discussion] Consensus approach via SURVIVOR: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [readme] SURVIVOR merge module purpose: [evidence span withheld — anchor status: external_doc]
- [intro] SURVIVOR scope statement: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples"
