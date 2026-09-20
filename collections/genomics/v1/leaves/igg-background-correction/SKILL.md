---
name: igg-background-correction
description: Use when processing nanoHiMe-seq data (e.g., HepG2 H3K27me3) where an
  antibody-targeted IP sample must be de-noised against a non-specific IgG control,
  or when benchmarking 6mA callers by discovery efficiency versus background noise.
when_to_use_negative:
- When no matched non-specific IgG control sequencing data exist for the nanoHiMe-seq
  experiment — the scaling-factor subtraction cannot be estimated without Rb
- When the target modification is endogenous mCpG rather than exogenous 6mA labeling
  — use the mCG-calling mode and CpG-specific thresholds instead
- When calling over the whole genome without peak regions is acceptable and runtime
  is not a concern — note the README warns whole-genome 6mA calling 'may be quite
  slow'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: nanoHiMe
  role: HMM-based 6mA-containing region caller run on both IP and IgG eventalign data
    (50 bp windows, 25 bp step, LLR-scored); provides the Rs and Rb terms for background
    correction
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: Produces the eventalign files (signal-to-kmer alignment) that are the direct
    input to nanoHiMe 6mA calling for IP and IgG samples
- name: Megalodon
  role: ONT baseline caller (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg)
    whose probability-scored 6mA calls are benchmarked against IgG-derived noise using
    the same efficiency-to-noise metric
- name: bedtools
  role: Dependency for restricting calling and window operations to ENCODE H3K27me3
    peak regions supplied as input.peak.bed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:0333e0e708da48cd46bed7a628014d8e7dd44792243e6519a3bfbf2959a7bbf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YinLabTJ/nanoHiMe
  tools:
  - Megalodon
  - bedtools
  - nanoHiMe
  - nanopolish
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# igg-background-correction

## Summary

Quantifies and removes non-specific IgG-derived background 6mA signal from nanoHiMe-seq data by calling 6mA on IgG control eventalign data and subtracting a scaled background rate (Rf=Rs−sf*Rb), then uses the IgG noise rate to compute efficiency-to-noise ratios for benchmarking 6mA callers.

## When to use

Use when processing nanoHiMe-seq data (e.g., HepG2 H3K27me3) where an antibody-targeted IP sample must be de-noised against a non-specific IgG control, or when benchmarking 6mA callers by discovery efficiency versus background noise.

## When NOT to use

- When no matched non-specific IgG control sequencing data exist for the nanoHiMe-seq experiment — the scaling-factor subtraction cannot be estimated without Rb
- When the target modification is endogenous mCpG rather than exogenous 6mA labeling — use the mCG-calling mode and CpG-specific thresholds instead
- When calling over the whole genome without peak regions is acceptable and runtime is not a concern — note the README warns whole-genome 6mA calling 'may be quite slow'

## Inputs

- nanopolish eventalign file (.txt or .gz) from nanoHiMe-seq IP sample (e.g., HepG2 H3K27me3 antibody)
- nanopolish eventalign file from non-specific IgG control nanopore reads
- input.peak.bed with ENCODE H3K27me3 ChIP-seq peak regions (e.g., ENCFF546TJV, ENCFF950VUB)
- Reference genome FASTA (hg19, upper-cased and samtools-indexed)

## Outputs

- output.6mA.methylation.txt / test.win.6mA_region.txt with per-window LOG(LIKELIHOOD_RATIO)_mA for IP and IgG samples
- Background-corrected 6mA signal per window (Rf=Rs−sf*Rb)
- Efficiency-to-noise ratio per LLR threshold, e.g., threshold-sweep table discovery_efficiency_threshold_sweep.csv

## How to apply

Run nanoHiMe in 6mA-containing region calling mode (50 bp windows, 25 bp step, restricted to ENCODE peak regions via input.peak.bed) on both the antibody IP eventalign data and the non-specific IgG control eventalign data. Sweep the log-likelihood-ratio threshold to classify 50 bp windows as 6mA-containing, compute discovery efficiency as the fraction of H3K27me3 peak windows recovered, and derive the background rate from the IgG calls. Apply the article's background correction Rf=Rs−sf*Rb with a scaling factor, and express caller performance as the efficiency-to-noise ratio (discovery efficiency divided by IgG-derived noise) at matched efficiencies. The article validated this at a discovery efficiency of 0.05, where nanoHiMe achieved an efficiency-to-noise ratio of 10.85 versus Megalodon's 6.85.

## Related tools

- **nanoHiMe** (HMM-based 6mA-containing region caller run on both IP and IgG eventalign data (50 bp windows, 25 bp step, LLR-scored); provides the Rs and Rb terms for background correction) — https://github.com/YinLabTJ/nanoHiMe
- **nanopolish** (Produces the eventalign files (signal-to-kmer alignment) that are the direct input to nanoHiMe 6mA calling for IP and IgG samples)
- **Megalodon** (ONT baseline caller (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg) whose probability-scored 6mA calls are benchmarked against IgG-derived noise using the same efficiency-to-noise metric)
- **bedtools** (Dependency for restricting calling and window operations to ENCODE H3K27me3 peak regions supplied as input.peak.bed)

## Examples

```
nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt.gz output.6mA 50 25 input.peak.bed
```

## Evaluation signals

- At a discovery efficiency of 0.05, the efficiency-to-noise ratio should reproduce ~10.85 for nanoHiMe and ~6.85 for Megalodon on HepG2 H3K27me3 nanoHiMe-seq data
- Corrected signal Rf = Rs − sf*Rb must decrease relative to raw Rs, and corrected 6mA enrichment should localize to ENCODE H3K27me3 peak windows
- IgG-derived noise rate should approach its floor as the LLR threshold increases; nanoHiMe 6mA windows carry positive LOG(LIKELIHOOD_RATIO)_mA values as support
- Window coordinates in output must conform to the 50 bp window / 25 bp step scheme and respect input.peak.bed boundaries

## Limitations

- Without an input.peak.bed, 6mA calling runs across the whole genome and 'may be quite slow'
- Background correction assumes a linear scaling factor sf between IgG and IP background; choosing sf and the 6mA LLR window threshold (8 or 32 in the paper) affects the efficiency-to-noise tradeoff
- Assignment of trained parameters to k-mers with methyladenines in different positions is imprecise; such k-mers are collapsed to an averaged k-mer, affecting LLR calibration

## Evidence

- [discussion] Background correction of 6mA calls using IgG control data with a scaling factor: [evidence span withheld — anchor status: not_found]
- [discussion] LLR window thresholds: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [readme] peak-restricted calling: "If you set input.peak.bed , 6mA calling will be limited in these peak regions. Otherwise, 6mA calling will be althrough the whole genome and may be quite slow."
- [full_text] nanoHiMe vs Megalodon efficiency-to-noise result: [evidence span withheld — anchor status: not_found]
