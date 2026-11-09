---
name: nanopore-6ma-region-calling
description: Use when you have nanopolish eventalign output from nanopore reads (e.g.,
  nanoHiMe-seq HepG2 H3K27me3 data) and need to identify 6mA-containing genomic windows,
  optionally limited to ChIP-seq/CUT&Tag peak regions.
when_to_use_negative:
- You need per-CpG 5-methylcytosine calls rather than 6mA-containing regions — use
  nanoHiMe mCG-calling mode instead.
- The input eventalign data was not produced with the matching trained model pipeline
  (Guppy v4.4.2 high accuracy basecalling + nanopolish eventalign with --scale-events),
  e.g., raw signal files or non-nanopolish event alignments.
- You expect modification calls from a tool trained on other model assumptions (e.g.,
  ONT Megalodon probability scores) — nanoHiMe LLR thresholds are not interchangeable
  with Megalodon probability thresholds and must be swept separately.
edam_operation: http://edamontology.org/operation_3900
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: nanoHiMe
  role: HMM-based caller that scores 50 bp windows for 6mA content via log likelihood
    ratios, optionally restricted to peak regions
  repo: github.com/YinLabTJ/nanoHiMe
- name: nanopolish v0.13.2
  role: Aligns nanopore electric-current events to reference k-mers to produce the
    eventalign input required by nanoHiMe
- name: Guppy v4.4.2
  role: Basecalls FAST5 files with the high accuracy model before event alignment
- name: minimap2 v2.17
  role: Aligns basecalled reads to the hg19 reference genome; reads are then filtered
    to MAPQ >= 20
- name: Megalodon v2.5.0
  role: Baseline ONT caller (model res_dna_r941_min_modbases-all-context_v001.cfg)
    used for probability-threshold benchmarking of nanoHiMe 6mA calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:b89bd984ca463f42e7d00d7ff561d0b6d1a87209aef1085e8f49fe2eecf5ca1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy v4.4.2
  - Megalodon v2.5.0
  - minimap2 v2.17
  - nanoHiMe
  - nanopolish
  - nanopolish v0.13.2
  edam_operation: http://edamontology.org/operation_3900
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nanopore-6ma-region-calling

## Summary

Calls 6mA-containing regions (50 bp windows, 25 bp step) from nanopolish eventalign nanopore reads using the nanoHiMe hidden Markov model, optionally restricting calling to peak regions (e.g., ENCODE H3K27me3 peaks) and scoring each window by a log likelihood ratio for 6mA methylation.

## When to use

Use when you have nanopolish eventalign output from nanopore reads (e.g., nanoHiMe-seq HepG2 H3K27me3 data) and need to identify 6mA-containing genomic windows, optionally limited to ChIP-seq/CUT&Tag peak regions.

## When NOT to use

- You need per-CpG 5-methylcytosine calls rather than 6mA-containing regions — use nanoHiMe mCG-calling mode instead.
- The input eventalign data was not produced with the matching trained model pipeline (Guppy v4.4.2 high accuracy basecalling + nanopolish eventalign with --scale-events), e.g., raw signal files or non-nanopolish event alignments.
- You expect modification calls from a tool trained on other model assumptions (e.g., ONT Megalodon probability scores) — nanoHiMe LLR thresholds are not interchangeable with Megalodon probability thresholds and must be swept separately.

## Inputs

- nanopolish eventalign file (test.eventalign.txt, optionally gzip) from mapped, q20-filtered nanopore reads
- Uppercase, samtools-faidx-indexed reference FASTA (e.g., ref_upper.fa / hg19.fa)
- Optional input.peak.bed restricting 6mA calling to target regions (e.g., ENCODE H3K27me3 ChIP-seq peaks ENCFF546TJV, ENCFF950VUB)

## Outputs

- output.6mA.methylation.txt (nanoHiMe 6mA output)
- test.win.6mA_region.txt with per-window 50 bp calls: CHR, Chain, Start, End, Read_ID, LOG(LIKELIHOOD_RATIO)_mA, Ref_sequence
- Threshold-sweep table (discovery_efficiency_threshold_sweep.csv) when used for IgG-noise benchmarking

## How to apply

First prepare event-level data: basecall FAST5 with Guppy v4.4.2 (high accuracy model), index with nanopolish, align reads to hg19 with minimap2 v2.17, filter to mapping quality >= 20 with samtools (samtools view -b -q 20 -F 4), and run nanopolish eventalign with --scale-events. Uppercase and faidx-index the reference FASTA with the provided upper.pl script. Then run nanoHiMe in 6mA mode with 50 bp windows and 25 bp step; supply an input.peak.bed (e.g., ENCODE H3K27me3 peaks ENCFF546TJV/ENCFF950VUB) to limit calling to biologically relevant regions, since whole-genome calling is slow. Classify windows using the LOG(LIKELIHOOD_RATIO)_mA column — positive values support 6mA methylation; the paper sets thresholds of 8 or 32 to call a 50 bp segment as 6mA-containing or non-6mA, and sweeps LLR thresholds (benchmarking against Megalodon probability thresholds) to compare discovery efficiency against non-specific IgG background noise.

## Related tools

- **nanoHiMe** (HMM-based caller that scores 50 bp windows for 6mA content via log likelihood ratios, optionally restricted to peak regions) — github.com/YinLabTJ/nanoHiMe
- **nanopolish v0.13.2** (Aligns nanopore electric-current events to reference k-mers to produce the eventalign input required by nanoHiMe)
- **Guppy v4.4.2** (Basecalls FAST5 files with the high accuracy model before event alignment)
- **minimap2 v2.17** (Aligns basecalled reads to the hg19 reference genome; reads are then filtered to MAPQ >= 20)
- **Megalodon v2.5.0** (Baseline ONT caller (model res_dna_r941_min_modbases-all-context_v001.cfg) used for probability-threshold benchmarking of nanoHiMe 6mA calls)

## Examples

```
nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt.gz output.6mA 50 25 input.peak.bed
```

## Evaluation signals

- At discovery efficiency 0.05, the efficiency-to-noise ratio (H3K27me3 peaks vs. IgG background) should reach ~10.85 for nanoHiMe vs. ~6.85 for Megalodon on HepG2 H3K27me3 nanoHiMe-seq data.
- Output schema check: test.win.6mA_region.txt has columns CHR, Chain, Start, End, Read_ID, LOG(LIKELIHOOD_RATIO)_mA, Ref_sequence with 50 bp windows stepped by 25 bp.
- Positive LOG(LIKELIHOOD_RATIO)_mA indicates support for 6mA; window-level calls using the paper's LLR thresholds (8 = 6mA-containing, 32 for high-stringency) should enrich windows inside input.peak.bed relative to IgG controls.
- Reference FASTA must be fully uppercase and faidx-indexed before calling; calls should only come from regions in input.peak.bed when that option is supplied.

## Limitations

- Whole-genome 6mA calling without input.peak.bed can be quite slow; peak-restricted calling is recommended.
- Model training collapses k-mers with methyladenine(s) in different positions to an averaged k-mer because partial adenine methylation prevents precise parameter assignment.
- Background 6mA signal in IgG controls must be corrected (Rf = Rs − sf*Rb) and LLR thresholds swept, since nanoHiMe and Megalodon use non-comparable score scales (LLR vs. probability).
- Caller is tied to the trained emission models for the specific modification chemistry; a unified model covering all introduced modification combinations (5-hmC, 5-fC, 4-mC, GpC methylation) does not yet exist.

## Evidence

- [readme] nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt(.gz) output.6mA 50 25 [input.peak.bed]: "nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt(.gz) output.6mA 50 25 [input.peak.bed]"
- [readme] If you set input.peak.bed , 6mA calling will be limited in these peak regions. Otherwise, 6mA calling will be althrough the whole genome: "If you set input.peak.bed , 6mA calling will be limited in these peak regions. Otherwise, 6mA calling will be althrough the whole genome"
- [discussion] we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [methods] samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [discussion] The alignment of the electric current events to k-mers of a known substring from the reference genome was obtained using nanopolish v0.13.2.: "The alignment of the electric current events to k-mers of a known substring from the reference genome was obtained using nanopolish v0.13.2."
