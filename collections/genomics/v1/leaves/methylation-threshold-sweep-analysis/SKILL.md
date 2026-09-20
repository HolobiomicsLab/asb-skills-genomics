---
name: methylation-threshold-sweep-analysis
description: Use when comparing modification-calling tools on nanopore eventalign
  data where true-positive regions (e.g. ENCODE ChIP-seq peaks) and a matched control
  (non-specific IgG) allow efficiency-vs-noise benchmarking across score thresholds.
when_to_use_negative:
- Input lacks a matched negative-control dataset (e.g. non-specific IgG eventalign),
  so background noise rates cannot be estimated and the efficiency-to-noise metric
  is undefined.
- The goal is per-CpG methylation calling (mCG mode) rather than 6mA-containing region
  calling; use nanoHiMe's mCG module with its 1.5/-1.5 LLR thresholds instead.
- Reads are not nanopolish eventaligned or were basecalled with a different Guppy
  model version, making LLR scores incomparable with the trained nanoHiMe model.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: nanoHiMe
  role: 6mA-containing region calling on eventalign data with LLR-scored 50 bp windows,
    restricted to input.peak.bed; also applied to IgG control to quantify noise
  repo: github.com/YinLabTJ/nanoHiMe
- name: Megalodon
  role: ONT baseline caller (v2.5.0, model res_dna_r941_min_modbases-all-context_v001.cfg)
    providing probability-scored 6mA calls for the same threshold sweep
- name: nanopolish
  role: v0.13.2 event alignment of electric current events to reference k-mers, producing
    the eventalign input required by nanoHiMe
- name: minimap2
  role: v2.17 alignment of basecalled reads to hg19 with default settings prior to
    event alignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:bc6d393d90cb6bb11a694a2f18823251342c442bf3a538495bf33e03f1af9183
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Megalodon
  - minimap2
  - nanoHiMe
  - nanopolish
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# methylation-threshold-sweep-analysis

## Summary

Benchmark nanopore 6mA/modification callers (nanoHiMe vs Megalodon) by sweeping their score thresholds (LLR for nanoHiMe, probability for Megalodon) and comparing discovery efficiency against non-specific IgG background noise on nanoHiMe-seq HepG2 H3K27me3 data.

## When to use

Use when comparing modification-calling tools on nanopore eventalign data where true-positive regions (e.g. ENCODE ChIP-seq peaks) and a matched control (non-specific IgG) allow efficiency-vs-noise benchmarking across score thresholds.

## When NOT to use

- Input lacks a matched negative-control dataset (e.g. non-specific IgG eventalign), so background noise rates cannot be estimated and the efficiency-to-noise metric is undefined.
- The goal is per-CpG methylation calling (mCG mode) rather than 6mA-containing region calling; use nanoHiMe's mCG module with its 1.5/-1.5 LLR thresholds instead.
- Reads are not nanopolish eventaligned or were basecalled with a different Guppy model version, making LLR scores incomparable with the trained nanoHiMe model.

## Inputs

- nanopolish eventalign text (optionally gzip) from HepG2 H3K27me3 nanoHiMe-seq nanopore reads
- nanopolish eventalign text from matched non-specific IgG control nanopore reads
- ENCODE H3K27me3 ChIP-seq peak BED (accessions ENCFF546TJV, ENCFF950VUB) as input.peak.bed
- Upper-cased indexed reference genome FASTA (e.g. hg19.fa / ref_upper.fa)
- Raw nanopore FAST5 reads for independent Megalodon re-calling

## Outputs

- output.6mA.methylation.txt / test.win.6mA_region.txt with per-window LOG(LIKELIHOOD_RATIO)_mA per read (CHR, Chain, Start, End, Read_ID, LLR, Ref_sequence)
- Megalodon probability-scored modification calls
- discovery_efficiency_threshold_sweep.csv with discovery efficiency and efficiency-to-noise ratio per threshold for both tools

## How to apply

Run nanoHiMe in 6mA-containing region calling mode on the HepG2 H3K27me3 nanopolish eventalign data, restricted to ENCODE H3K27me3 peak regions via input.peak.bed, using 50 bp windows with a 25 bp step, and classify windows as 6mA-containing or non-6mA by sweeping the LLR threshold (the paper used 8 or 32 as fixed LLR cutoffs for 50 bp windows). Run Megalodon v2.5.0 with the res_dna_r941_min_modbases-all-context_v001.cfg model on the same nanopore reads to obtain probability-scored calls, and run nanoHiMe 6mA calling on the non-specific IgG control eventalign data over the same regions to estimate background noise. At each threshold, compute discovery efficiency as the fraction of H3K27me3 peak windows recovered and the efficiency-to-noise ratio using the IgG-derived background rate, then tabulate results for both tools (discovery_efficiency_threshold_sweep.csv). Validation: at a discovery efficiency of 0.05, the efficiency-to-noise ratio should reproduce ~10.85 for nanoHiMe versus ~6.85 for Megalodon, i.e. nanoHiMe more than 1.5 times higher.

## Related tools

- **nanoHiMe** (6mA-containing region calling on eventalign data with LLR-scored 50 bp windows, restricted to input.peak.bed; also applied to IgG control to quantify noise) — github.com/YinLabTJ/nanoHiMe
- **Megalodon** (ONT baseline caller (v2.5.0, model res_dna_r941_min_modbases-all-context_v001.cfg) providing probability-scored 6mA calls for the same threshold sweep)
- **nanopolish** (v0.13.2 event alignment of electric current events to reference k-mers, producing the eventalign input required by nanoHiMe)
- **minimap2** (v2.17 alignment of basecalled reads to hg19 with default settings prior to event alignment)

## Examples

```
nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt output.6mA 50 25 input.peak.bed
```

## Evaluation signals

- At discovery efficiency 0.05, efficiency-to-noise ratio ≈ 10.85 for nanoHiMe and ≈ 6.85 for Megalodon (nanoHiMe > 1.5x higher).
- Output file test.win.6mA_region.txt contains the 7 expected columns including a LOG(LIKELIHOOD_RATIO)_mA column where positive values indicate 6mA support.
- 6mA calling must be confined to regions listed in input.peak.bed (ENCODE H3K27me3 peaks ENCFF546TJV/ENCFF950VUB), not genome-wide.
- The threshold-sweep table covers both tools at every swept threshold with matching H3K27me3 peak-window sets.
- IgG-derived background rate must be recomputed at each threshold using the same 50 bp/25 bp window scheme.

## Limitations

- Megalodon cannot accurately identify methylated CpG sites and its accuracy for predicting 6mA-containing sites is significantly lower than nanoHiMe in dual-modification DNA.
- Without input.peak.bed, nanoHiMe 6mA calling runs across the whole genome and may be quite slow.
- Trained parameters for k-mers with methyladenines in different positions cannot be precisely assigned, so such k-mers are collapsed to an averaged k-mer, affecting LLR precision.

## Evidence

- [discussion] LLR thresholds and window classification: "set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [results] efficiency-to-noise benchmark result: [evidence span withheld — anchor status: not_found]
- [readme] peak-restricted calling via input.peak.bed: "If you set input.peak.bed , 6mA calling will be limited in these peak regions. Otherwise, 6mA calling will be althrough the whole genome and may be quite slow."
- [readme] positive LLR semantics: "A positive value in LOG(LIKELIHOOD_RATIO)_mA column shows the support for 6mA methylation."
