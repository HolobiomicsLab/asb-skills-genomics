---
name: structural-variant-detection-from-short-read-alignments
description: Use when you have mapped short-read BAM files for multiple strains or
  samples and need preliminary SV calls filtered to a high-specificity consensus set
  (e.g. >=2-caller support).
when_to_use_negative:
- The goal is maximum sensitivity/exhaustive SV discovery rather than a high-specificity
  consensus — the study explicitly prioritized specificity, so not all SVs are detected
- Input data is long-read sequencing rather than short-read alignments, since the
  callers and signals used here are paired-end/split-read based
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Delly
  role: Detects deletions, duplications, inversions and translocations from paired-end
    and split-read signals in mapped BAMs
- name: Lumpy
  role: Detects SVs independently from discordant read-pair and split-read evidence
    on the same BAMs
- name: Pindel
  role: Detects SVs independently from split-read alignments on the same BAMs
- name: cn.MOPs
  role: Fourth caller applied to short-read data to infer copy-number-based SVs
- name: SURVIVOR
  role: Merges caller VCFs to generate a consensus/multi-sample VCF; also simulates
    and evaluates SV calls for parameter optimization
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:f48dc55abc802bf7f7e30b24056886ac9065336243f70047155f21d5f77caa3c
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# structural-variant-detection-from-short-read-alignments

## Summary

Generates a consensus catalogue of structural variants (deletions, duplications, inversions, translocations) from mapped short-read alignments by running multiple SV callers (Delly, Lumpy, Pindel, cn.MOPs) and retaining only calls supported by at least two callers, merged with SURVIVOR.

## When to use

Use when you have mapped short-read BAM files for multiple strains or samples and need preliminary SV calls filtered to a high-specificity consensus set (e.g. >=2-caller support).

## When NOT to use

- The goal is maximum sensitivity/exhaustive SV discovery rather than a high-specificity consensus — the study explicitly prioritized specificity, so not all SVs are detected
- Input data is long-read sequencing rather than short-read alignments, since the callers and signals used here are paired-end/split-read based

## Inputs

- Mapped short-read BAM files per strain (161 S. pombe strains in the source study)
- Reference genome FASTA and annotation (e.g. LTR/repeat annotations for downstream filtering)

## Outputs

- Per-caller raw SV callsets (VCF)
- Consensus SV VCF retaining only calls detected by at least two callers (~315 variant calls: deletions, duplications, inversions, translocations)
- Manually vetted high-confidence SV catalogue (113 SVs in the source study)

## How to apply

Run Delly, Lumpy and Pindel independently on the same mapped strain BAM files, using parameters optimized on simulated data (e.g. via SURVIVOR's simSV/eval modules). Convert each caller's raw output to valid VCF with consistent coordinates and SV type annotations. Merge the VCFs with SURVIVOR's merge module, retaining only SVs detected by at least two callers with matching SV type and breakpoints within 1 kb. The priority is high specificity (correctness of each inferred SV) over high sensitivity; in the source study this two-caller filter reduced four-caller predictions to 315 calls (141 deletions, 112 duplications, 26 inversions, 36 translocations) across 161 S. pombe strains, later manually vetted to a curated set of 113 SVs.

## Related tools

- **Delly** (Detects deletions, duplications, inversions and translocations from paired-end and split-read signals in mapped BAMs)
- **Lumpy** (Detects SVs independently from discordant read-pair and split-read evidence on the same BAMs)
- **Pindel** (Detects SVs independently from split-read alignments on the same BAMs)
- **cn.MOPs** (Fourth caller applied to short-read data to infer copy-number-based SVs)
- **SURVIVOR** (Merges caller VCFs to generate a consensus/multi-sample VCF; also simulates and evaluates SV calls for parameter optimization) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
git clone https://github.com/fritzsedlazeck/SURVIVOR.git && cd SURVIVOR/Debug && make && ./SURVIVOR merge
```

## Evaluation signals

- Consensus VCF contains only SVs detected by at least two callers with matching SV type and breakpoints within 1 kb
- Callset size matches expectation for the strain panel (315 preliminary calls in the source study, later curated to 113 SVs)
- SV type composition is reported and plausible (e.g. 141 deletions, 112 duplications, 26 inversions, 36 translocations)
- Parameters were optimized on simulated data (SURVIVOR simSV/eval) before application to real reads
- All remaining candidates pass manual inspection of read alignments in multiple strains

## Limitations

- High specificity is prioritized over high sensitivity, so not all SVs present will be detected
- Translocations and inversions are more challenging to accurately genotype; they may require exclusion from downstream re-examination
- Remaining calls need further filtering (e.g. breakpoints overlapping low complexity regions or annotated LTRs) and manual visual inspection of read alignments

## Evidence

- [abstract] Initial SV prediction with four callers on short-read data: "To predict an initial set of SVs, we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs)24252627 to existing short-read data17, using parameters optimized on simulated data"
- [abstract] Filter to at least two callers yielding 315 calls: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [discussion] SURVIVOR consensus approach with filtering and manual inspection: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [readme] SURVIVOR purpose from README: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples"
- [discussion] Specificity over sensitivity: "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
