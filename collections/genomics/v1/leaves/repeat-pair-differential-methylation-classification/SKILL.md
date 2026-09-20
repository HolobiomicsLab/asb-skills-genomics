---
name: repeat-pair-differential-methylation-classification
description: Use when you have DeepSignal-plant per-site methylation frequency tables
  for two homologous/repeat regions (or two samples) and need to identify cytosine-level
  and region-level differential methylation between paired repeats in plant genomes.
when_to_use_negative:
- If methylation calls come from short-read bisulfite sequencing — the article notes
  bisulfite cannot profile 5mCs in repetitive genome regions, invalidating repeat-pair
  methylation comparisons
- If the regions lack the repeat-pair criteria (length >100, identity score >0.99,
  at least one cytosine) — thresholds lose meaning for non-homologous or cytosine-free
  regions
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DeepSignal-plant
  role: Provides per-read 5mC calls and call_freq per-site methylation frequencies
    for all three contexts (CpG/CHG/CHH) that feed the differential methylation comparison
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw Nanopore signals to genome reference bases upstream of methylation
    calling
  repo: https://github.com/nanoporetech/tombo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:2aadfcdd41d882cf37030656ab831a0723711b40087915ab7a3d9142449c39ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Tombo
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Repeat-pair differential methylation classification

## Summary

Classifies pre-computed repeat pairs (length >100 bp, identity score >0.99, at least one cytosine) as differentially methylated by comparing per-site methylation frequencies from DeepSignal-plant call_freq tables between the two regions of each pair, quantifying the fraction of differentially methylated repeat pairs per species and per sequence context.

## When to use

Use when you have DeepSignal-plant per-site methylation frequency tables for two homologous/repeat regions (or two samples) and need to identify cytosine-level and region-level differential methylation between paired repeats in plant genomes.

## When NOT to use

- If methylation calls come from short-read bisulfite sequencing — the article notes bisulfite cannot profile 5mCs in repetitive genome regions, invalidating repeat-pair methylation comparisons
- If the regions lack the repeat-pair criteria (length >100, identity score >0.99, at least one cytosine) — thresholds lose meaning for non-homologous or cytosine-free regions

## Inputs

- DeepSignal-plant call_freq methylation-frequency TSVs (e.g. fast5s.C.call_mods.frequency.tsv) for the two Nanopore samples of each species (chrom, pos, strand, coverage, modification_frequency)
- Pre-computed repeat pair table: region pairs with length >100, identity score >0.99, and at least one cytosine
- Genome reference for cytosine context assignment (CpG/CHG/CHH), e.g. GCF_000001735.4_TAIR10.1 (A. thaliana) or IRGSP-1.0/GCA_001433935.1 (O. sativa)

## Outputs

- dmr_summary.tsv: fraction of differentially methylated repeat pairs per species with per-context (CpG/CHG/CHH) breakdown
- Per-repeat-pair labels (differentially methylated vs not) with counts of differentially methylated cytosines and their contexts

## How to apply

Load the call_freq modification-frequency TSVs (e.g. fast5s.C.call_mods.frequency.tsv) for the two Nanopore samples, reading chrom, pos, strand, coverage, and modification_frequency columns. Load the pre-computed repeat pairs — regions with length >100, identity score >0.99, and at least one cytosine — and map each paired cytosine's methylation frequencies rmet1 and rmet2 from the two reads of the pair. Label a cytosine differentially methylated if |rmet1 − rmet2| >= 0.5, and label the repeat pair differentially methylated if at least 10% of its cytosines are differentially methylated; record the sequence context (CpG/CHG/CHH) of the differing cytosines. Rationale: Nanopore reads profile 5mC even in repetitive regions where short-read bisulfite fails, so repeat-pair methylation can be assessed per cytosine rather than averaged. Summarize the fraction of differentially methylated pairs per species and per context into a dmr_summary.tsv table.

## Related tools

- **DeepSignal-plant** (Provides per-read 5mC calls and call_freq per-site methylation frequencies for all three contexts (CpG/CHG/CHH) that feed the differential methylation comparison) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw Nanopore signals to genome reference bases upstream of methylation calling) — https://github.com/nanoporetech/tombo

## Examples

```
deepsignal_plant call_freq --input deeps_signal.call_mods.tsv > fast5s.C.call_mods.frequency.tsv
```

## Evaluation signals

- Reproduce the reported fractions: ~9% differentially methylated repeat pairs in A. thaliana and ~6% in O. sativa
- Context dominance check: CpG context dominant among DMR pairs in A. thaliana; CHG context dominant in O. sativa
- Schema check on dmr_summary.tsv: every repeat pair classified, with ≥10%-of-cytosines rule and |rmet1 − rmet2| >= 0.5 thresholds verifiably applied
- Coverage sanity check: cytosines in the frequency tables should carry DeepSignal-plant coverage metadata; pairs lacking mapped cytosines in either sample must be excluded from the denominator

## Limitations

- Repeat-pair selection relied on in-house Python scripts with no stated deposition — the pair set itself is a reproducibility gap
- Thresholds (identity >0.99, |rmet1 − rmet2| >= 0.5, ≥10% of cytosines) are fixed choices from the paper; results are sensitive to them
- Repeat pairs must contain at least one cytosine and sufficient DeepSignal-plant read coverage; low-coverage cytosines in repeats may make rmet values unstable

## Evidence

- [discussion] select two regions of which length >100 and identity score >0.99 as repeat pairs: "select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [discussion] differentially methylated cytosine threshold: "the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5"
- [intro] bisulfite limitation in repeats: "due to short-read sequencing, bisulfite sequencing cannot profile 5mCs in the repetitive genome regions"
- [discussion] methylation frequency calculation: "by counting the number of reads where the site is called methylated and the total number of reads mapped to the site, DeepSignal-plant calculates a methylation frequency"
- [methods] call_freq function: "A modification-frequency file can be generated by `call_freq` function with the call_mods file as input"
