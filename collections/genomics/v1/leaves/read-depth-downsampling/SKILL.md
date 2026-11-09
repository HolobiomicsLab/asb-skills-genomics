---
name: read-depth-downsampling
description: Use when testing how far SMRT read depth can be reduced before single-molecule
  SMSN methylation detection (sensitivity/specificity) degrades, e.g. to probe reference-free
  de novo motif discovery at low coverage.
when_to_use_negative:
- Library prep is long (10Kb+) with single long subreads spanning many distinct motif
  sites — use the SMp motif-pooled protocol instead, whose reliability scales with
  number of distinct motif sites per subread.
- Goal is population-level consensus methylation calling rather than single-molecule,
  single-nucleotide methylation status.
- Modification has kinetic signatures extending into surrounding bases — beyond the
  scope of SMALR's current per-position approach.
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: SMALR
  role: Run in SMsn mode on downsampled native/WGA alignments to compute single-molecule,
    single-nucleotide SMSN methylation scores
  repo: https://github.com/fanglab/smalr
- name: samtools
  role: Downsample native and WGA aligned BAM/cmp.h5 alignments to reduced genomic
    coverage
- name: Python 2.7
  role: SMALR runtime dependency and parsing of per-contig SMsn.out score files
- name: R
  role: Downstream statistics on SMSN score separation (sensitivity/specificity) across
    coverage levels
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:dd8fe108bb3b4c47d4e4c31f5ea51437046fbd2e4aecc4bc9734d81bd655c229
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - Python
  - Python 2.7
  - R
  - SMALR
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# read-depth-downsampling

## Summary

Downsample native and WGA SMRT alignments (cmp.h5/BAM) to reduced genomic coverage with samtools, then run SMALR SMsn to determine the single-molecule coverage floor at which SMSN-based 6mA detection retains accuracy.

## When to use

Use when testing how far SMRT read depth can be reduced before single-molecule SMSN methylation detection (sensitivity/specificity) degrades, e.g. to probe reference-free de novo motif discovery at low coverage.

## When NOT to use

- Library prep is long (10Kb+) with single long subreads spanning many distinct motif sites — use the SMp motif-pooled protocol instead, whose reliability scales with number of distinct motif sites per subread.
- Goal is population-level consensus methylation calling rather than single-molecule, single-nucleotide methylation status.
- Modification has kinetic signatures extending into surrounding bases — beyond the scope of SMALR's current per-position approach.

## Inputs

- native aligned_reads.cmp.h5 (or BAM) SMRT alignment
- matched WGA aligned_reads.cmp.h5 (or BAM) SMRT alignment
- known motif site positions (e.g. 5'-CTGCAG MTase target) on both strands

## Outputs

- downsampled native and WGA alignment files at reduced genomic coverage
- per-contig SMsn.out files with per-molecule SMSN scores
- sensitivity/specificity estimates per coverage threshold (e.g. 98.5%/99.5% at covSM>=15)
- recovered_motif_CTGCAG.tsv concordance assessment for de novo recovered motif

## How to apply

Downsample the C227 native and WGA aligned cmp.h5/BAM alignments to progressively reduced genomic coverage using samtools. Run SMALR in SMsn mode on the downsampled inputs, filtering subreads shorter than 100 bp or below 0.8 accuracy (--minSubreadLen 100, --minAcc 0.8) and applying per-molecule native (--nativeCovThresh) and aggregate WGA (--wgaCovThresh) coverage thresholds. Parse per-contig SMsn.out files to extract SMSN scores (native minus WGA mean subread-normalized ln(IPD)) at candidate motif positions on both strands, then assess sensitivity and specificity versus full coverage and locate the coverage threshold where performance holds. Rationale: SMsn reliability increases with more passes per molecule, so this skill empirically finds the reduced-coverage floor (covSM≥15 gave 98.5% sensitivity, 99.5% specificity) that still supports robust methylated/non-methylated separation and de novo motif discovery.

## Related tools

- **SMALR** (Run in SMsn mode on downsampled native/WGA alignments to compute single-molecule, single-nucleotide SMSN methylation scores) — https://github.com/fanglab/smalr
- **samtools** (Downsample native and WGA aligned BAM/cmp.h5 alignments to reduced genomic coverage)
- **Python 2.7** (SMALR runtime dependency and parsing of per-contig SMsn.out score files)
- **R** (Downstream statistics on SMSN score separation (sensitivity/specificity) across coverage levels)

## Examples

```
samtools view -s 0.25 native_aligned.bam > native_ds.bam && SMALR --mode SMsn --native native_ds.bam --wga wga_ds.bam --minSubreadLen 100 --minAcc 0.8
```

## Evaluation signals

- At the retained coverage threshold, sensitivity and specificity reach ~98.5% and ~99.5% for 6mA detection (covSM>=15 in the C227 E. coli dataset).
- Per-contig SMsn.out outputs contain SMSN scores computed as native minus WGA mean subread-normalized ln(IPD) at candidate motif positions on both strands.
- Subread filter parameters (minSubreadLen=100, minAcc=0.8) and per-molecule native / aggregate WGA coverage thresholds are applied to the downsampled data, not the full-coverage data.
- De novo clustering of high-SMSN-score methylated sites by flanking sequence context recovers the known C227 5'-CTGCAG MTase target with no prior motif knowledge.
- Downsampling is verified by recomputing genomic/single-molecule coverage from the downsampled alignments before running SMALR.

## Limitations

- Detection depends on the number of repeated observations per single molecule; for a given read length there is a tradeoff between library size and accuracy of single-molecule, single-nucleotide methylation detection.
- Modifications whose kinetic signatures extend into surrounding bases require more sophisticated algorithms and are beyond the current scope.
- SMsn reliability increases with more passes (higher single-molecule coverage), so performance degrades below the empirical coverage floor.

## Evidence

- [discussion] the robust separation between methylated and non-methylated sites based on SMSN scores at low levels of genomic sequence coverage suggests the possibility of reference-free, de novo discovery of: "the robust separation between methylated and non-methylated sites based on SMSN scores at low levels of genomic sequence coverage suggests the possibility of reference-free, de novo discovery of"
- [readme] The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage).: "The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage)."
- [methods] Per mol/strand coverage threshold below which to ignore molecules [10]: "Per mol/strand coverage threshold below which to ignore molecules [10]"
- [methods] Each motif site on each sequencing molecule is assessed for methylation status.: "Each motif site on each sequencing molecule is assessed for methylation status."
