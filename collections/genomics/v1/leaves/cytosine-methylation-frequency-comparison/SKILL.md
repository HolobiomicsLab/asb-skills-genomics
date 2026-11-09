---
name: cytosine-methylation-frequency-comparison
description: Use when you have DeepSignal-plant call_freq methylation-frequency tables
  (chrom, pos, strand, coverage, modification_frequency) for two Nanopore samples
  and pre-computed repeat pairs (length ≥100, identity ≥0.99) and need to quantify
  differentially methylated repeat pairs.
when_to_use_negative:
- You need per-read methylation calls rather than a comparison of per-site methylation
  frequencies between two regions (run deepsignal_plant call_mods/call_freq first
  instead)
- The regions of interest are not paired repeats meeting the length >100 / identity
  >0.99 criteria — the ≥10%-of-cytosines pair-level statistic is undefined for unrelated
  loci
- The methylation frequencies come from bisulfite or a tool other than DeepSignal-plant-style
  call_freq tables; the rmet1/rmet2 comparison assumes comparable per-site frequency
  semantics
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: DeepSignal-plant
  role: Produces the per-site methylation frequencies (call_freq) for all three contexts
    that are compared across the two samples of each repeat pair
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw Nanopore signals to the genome reference before methylation
    calling, ensuring site positions are consistent between the two samples
  repo: https://github.com/nanoporetech/tombo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:6b513a0ae43654ae31ae94b4820a128fca79afe128730223bb4ce2a4b5548313
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Tombo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cytosine-methylation-frequency-comparison

## Summary

Identifies differentially methylated repeat pairs by comparing per-site 5mC methylation frequencies (call_freq output from DeepSignal-plant) between two repeat regions in a pair, flagging cytosines with |rmet1 − rmet2| ≥ 0.5 and pairs with ≥10% differing cytosines, broken down by CpG/CHG/CHH context.

## When to use

Use when you have DeepSignal-plant call_freq methylation-frequency tables (chrom, pos, strand, coverage, modification_frequency) for two Nanopore samples and pre-computed repeat pairs (length ≥100, identity ≥0.99) and need to quantify differentially methylated repeat pairs.

## When NOT to use

- You need per-read methylation calls rather than a comparison of per-site methylation frequencies between two regions (run deepsignal_plant call_mods/call_freq first instead)
- The regions of interest are not paired repeats meeting the length >100 / identity >0.99 criteria — the ≥10%-of-cytosines pair-level statistic is undefined for unrelated loci
- The methylation frequencies come from bisulfite or a tool other than DeepSignal-plant-style call_freq tables; the rmet1/rmet2 comparison assumes comparable per-site frequency semantics

## Inputs

- DeepSignal-plant call_freq modification-frequency tsv files (e.g. fast5s.C.call_mods.frequency.tsv) for two Nanopore samples per repeat pair, with chrom, pos, strand, coverage, modification_frequency columns
- Pre-computed repeat pair list (region pairs with length >100, identity score >0.99, at least one cytosine)
- A. thaliana (GCF_000001735.4_TAIR10.1) and/or O. sativa (IRGSP-1.0 / GCA_001433935.1) genome reference for context assignment

## Outputs

- dmr_summary.tsv with fraction of differentially methylated repeat pairs per species and per-context (CpG/CHG/CHH) breakdown
- Per-pair labels with differentially methylated cytosine counts and their sequence contexts

## How to apply

Load the call_freq modification-frequency tsv files (e.g. fast5s.C.call_mods.frequency.tsv) for the two samples of a pair, reading chrom, pos, strand, coverage, and modification_frequency columns; these per-site frequencies come from counting reads called methylated over total mapped reads per cytosine. Load the pre-computed repeat pairs (regions with length >100, identity score >0.99, at least one cytosine) and map each cytosine's rmet1 and rmet2 from the two samples. Label a cytosine differentially methylated if |rmet1 − rmet2| >= 0.5, and label the repeat pair differentially methylated if at least 10% of its cytosines are differentially methylated. Record the sequence context (CpG/CHG/CHH) of the differing cytosines, compute the DMR-pair fraction per species with a per-context breakdown, and write dmr_summary.tsv. Rationale: Nanopore/DeepSignal-plant frequency calls cover repetitive regions that bisulfite short reads cannot, making this comparison feasible genome-wide.

## Related tools

- **DeepSignal-plant** (Produces the per-site methylation frequencies (call_freq) for all three contexts that are compared across the two samples of each repeat pair) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw Nanopore signals to the genome reference before methylation calling, ensuring site positions are consistent between the two samples) — https://github.com/nanoporetech/tombo

## Examples

```
dm = abs(rmet1 - rmet2) >= 0.5; pair_is_dmr = dm.mean() >= 0.10
```

## Evaluation signals

- Reproduce the reported DMR-pair fractions: ~9% of repeat pairs in A. thaliana and ~6% in O. sativa classified as differentially methylated
- Context breakdown check: CpG context is dominant among DMR pairs in A. thaliana and CHG context is dominant in O. sativa
- Invariant: every pair labeled differentially methylated has ≥10% of its cytosines with |rmet1 − rmet2| ≥ 0.5 and at least one cytosine
- Schema check: dmr_summary.tsv contains per-species and per-context fractions consistent with the number of loaded repeat pairs

## Limitations

- Repeat-pair selection relied on in-house Python scripts with no stated deposition, so pair construction is a reproducibility gap
- Per-site methylation frequencies depend on the bisulfite-trained DeepSignal-plant model, so comparison quality is bounded by bisulfite gold-standard quality
- CHG/CHH high-confidence methylated training labels for A. thaliana used a union of replicates (lower stringency), a potential label-noise source affecting frequency accuracy
- Pairs with few cytosines are sensitive to the ≥10% threshold; coverage filtering details for the DMR analysis are deferred to supplementary materials

## Evidence

- [discussion] the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5: "the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5"
- [discussion] select two regions of which length >100 and identity score >0.99 as repeat pairs: "we used in-house Python scripts to select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [methods] A modification-frequency file can be generated by `call_freq` function with the call_mods file as input: "A modification-frequency file can be generated by `call_freq` function with the call_mods file as input"
- [discussion] by counting the number of reads where the site is called methylated and the total number of reads mapped to the site, DeepSignal-plant calculates a methylation frequency: "DeepSignal-plant calculates a methylation frequency"
