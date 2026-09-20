---
name: methylation-calling-roc-auc-benchmarking
description: Use when comparing nanopore methylation-detection tools on 6mA-containing
  site detection, especially when performance may depend on site length and the presence
  of co-occurring methylated CpGs.
when_to_use_negative:
- When the goal is per-read CpG (5mC) methylation calling rather than 6mA-containing-site
  detection benchmarking — use nanoHiMe's mCG-calling mode instead
- When data are not nanopore R9.4.1 signal reads with a compatible reference (the
  trained models and baselines assume Guppy v4.4.2 / R9.4.1 chemistry)
- When a unified methylation-calling model covering combinations of other introduced
  modifications (5-hmC, 5-fC, 4-mC, GpC methylation) is needed — such a model does
  not yet exist per the authors
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: nanoHiMe
  role: HMM-based caller of 6mA-containing regions from eventalign tables; provides
    per-window log likelihood ratios for ROC analysis
  repo: github.com/YinLabTJ/nanoHiMe
- name: Megalodon
  role: ONT-provided baseline 6mA/mCpG caller (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg)
    benchmarked against nanoHiMe
- name: Guppy
  role: Basecaller (v4.4.2, high accuracy model) producing FASTQ from FAST5 signal
    data
- name: Minimap2
  role: Aligns basecalled reads to the reference (v2.17, -ax map-ont)
- name: nanopolish
  role: Indexes reads to signal data and aligns events to reference k-mers (v0.13.2,
    eventalign --scale-events)
- name: Samtools
  role: Sorts, indexes, and filters alignments to mapped MAPQ>=20 reads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:33081323115fa5ee42a9b3eeadef81d9e1a8ff941c66583bcc6bf318ac1e2e74
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - Samtools
  - nanoHiMe
  - nanopolish
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# methylation-calling-roc-auc-benchmarking

## Summary

Benchmarks nanopore methylation callers (nanoHiMe vs Megalodon) for identifying 6mA-containing genomic sites of varying lengths (12–100 bp) by classifying windows via log likelihood ratio and computing ROC AUC per site length and CpG-methylation context.

## When to use

Use when comparing nanopore methylation-detection tools on 6mA-containing site detection, especially when performance may depend on site length and the presence of co-occurring methylated CpGs.

## When NOT to use

- When the goal is per-read CpG (5mC) methylation calling rather than 6mA-containing-site detection benchmarking — use nanoHiMe's mCG-calling mode instead
- When data are not nanopore R9.4.1 signal reads with a compatible reference (the trained models and baselines assume Guppy v4.4.2 / R9.4.1 chemistry)
- When a unified methylation-calling model covering combinations of other introduced modifications (5-hmC, 5-fC, 4-mC, GpC methylation) is needed — such a model does not yet exist per the authors

## Inputs

- E. coli K12 MG1655 FAST5 signal data (6mA-positive and 6mA-negative datasets, with and without methylated CpG context)
- Reference genome FASTA (E. coli K12 MG1655, uppercased and samtools-faidx-indexed)
- Nanopolish eventalign table (test.eventalign.txt, optionally gzip)
- Optional peak BED file (input.peak.bed) to restrict 6mA calling

## Outputs

- output.6mA.methylation.txt and test.win.6mA_region.txt from nanoHiMe (window log likelihood ratios)
- 6ma_auc_vs_length.csv with ROC AUC per site length and condition per caller
- Plot of AUC versus site length for nanoHiMe and Megalodon

## How to apply

Basecall E. coli FAST5 signal data with Guppy v4.4.2 (high accuracy model), index with nanopolish index, align with minimap2 (-ax map-ont), and filter to mapped reads with MAPQ>=20 (samtools view -b -q 20 -F 4). Produce the eventalign table with nanopolish eventalign (-n -t 16 --scale-events), convert the reference to uppercase with upper.pl, and run nanoHiMe 6mA calling on 50 bp windows with 25 bp step; run Megalodon v2.5.0 (res_dna_r941_min_modbases-all-context_v001.cfg) on the same data as the ONT baseline. Sample genomic sites of 12–100 bp from 6mA-positive and 6mA-negative E. coli datasets, with and without methylated CpG context, and classify each window using log likelihood ratios (thresholds such as 8 or 32 for calling a 50 bp segment 6mA-containing). Compute ROC AUC per site length and condition for both callers and plot AUC versus site length.

## Related tools

- **nanoHiMe** (HMM-based caller of 6mA-containing regions from eventalign tables; provides per-window log likelihood ratios for ROC analysis) — github.com/YinLabTJ/nanoHiMe
- **Megalodon** (ONT-provided baseline 6mA/mCpG caller (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg) benchmarked against nanoHiMe)
- **Guppy** (Basecaller (v4.4.2, high accuracy model) producing FASTQ from FAST5 signal data)
- **Minimap2** (Aligns basecalled reads to the reference (v2.17, -ax map-ont))
- **nanopolish** (Indexes reads to signal data and aligns events to reference k-mers (v0.13.2, eventalign --scale-events))
- **Samtools** (Sorts, indexes, and filters alignments to mapped MAPQ>=20 reads)

## Examples

```
nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt.gz output.6mA 50 25 [input.peak.bed]
```

## Evaluation signals

- 40 bp AUCs should reproduce reported values: nanoHiMe 0.98 (no-mCpG) / 0.992 (with-mCpG); Megalodon 0.979 / 0.967
- nanoHiMe AUC should exceed Megalodon at each site length in the mCpG-containing condition (0.992–0.999 vs 0.967–0.982 across 40–100 bp)
- test.win.6mA_region.txt rows must carry CHR, Chain, Start, End, Read_ID, Log(LIKELIHOOD_RATIO)_mA and Ref_sequence columns, with positive LR supporting 6mA
- AUC values must fall in [0, 1] and the CSV must cover all sampled lengths (12–100 bp) under both CpG-context conditions

## Limitations

- nanoHiMe's advantage is demonstrated on R9.4.1 E. coli spike-in data with specific Guppy/nanopolish versions; parameter retraining is required for other chemistries or k-mer distributions
- Assignment of trained parameters to k-mers with methyladenine(s) in different positions is imprecise, so such k-mers are collapsed to an averaged k-mer
- Megalodon could not accurately identify methylated CpG sites, so comparisons in the dual-modification (6mA + mCpG) condition favor nanoHiMe by design

## Evidence

- [results] For sites also containing mCpG(s), nanoHiMe substantially outperformed Megalodon, with AUCs of 0.992–0.999 across 40–100 bp sites versus Megalodon's 0.967–0.982: [evidence span withheld — anchor status: not_found]
- [discussion] Megalodon (v2.5.0) using model res_dna_r941_min_modbases-all-context_v001.cfg: "Megalodon (v2.5.0) using model res_dna_r941_min_modbases-all-context_v001.cfg"
- [discussion] we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [readme] samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [readme] nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt(.gz) output.6mA 50 25 [input.peak.bed]: "nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt(.gz) output.6mA 50 25 [input.peak.bed]  # test.eventalign.txt is created with nanopolish , could be in gzip format"
