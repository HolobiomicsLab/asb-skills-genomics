---
name: long-read-transcript-isoform-quantification
description: Use when you need isoform-level expression estimates from ONT direct-RNA
  (RNA004) reads aligned to a transcriptome, or when validating a nanopore simulator
  by comparing ESPRESSO-estimated isoform abundances to ground-truth transcript abundances.
when_to_use_negative:
- Input is short-read (Illumina) RNA-seq data — ESPRESSO-based isoform quantification
  here is designed for ONT long reads.
- Input is already a gene/isoform count matrix — apply the correlation comparison
  instead of re-quantifying.
- The goal is genomic DNA variant calling or metagenomic abundance only without a
  transcriptome quantification component.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: NanoSimFormer
  role: Simulates ONT direct-RNA (RNA004) signals from the transcriptome reference
    using the ont_rna004_4khz preset, with optional --trans-profile TSV for per-transcript
    truncation
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls the simulated POD5 signals into FASTQ reads via the NanoSimFormer
    --basecall option before alignment
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled HG002-RNA004 reads to the HG002 transcriptome reference
    to produce the BAM input for isoform quantification
- name: ESPRESSO
  role: Quantifies transcript isoform abundances from the aligned RNA004 read BAM,
    producing the estimates correlated against ground truth
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:96c1ac608d52f7669b9f80d2d67fb807b1c55d1819fdd32cddc1cd0466083baf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Dorado
  - ESPRESSO
  - NanoSimFormer
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# long-read-transcript-isoform-quantification

## Summary

Quantifies transcript isoform abundances from long-read (ONT direct-RNA) sequencing data by aligning basecalled reads to a transcriptome reference with minimap2 and running ESPRESSO, then validating quantification fidelity via Pearson correlation against known abundances (target ~0.996 for NanoSimFormer-simulated HG002 RNA004 reads).

## When to use

Use when you need isoform-level expression estimates from ONT direct-RNA (RNA004) reads aligned to a transcriptome, or when validating a nanopore simulator by comparing ESPRESSO-estimated isoform abundances to ground-truth transcript abundances.

## When NOT to use

- Input is short-read (Illumina) RNA-seq data — ESPRESSO-based isoform quantification here is designed for ONT long reads.
- Input is already a gene/isoform count matrix — apply the correlation comparison instead of re-quantifying.
- The goal is genomic DNA variant calling or metagenomic abundance only without a transcriptome quantification component.

## Inputs

- ONT direct-RNA (RNA004) reads as FASTQ (basecalled from simulated POD5 signals with Dorado)
- HG002 transcriptome reference for minimap2 alignment
- Optional --trans-profile TSV specifying per-transcript trunc_start/trunc_end
- Ground-truth isoform abundance table for correlation computation

## Outputs

- minimap2 alignment BAM of RNA004 reads against the transcriptome reference
- ESPRESSO transcript isoform abundance estimates
- Pearson correlation values (metagenomic ~0.9932, transcriptomic ~0.996) written to abundance_correlations.tsv

## How to apply

Simulate direct-RNA signals with NanoSimFormer using the ont_rna004_4khz preset, optionally supplying a --trans-profile TSV with per-transcript trunc_start/trunc_end values, and basecall the simulated POD5 signals with the built-in Dorado --basecall option to obtain FASTQ reads. Align the basecalled HG002-RNA004 reads to the HG002 transcriptome reference using minimap2. Run ESPRESSO on the aligned BAM to quantify transcript isoform abundances. Compute Pearson correlations between estimated and known isoform abundances; NanoSimFormer reaches approximately 0.996 transcriptomic correlation, and results can be compared against the metagenomic strain-abundance correlations (0.9932 for NanoSimFormer vs 0.9577 seq2squiggle and 0.9378 Squigulator). Write all correlation values to abundance_correlations.tsv as validation output.

## Related tools

- **NanoSimFormer** (Simulates ONT direct-RNA (RNA004) signals from the transcriptome reference using the ont_rna004_4khz preset, with optional --trans-profile TSV for per-transcript truncation) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls the simulated POD5 signals into FASTQ reads via the NanoSimFormer --basecall option before alignment) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled HG002-RNA004 reads to the HG002 transcriptome reference to produce the BAM input for isoform quantification)
- **ESPRESSO** (Quantifies transcript isoform abundances from the aligned RNA004 read BAM, producing the estimates correlated against ground truth)

## Examples

```
nanosimformer simulate --mode Reference --preset ont_rna004_4khz --trans-profile transcripts.tsv --basecall
```

## Evaluation signals

- Pearson correlation between ESPRESSO-estimated and known isoform abundances reaches approximately 0.996 for NanoSimFormer-simulated reads
- A non-empty FASTQ file of basecalled RNA004 reads is produced by the Dorado --basecall step before alignment
- minimap2 produces a valid BAM with reads mapped to the HG002 transcriptome reference
- Metagenomic comparison baseline is exceeded: 0.9932 (NanoSimFormer) vs 0.9577 (seq2squiggle) and 0.9378 (Squigulator)
- All correlation values are written to abundance_correlations.tsv for downstream verification

## Limitations

- The transcriptomic validation relies on the RNA004 statistical model derived from HG002; fidelity may differ for other samples or chemistries
- Per-transcript truncation (trunc_start/trunc_end in the --trans-profile TSV) must be specified to model realistic transcript coverage; omitting it changes the quantification ground truth
- Changelog information for the source repository is unavailable, so tool version drift cannot be verified from the repo alone

## Evidence

- [intro] It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads.: "It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads"
- [methods] Basecall step: [evidence span withheld — anchor status: not_found]
- [methods] HG002-derived model: "statistical model derived from the HG002 R10.4.1 sample"
- [other] ESPRESSO quantification step: [evidence span withheld — anchor status: not_found]
- [other] Reported correlations: [evidence span withheld — anchor status: not_found]
