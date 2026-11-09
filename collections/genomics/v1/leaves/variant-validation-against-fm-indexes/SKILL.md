---
name: variant-validation-against-fm-indexes
description: Use when a set of long-read-derived TR/variant calls (e.g., TRiCoLOR
  BCF output from ONT or PB haplotype-resolved BAMs) needs an independent, short-read-based
  validation ratio computed from same-sample Illumina data.
when_to_use_negative:
- No Illumina short-read data or pre-built FM indexes are available for the same samples,
  so FM-index lookup cannot be performed
- Validating variant types other than the TRs TRiCoLOR calls (e.g., SNVs or structural
  variants outside ≥50 bp tandem repeats), which the FM-index TR validation approach
  was not designed to score
- Repeats located in unassembled genome regions (centromeres, telomeres), which TRiCoLOR
  cannot identify in the first place
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: TRiCoLOR
  role: Produces the de novo TR variant calls (BCF) that are validated against the
    Illumina FM indexes
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: Upstream module identifying candidate low-entropy repetitive regions that
    are filtered and validated downstream
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR REFER
  role: Upstream module genotyping TRs via SPOA consensus and RegEx motif matching,
    emitting the BCF calls to be validated
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns ONT reads to GRCh38 to generate the haplotype-resolved BAMs from which
    validated calls derive
- name: SPOA
  role: Builds haplotype-specific consensus sequences (~77% ONT / ~88% PB error reduction)
    used for the calls being validated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:763176f29b2f2c4cbf4a4a9c298964237ecbcb31d90194c745001a193f1920a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - Docker
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# variant-validation-against-fm-indexes

## Summary

Validate de novo tandem repeat calls from long-read data by checking whether each variant is present in alignment-free Illumina FM indexes of the same sample, counting calls as valid when found with ≤2 bp discrepancies, and optionally confirming non-reference alleles against an assembly-based truth set.

## When to use

Use when a set of long-read-derived TR/variant calls (e.g., TRiCoLOR BCF output from ONT or PB haplotype-resolved BAMs) needs an independent, short-read-based validation ratio computed from same-sample Illumina data.

## When NOT to use

- No Illumina short-read data or pre-built FM indexes are available for the same samples, so FM-index lookup cannot be performed
- Validating variant types other than the TRs TRiCoLOR calls (e.g., SNVs or structural variants outside ≥50 bp tandem repeats), which the FM-index TR validation approach was not designed to score
- Repeats located in unassembled genome regions (centromeres, telomeres), which TRiCoLOR cannot identify in the first place

## Inputs

- TRiCoLOR REFER output: called TR variants in BCF-compliant format (TRs ≥50 bp)
- Illumina FM indexes built from the same samples' short-read sequencing data
- Assembly-based truth set of non-reference TR alleles (for HG00733 confirmation)

## Outputs

- Per-sample validation ratios (~82% HG00733, ~85% HG00514, ~86% NA19240)
- Assembly-based confirmation ratio for HG00733 non-reference TR alleles (42/58, ~75%)
- Per-call valid/invalid classification (≤2 bp discrepancy in FM index)

## How to apply

After running TRiCoLOR SENSoR and REFER on haplotype-resolved BAMs (e.g., HGSVC ONT/PB samples HG00514, HG00733, NA19240 aligned to GRCh38 with minimap2), build FM indexes from the same samples' Illumina sequencing data. For each called TR variant, search the FM indexes and count the call as valid if the variant is found with ≤2 bp discrepancies. Compute per-sample validation ratios; on the HGSVC real data this yields ~82% for HG00733 (ONT and PB), ~85% for HG00514, and ~86% for NA19240. For HG00733, additionally confirm non-reference TR alleles against an assembly-based truth set, requiring the same variant type and a predicted-variant size difference not exceeding 50 bp, giving a confirmation rate of 42/58 (~75%).

## Related tools

- **TRiCoLOR** (Produces the de novo TR variant calls (BCF) that are validated against the Illumina FM indexes) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SENSoR** (Upstream module identifying candidate low-entropy repetitive regions that are filtered and validated downstream) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR REFER** (Upstream module genotyping TRs via SPOA consensus and RegEx motif matching, emitting the BCF calls to be validated) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns ONT reads to GRCh38 to generate the haplotype-resolved BAMs from which validated calls derive)
- **SPOA** (Builds haplotype-specific consensus sequences (~77% ONT / ~88% PB error reduction) used for the calls being validated)

## Evaluation signals

- Per-sample validation ratios reproduce the paper's reported values: ~82% HG00733 (ONT and PB), ~85% HG00514, ~86% NA19240
- HG00733 assembly-based confirmation of non-reference TR alleles equals 42/58 (~75%)
- A call is counted valid only when found in the Illumina FM indexes with ≤2 bp discrepancies
- Assembly confirmation requires same variant type and predicted-vs-truth size difference ≤50 bp
- All calls subjected to validation are TRs ≥50 bp stored in BCF-compliant format

## Limitations

- FM-index validation is alignment-free and depends on Illumina coverage of the repeat locus; short reads may not span long expanded repeats
- Calls with up to 2 bp discrepancy are accepted, so near-miss matches are treated as valid rather than exact matches
- TRiCoLOR cannot identify repeats in unassembled regions (centromeres, telomeres), so those cannot be validated; no support for polyploid species or haploid chromosomes such as the human Y chromosome

## Evidence

- [results] ≤2 bp discrepancy rule: "we counted as valid calls also variants that are found in the Illumina FM indexes with ≤2 bp discrepancies"
- [results] High validation rates on real data: [evidence span withheld — anchor status: not_found]
- [results] FM-index validation approach: [evidence span withheld — anchor status: not_found]
- [results] Assembly confirmation criteria: "the predicted variant size is roughly similar (i.e., the difference does not exceed 50 bp)"
- [methods] ≥50 bp TR storage: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [discussion] Scope limitation: "TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)"
