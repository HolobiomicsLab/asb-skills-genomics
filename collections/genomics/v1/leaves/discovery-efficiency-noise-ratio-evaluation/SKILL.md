---
name: discovery-efficiency-noise-ratio-evaluation
description: 'Use when comparing modification callers on nanoHiMe-seq data and you
  need a threshold-free robustness metric: sweep LLR or probability cutoffs, measure
  fraction of ChIP-seq peak windows recovered, and penalize calls seen in an IgG control.'
when_to_use_negative:
- Input is ordinary CpG methylation data without an exogenous 6mA label and without
  an IgG control — background-corrected 6mA window calling is not meaningful
- The question is per-cytosine 5mC accuracy rather than 6mA-containing-region calling
  benchmarked against ChIP peaks
- No matched non-specific IgG (background) control exists, so the noise term of the
  efficiency-to-noise ratio cannot be computed
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: nanoHiMe
  role: HMM-based caller used in 6mA-containing-region calling mode (50 bp windows,
    25 bp step, LLR-scored) on H3K27me3 and IgG eventalign data
  repo: github.com/YinLabTJ/nanoHiMe
- name: Megalodon
  role: ONT baseline caller (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg)
    providing probability-scored 6mA calls for the comparison arm
- name: nanopolish
  role: Aligns electric current events to reference k-mers to produce the eventalign
    input required by nanoHiMe
- name: Guppy
  role: Basecalling of FAST5 to FASTQ (v4.4.2, high accuracy model) upstream of alignment
    and event alignment
- name: minimap2
  role: Aligns basecalled reads to hg19 (v2.17, default settings) before nanopolish
    event alignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:3450efecf8feb1f4043ba5304f10e8ea1eb51bd14fb12429eb1de57f7f608441
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Megalodon
  - minimap2
  - nanoHiMe
  - nanopolish
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# discovery-efficiency-noise-ratio-evaluation

## Summary

Benchmark nanopore 6mA-containing-region callers (nanoHiMe vs Megalodon) by sweeping LLR/probability thresholds and computing, at each threshold, discovery efficiency over ENCODE H3K27me3 peak windows against non-specific IgG background noise, summarized as an efficiency-to-noise ratio.

## When to use

Use when comparing modification callers on nanoHiMe-seq data and you need a threshold-free robustness metric: sweep LLR or probability cutoffs, measure fraction of ChIP-seq peak windows recovered, and penalize calls seen in an IgG control.

## When NOT to use

- Input is ordinary CpG methylation data without an exogenous 6mA label and without an IgG control — background-corrected 6mA window calling is not meaningful
- The question is per-cytosine 5mC accuracy rather than 6mA-containing-region calling benchmarked against ChIP peaks
- No matched non-specific IgG (background) control exists, so the noise term of the efficiency-to-noise ratio cannot be computed

## Inputs

- nanopolish eventalign text file (.eventalign.txt, optionally gzipped) for HepG2 H3K27me3 nanoHiMe-seq reads
- nanopolish eventalign file for the non-specific IgG control sample
- Reference genome fasta (upper-cased, samtools-indexed, e.g. hg19)
- ENCODE H3K27me3 ChIP-seq peak BED (input.peak.bed; e.g. ENCFF546TJV, ENCFF950VUB) restricting calling to peaks
- Raw nanopore FAST5 reads and basecalled FASTQ (for the Megalodon comparison arm)

## Outputs

- nanoHiMe output.6mA.methylation.txt / win.6mA_region.txt tables with per-read per-window LOG(LIKELIHOOD_RATIO)_mA values
- Megalodon probability-scored modification calls
- discovery_efficiency_threshold_sweep.csv with discovery efficiency and efficiency-to-noise ratio for both tools at every LLR/probability threshold
- Validation check: efficiency-to-noise ratios at discovery efficiency 0.05 (10.85 nanoHiMe, 6.85 Megalodon)

## How to apply

Run nanoHiMe in 6mA-containing region calling mode on HepG2 H3K27me3 nanopolish eventalign data, restricting calls to ENCODE H3K27me3 peak regions via input.peak.bed with 50 bp windows and 25 bp step; independently run Megalodon v2.5.0 (model res_dna_r941_min_modbases-all-context_v001.cfg) on the same nanopore reads to get probability-scored calls. Repeat the 6mA calling on the non-specific IgG control eventalign data over the same regions to quantify background. Sweep the LLR threshold (nanoHiMe; e.g. 8 vs 32 for 6mA-containing vs non-6mA windows in the paper) and the Megalodon probability threshold, classifying each 50 bp window as 6mA-containing or non-6mA at every cutoff. At each threshold compute discovery efficiency as the fraction of H3K27me3 peak windows recovered and the efficiency-to-noise ratio using the IgG-derived background rate, then tabulate both metrics for both tools in discovery_efficiency_threshold_sweep.csv. Validation: at discovery efficiency 0.05, reproduce the reported efficiency-to-noise ratios of 10.85 for nanoHiMe and 6.85 for Megalodon, confirming nanoHiMe's greater robustness.

## Related tools

- **nanoHiMe** (HMM-based caller used in 6mA-containing-region calling mode (50 bp windows, 25 bp step, LLR-scored) on H3K27me3 and IgG eventalign data) — github.com/YinLabTJ/nanoHiMe
- **Megalodon** (ONT baseline caller (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg) providing probability-scored 6mA calls for the comparison arm)
- **nanopolish** (Aligns electric current events to reference k-mers to produce the eventalign input required by nanoHiMe)
- **Guppy** (Basecalling of FAST5 to FASTQ (v4.4.2, high accuracy model) upstream of alignment and event alignment)
- **minimap2** (Aligns basecalled reads to hg19 (v2.17, default settings) before nanopolish event alignment)

## Examples

```
samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam && nanoHiMe/nanoHiMe 6mA hg19_upper.fa test.eventalign.txt.gz output.6mA 50 25 input.peak.bed
```

## Evaluation signals

- At discovery efficiency 0.05, efficiency-to-noise ratio reproduces 10.85 for nanoHiMe and 6.85 for Megalodon (ratio >1.5x higher for nanoHiMe)
- discovery_efficiency_threshold_sweep.csv contains rows for every swept LLR/probability threshold for both tools, each with discovery efficiency and efficiency-to-noise ratio
- nanoHiMe output contains positive LOG(LIKELIHOOD_RATIO)_mA values only for windows classified as 6mA-containing at the chosen threshold (paper threshold: 8 or 32 per 50 bp window)
- All nanoHiMe windows fall within the ENCODE H3K27me3 peak BED when input.peak.bed is supplied; IgG-derived background rate is computed over the identical regions
- Upstream QC: reads filtered to mapping quality >=20 (samtools view -b -q 20 -F 4) before event alignment

## Limitations

- Megalodon's accuracy for predicting 6mA-containing sites was significantly lower than nanoHiMe's, and Megalodon could not accurately identify methylated CpG sites in dual-modification DNA
- 6mA calling over the whole genome (no input.peak.bed) may be quite slow per the README
- Trained parameters cannot be precisely assigned to k-mers with methyladenines in different positions, so such k-mers are collapsed to an averaged k-mer
- Threshold sweep results depend on the fixed 50 bp window / 25 bp step scheme and the chosen LLR cutoffs (8 for 6mA-containing, 32 for non-6mA windows in the paper)

## Evidence

- [results] At a discovery efficiency of 0.05, the efficiency-to-noise ratio of nanoHiMe was 10.85, more than 1.5 times higher than Megalodon's ratio of 6.85: [evidence span withheld — anchor status: not_found]
- [discussion] Megalodon model and version: "Megalodon (v2.5.0) using model res_dna_r941_min_modbases-all-context_v001.cfg"
- [readme] nanoHiMe 6mA invocation with peak BED and window parameters: "nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt(.gz) output.6mA 50 25 [input.peak.bed]"
- [readme] Peak-region restriction rationale: "If you set input.peak.bed , 6mA calling will be limited in these peak regions. Otherwise, 6mA calling will be althrough the whole genome and may be quite slow."
- [discussion] Fixed LLR thresholds for window classification: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
