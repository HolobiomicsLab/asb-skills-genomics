---
name: plant-epigenome-dmr-quantification
description: Use when you have DeepSignal-plant call_freq methylation-frequency tables
  (chrom, pos, strand, coverage, modification_frequency) for two Nanopore samples
  and pre-computed repeat pairs, and need to quantify differentially methylated repeat
  pairs per species and cytosine context.
when_to_use_negative:
- You only have single-sample methylation calls and no paired repeat regions — the
  differential criterion |rmet1 − rmet2| >= 0.5 requires two samples per pair
- You need genome-wide DMR detection between arbitrary regions (genes, promoters)
  rather than repeat-pair-based DMRs
- Input is raw bisulfite sequencing data — the paper's point is that short-read bisulfite
  cannot profile 5mCs in repetitive genome regions, so it cannot support this repeat-pair
  analysis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: DeepSignal-plant
  role: Generates the per-site methylation frequencies consumed by this analysis via
    its call_mods and call_freq functions
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw Nanopore signals to genome reference bases before DeepSignal-plant
    methylation calling
  repo: https://github.com/nanoporetech/tombo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:2c813a73aefb63b11ce4fb99c76bafa6dd088f112dc533f0b834c1848dbe4e0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Tombo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# plant-epigenome-dmr-quantification

## Summary

Quantifies the fraction of repeat pairs (length >100, identity >0.99, ≥1 cytosine) that are differentially methylated between the paired regions, using per-site methylation frequencies from DeepSignal-plant Nanopore calls in A. thaliana and O. sativa.

## When to use

Use when you have DeepSignal-plant call_freq methylation-frequency tables (chrom, pos, strand, coverage, modification_frequency) for two Nanopore samples and pre-computed repeat pairs, and need to quantify differentially methylated repeat pairs per species and cytosine context.

## When NOT to use

- You only have single-sample methylation calls and no paired repeat regions — the differential criterion |rmet1 − rmet2| >= 0.5 requires two samples per pair
- You need genome-wide DMR detection between arbitrary regions (genes, promoters) rather than repeat-pair-based DMRs
- Input is raw bisulfite sequencing data — the paper's point is that short-read bisulfite cannot profile 5mCs in repetitive genome regions, so it cannot support this repeat-pair analysis

## Inputs

- DeepSignal-plant call_freq methylation-frequency tables per sample (e.g. fast5s.C.call_mods.frequency.tsv with chrom, pos, strand, coverage, modification_frequency columns)
- Pre-computed repeat pairs: region pairs with length >100, identity score >0.99, and at least one cytosine

## Outputs

- dmr_summary.tsv: fraction of differentially methylated repeat pairs per species plus per-context (CpG/CHG/CHH) breakdown

## How to apply

Load the DeepSignal-plant call_freq modification-frequency TSVs (e.g. fast5s.C.call_mods.frequency.tsv) for the two Nanopore samples of each species, reading chrom, pos, strand, coverage, and modification_frequency columns. For each pre-computed repeat pair (length >100, identity score >0.99, at least one cytosine), map each paired cytosine's methylation frequencies rmet1 and rmet2 from the two samples. Label a paired cytosine differentially methylated if |rmet1 − rmet2| >= 0.5, and label the whole repeat pair differentially methylated if at least 10% of its cytosines are differentially methylated, recording the sequence context (CpG/CHG/CHH) of the differing cytosines. Compute the fraction of differentially methylated repeat pairs per species and the per-context breakdown, writing the table to dmr_summary.tsv. Rationale: Nanopore reads cover repetitive regions that short-read bisulfite sequencing cannot profile, making repeat-pair DMR quantification feasible only from DeepSignal-plant per-site frequencies.

## Related tools

- **DeepSignal-plant** (Generates the per-site methylation frequencies consumed by this analysis via its call_mods and call_freq functions) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw Nanopore signals to genome reference bases before DeepSignal-plant methylation calling) — https://github.com/nanoporetech/tombo

## Examples

```
deepsignal_plant call_mods --input_path fast5s.C.features.tsv
```

## Evaluation signals

- Reproduced DMR-pair fractions of ~9% in A. thaliana and ~6% in O. sativa
- CG context dominant among differentially methylated repeat pairs in A. thaliana; CHG context dominant in O. sativa
- dmr_summary.tsv written with per-species fractions and per-context breakdown
- Every counted DMR pair satisfies the defining criteria: length >100, identity >0.99, ≥1 cytosine, ≥10% of cytosines with |rmet1 − rmet2| >= 0.5

## Limitations

- Repeat-pair selection relies on in-house Python scripts with no stated deposition, limiting reproducibility
- Per-site methylation frequencies are bounded by DeepSignal-plant accuracy and per-site read coverage from the Nanopore samples
- Key details (Supplementary Tables 3, 4, 10, 11, 12; Supplementary Notes 2–5) are deferred to supplementary materials and not in the main document

## Evidence

- [discussion] the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5: "the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5"
- [discussion] Repeat-pair criteria: "select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [methods] call_freq function: "A modification-frequency file can be generated by `call_freq` function with the call_mods file as input"
- [readme] DeepSignal-plant BiLSTM: "deepsignal-plant applies BiLSTM to detect methylation from Nanopore reads"
