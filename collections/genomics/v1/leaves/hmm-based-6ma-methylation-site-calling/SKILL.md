---
name: hmm-based-6ma-methylation-site-calling
description: Use when nanopore FAST5/FASTQ data must be scored for 6mA-containing
  sites (optionally alongside endogenous mCpGs), e.g. benchmarking callers on E. coli
  K12 MG1655 or detecting labeled adenines in nanoHiMe-seq reads.
when_to_use_negative:
- Data are short-read bisulfite (WGBS/Bismark) or PacBio — the HMM consumes nanopolish
  eventalign tables from ONT R9.4.1 signal, not methylation calls from other platforms
- 'You only need CpG methylation: use nanoHiMe''s ''mCG'' mode (thresholds 1.5/-1.5)
  rather than the 6mA caller'
- Target modifications beyond 6mA and mCpG (5hmC, 5fC, 4-mC, GpC) — the paper notes
  no unified calling model for these combinations exists yet
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
tools:
- name: nanoHiMe
  role: HMM-based 6mA (and mCG) methylation caller run on the eventalign table; also
    provides train-model to fit per-6-mer Gaussian emission parameters
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: v0.13.2 eventalign (-n, -t 16, --scale-events) maps nanopore current events
    to reference 6-mers as HMM input
- name: Guppy
  role: v4.4.2 'high accuracy' basecaller producing the FASTQ read set
- name: Minimap2
  role: v2.17 (-ax map-ont) alignment of basecalled reads before event alignment
- name: Megalodon
  role: v2.5.0 ONT baseline caller (res_dna_r941_min_modbases-all-context_v001.cfg)
    used for AUC comparison against nanoHiMe
- name: Samtools
  role: v1.9 sorting, indexing and MAPQ>=20 mapped-read filtering prior to event alignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:b5bb0dba7029fd58db6ceb1dcd895c0d221f09ab1b0d782707b9bb342615aa5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YinLabTJ/nanoHiMe
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - Samtools
  - nanoHiMe
  - nanopolish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hmm-based-6ma-methylation-site-calling

## Summary

Calls N6-methyladenine (6mA)-containing regions from Oxford Nanopore reads using nanoHiMe's hidden Markov model, which fits Gaussian-mixture emission distributions per 6-mer to nanopolish eventalign data and classifies windows by log likelihood ratio. This is the computational alternative to ONT's Megalodon/Guppy for 6mA detection in any sequence context.

## When to use

Use when nanopore FAST5/FASTQ data must be scored for 6mA-containing sites (optionally alongside endogenous mCpGs), e.g. benchmarking callers on E. coli K12 MG1655 or detecting labeled adenines in nanoHiMe-seq reads.

## When NOT to use

- Data are short-read bisulfite (WGBS/Bismark) or PacBio — the HMM consumes nanopolish eventalign tables from ONT R9.4.1 signal, not methylation calls from other platforms
- You only need CpG methylation: use nanoHiMe's 'mCG' mode (thresholds 1.5/-1.5) rather than the 6mA caller
- Target modifications beyond 6mA and mCpG (5hmC, 5fC, 4-mC, GpC) — the paper notes no unified calling model for these combinations exists yet

## Inputs

- Oxford Nanopore FAST5 signal files (e.g. MinION flowcell output from nanoHiMe-seq or E. coli DNA)
- Basecalled FASTQ from Guppy v4.4.2 with nanopolish index files
- BAM of reads aligned with minimap2 and filtered to MAPQ>=20
- nanopolish eventalign table (--scale-events, plain or gzip)
- Reference genome FASTA (e.g. E. coli K12 MG1655 or hg19), uppercased and samtools-faidx'd
- Optional BED of peak regions (input.peak.bed) to restrict calling

## Outputs

- output.6mA.methylation.txt with per-window Log(LIKELIHOOD_RATIO)_mA values
- test.win.6mA_region.txt listing CHR, Chain, Start, End, Read_ID, Log(LIKELIHOOD_RATIO)_mA and Ref_sequence for predicted 6mA-containing regions
- For benchmarking: 6ma_auc_vs_length.csv of ROC AUCs per site length (12–100 bp) and CpG context

## How to apply

Basecall FAST5 signal with Guppy v4.4.2 (high accuracy, dna_r9.4.1_450bps_hac.cfg), index reads with 'nanopolish index -d fast5', align with minimap2 (-ax map-ont), then retain mapped, MAPQ>=20 reads via 'samtools view -b -q 20 -F 4'. Run 'nanopolish eventalign -n -t 16 --scale-events' to align electric-current events to reference 6-mers, since the HMM's emission distributions are per-6-mer Gaussian mixtures fit by expectation-maximization with 5 Viterbi re-alignment iterations. Uppercase the reference with upper.pl, index with samtools faidx, and invoke 'nanoHiMe 6mA' with a 50 bp window and 25 bp step (optionally restricted to input.peak.bed); positive Log(LIKELIHOOD_RATIO)_mA values support 6mA methylation. Classify a 50 bp segment as 6mA-containing or non-6mA using fixed LLR thresholds of 8 or 32, and for benchmarks compare AUC against Megalodon v2.5.0 (res_dna_r941_min_modbases-all-context_v001.cfg) across 12–100 bp site lengths.

## Related tools

- **nanoHiMe** (HMM-based 6mA (and mCG) methylation caller run on the eventalign table; also provides train-model to fit per-6-mer Gaussian emission parameters) — https://github.com/YinLabTJ/nanoHiMe
- **nanopolish** (v0.13.2 eventalign (-n, -t 16, --scale-events) maps nanopore current events to reference 6-mers as HMM input)
- **Guppy** (v4.4.2 'high accuracy' basecaller producing the FASTQ read set)
- **Minimap2** (v2.17 (-ax map-ont) alignment of basecalled reads before event alignment)
- **Megalodon** (v2.5.0 ONT baseline caller (res_dna_r941_min_modbases-all-context_v001.cfg) used for AUC comparison against nanoHiMe)
- **Samtools** (v1.9 sorting, indexing and MAPQ>=20 mapped-read filtering prior to event alignment)

## Examples

```
nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt.gz output.6mA 50 25
```

## Evaluation signals

- Both expected files exist: output.6mA.methylation.txt and test.win.6mA_region.txt with columns CHR, Chain, Start, End, Read_ID, Log(LIKELIHOOD_RATIO)_mA, Ref_sequence
- Positive Log(LIKELIHOOD_RATIO)_mA values co-locate with known 6mA sites; window step is 25 bp with 50 bp windows as invoked
- Benchmark 40 bp AUCs reproduce reported values: nanoHiMe 0.98 (no-mCpG) / 0.992 (with-mCpG); Megalodon 0.979 / 0.967, with nanoHiMe exceeding Megalodon at every site length (nanoHiMe 0.992–0.999 vs Megalodon 0.967–0.982 across 40–100 bp mCpG-containing sites)
- Reads entering eventalign are mapped with MAPQ>=20 (samtools -q 20 -F 4 filter applied)
- LLR classification thresholds 8 or 32 for 50 bp segments yield 6mA-containing vs non-6mA labels consistent with the sampled E. coli positive/negative site sets

## Limitations

- nanoHiMe's k-mer parameter assignment is imprecise for k-mers with methyladenine(s) in different positions, so such k-mers are collapsed to an averaged k-mer
- A unified model covering other introduced modifications (5hmC, 5fC, 4-mC, GpC methylation) does not yet exist and is deferred to future work
- Haplotype phasing of 6mA-containing regions and CpG methylation depends on heterozygous SNV/INDEL density at the locus
- Genome-wide calling without input.peak.bed 'may be quite slow' per the README

## Evidence

- [readme] positive LLR supports 6mA: "A positive value in LOG(LIKELIHOOD_RATIO)_mA column shows the support for 6mA methylation."
- [readme] 6mA invocation: "nanoHiMe/nanoHiMe 6mA ref_upper.fa test.eventalign.txt(.gz) output.6mA 50 25 [input.peak.bed]"
- [discussion] HMM joint calling objective: "We also developed a computational tool to simultaneously identify 6mAs and the endogenous mCpGs from individual nanopore reads using an HMM."
- [discussion] classification thresholds: "set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [discussion] EM/Viterbi training: "We iterated the process for 5 times and assigned the parameters from the final iteration"
- [discussion] Megalodon inferiority finding: "its accuracy for predicting 6mA-containing sites was also significantly lower than the accuracy of nanoHiMe"
