---
name: coverage-stratified-sensitivity-specificity-estimation
description: Use when running SMALR's SMsn protocol on native-vs-WGA PacBio SMRT data
  and you need to determine how many passes per molecule (covSM) are required before
  single-molecule methylation calls reach acceptable sensitivity/specificity.
when_to_use_negative:
- Long (10Kb+) sequencing library preps where each subread spans many distinct motif
  sites — use the SMp protocol instead, since pooling per molecule, not per-site stratification,
  is appropriate there
- Detection of modification types whose kinetic signatures extend into surrounding
  bases — the article states these require more sophisticated algorithms beyond this
  framework
- Mixed bacterial populations (clinical isolates, microbiome samples) — the study
  validated this on cultures of single bacterial strains only
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SMALR
  role: Provides the SMsn protocol that computes per-molecule, per-motif-site methylation
    scores from native-vs-WGA kinetics, the raw material for coverage stratification
  repo: https://github.com/fanglab/smalr
- name: Python 2.7
  role: Runtime dependency for running SMALR
  repo: https://github.com/fanglab/smalr
- name: samtools
  role: Dependency (>= 0.1.18) for BAM manipulation in the alignment pipeline
- name: R
  role: Dependency (>= 3.0) for downstream score-distribution analysis and stratified
    performance estimation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:232f3abdf48d8e1e85c27c5bf5968208f42ce1a97848236c3a75c4f626e3fff5
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# coverage-stratified-sensitivity-specificity-estimation

## Summary

Stratifies single-molecule methylation calls (SMSN scores from SMALR) by per-molecule coverage (covSM) and estimates methylation-call sensitivity and specificity at each stratum against a known fully methylated motif landscape (e.g., 6mA at 5'-CTGCAG in E. coli O104:H4 C227-11). Establishes the coverage floor (covSM>=15) needed for accurate single-molecule, single-nucleotide detection.

## When to use

Use when running SMALR's SMsn protocol on native-vs-WGA PacBio SMRT data and you need to determine how many passes per molecule (covSM) are required before single-molecule methylation calls reach acceptable sensitivity/specificity.

## When NOT to use

- Long (10Kb+) sequencing library preps where each subread spans many distinct motif sites — use the SMp protocol instead, since pooling per molecule, not per-site stratification, is appropriate there
- Detection of modification types whose kinetic signatures extend into surrounding bases — the article states these require more sophisticated algorithms beyond this framework
- Mixed bacterial populations (clinical isolates, microbiome samples) — the study validated this on cultures of single bacterial strains only

## Inputs

- Short-insert native PacBio SMRT alignment (aligned_reads.cmph5 / BAM) for the methylated sample
- Matching WGA PacBio SMRT alignment (aligned_reads.cmp.h5 / BAM) as the unmethylated control
- Reference genome FASTA
- inputs_file listing native_cmph5, wga_cmph5, ref, and fastq=NONE
- Known methylated motif and modified position (e.g., 5'-CTGCAG, mod_pos=2)

## Outputs

- Per-molecule, per-motif-site table with SMSN score, covSM (number of passes), data-point counts, and mean subread length
- Coverage-stratified sensitivity and specificity estimates for methylation calling
- Coverage threshold (e.g., covSM>=15) at which target performance is achieved

## How to apply

Run the SMsn protocol of SMALR on the methylated motif (e.g., --motif=CTGCAG --mod_pos=2) comparing native and WGA cmp.h5 alignments, keeping documented subread filters (minSubreadLen 100, minAcc 0.8, wgaCovThresh 10). For each motif site on each sequencing molecule compute the SMSN score as native minus aggregate WGA score, each the mean of subread-normalized ln(IPD) values, recording molecule ID, data-point counts, and mean subread length. Stratify motif sites by covSM (number of passes per molecule) and classify sites as methylated or non-methylated from the SMSN score distributions without applying subjective score thresholds. Finally, estimate sensitivity and specificity at each covSM stratum against the expected fully methylated motif landscape; the rationale is that SMsn reliability increases with passes per molecule, so performance must be validated as a function of coverage.

## Related tools

- **SMALR** (Provides the SMsn protocol that computes per-molecule, per-motif-site methylation scores from native-vs-WGA kinetics, the raw material for coverage stratification) — https://github.com/fanglab/smalr
- **Python 2.7** (Runtime dependency for running SMALR) — https://github.com/fanglab/smalr
- **samtools** (Dependency (>= 0.1.18) for BAM manipulation in the alignment pipeline)
- **R** (Dependency (>= 3.0) for downstream score-distribution analysis and stratified performance estimation)

## Examples

```
smalr -i --SMsn --motif=CTGCAG --mod_pos=2 --procs=4 -c 15 input.txt
```

## Evaluation signals

- At covSM>=15 on C227 5'-CTGCAG sites, sensitivity should reach ~98.5% and specificity ~99.5% against the expected fully methylated landscape
- Methylation/non-methylation classes should separate from SMSN score distributions without imposing subjective score thresholds
- Performance should degrade at lower covSM strata — a monotonic improvement with increasing passes per molecule confirms correct covSM computation
- SMsn scores should increase in reliability as single-molecule coverage increases (more passes per motif site)

## Limitations

- For a given read length there is a tradeoff between library size and accuracy of single-molecule, single-nucleotide methylation detection, since detection depends on the number of repeated observations per single molecule
- Detection of modifications with kinetic signatures extending into surrounding bases (e.g., non-adenine modifications) is beyond the scope of this approach and would require more sophisticated algorithms
- Existing consensus-based methods cannot provide the resolution necessary to survey complex MTase activity; conversely, this single-molecule approach was validated only on clonal single-strain cultures

## Evidence

- [methods] Each motif site on each sequencing molecule is assessed for methylation status.: "Each motif site on each sequencing molecule is assessed for methylation status."
- [methods] Aggregate WGA coverage threshold below which to skip analysis at that position [10]: "Aggregate WGA coverage threshold below which to skip analysis at that
	                                                        position [10]"
- [readme] The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage).: "The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage)."
- [intro] This is designed for use with short (~250bp) sequencing library preps, where the long read lengths of SMRT reads enables multiple passes over each motif site.: "This is designed for use with short (~250bp) sequencing library preps, where the long read lengths of SMRT reads enables multiple passes over each motif site."
- [discussion] The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds, even at low levels of genomic: "The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds, even at low levels of genomic"
