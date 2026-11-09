---
name: single-molecule-methylation-calling
description: Use when you have matched native and WGA PacBio SMRT cmp.h5 alignments
  for a bacterial genome and need per-molecule, per-motif-site methylation calls (e.g.,
  quantifying methylated vs non-methylated fractions of a motif like 5'-CTGCAG).
when_to_use_negative:
- Your SMRT library prep is long (10Kb+ subreads spanning many distinct motif sites)
  — use SMALR's SMp protocol instead, which pools all motif sites per molecule rather
  than assessing each site individually.
- You only have population-level kinetics and want a consensus methylome call with
  no native/WGA pair available — SMsn requires matched WGA alignments for the score
  difference.
- The modification type creates kinetic signatures extending into surrounding bases
  — the paper states detection of such modifications is beyond the scope of the current
  work.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3656
- http://edamontology.org/topic_3168
tools:
- name: SMALR
  role: Framework implementing the SMsn single-molecule, single-nucleotide scoring
    protocol (native minus WGA log-IPD difference) with motif-site identification
    and subread filtering
  repo: https://github.com/fanglab/smalr
- name: Python 2.7
  role: Runtime dependency for running the SMALR pipeline (installed via virtualenv/setup.py)
  repo: https://github.com/fanglab/smalr
- name: samtools
  role: Dependency (>= 0.1.18) for BAM/cmp.h5 handling in the SMALR pipeline
- name: R
  role: Dependency (>= 3.0) for downstream analysis of SMsn.out results
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:25ec46845fb703380f6a654625cdeb706efcd1cc35d3c623bb863f026def6ce8
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3656
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-molecule-methylation-calling

## Summary

SMALR's SMsn protocol calls methylation status at each motif site on each individual SMRT sequencing molecule by comparing subread-normalized ln(IPD) kinetics of native DNA against a whole-genome-amplified (WGA) control, yielding a single-molecule score rather than a population-level consensus.

## When to use

Use when you have matched native and WGA PacBio SMRT cmp.h5 alignments for a bacterial genome and need per-molecule, per-motif-site methylation calls (e.g., quantifying methylated vs non-methylated fractions of a motif like 5'-CTGCAG).

## When NOT to use

- Your SMRT library prep is long (10Kb+ subreads spanning many distinct motif sites) — use SMALR's SMp protocol instead, which pools all motif sites per molecule rather than assessing each site individually.
- You only have population-level kinetics and want a consensus methylome call with no native/WGA pair available — SMsn requires matched WGA alignments for the score difference.
- The modification type creates kinetic signatures extending into surrounding bases — the paper states detection of such modifications is beyond the scope of the current work.

## Inputs

- Native PacBio SMRT aligned reads (aligned_reads.cmph5 / BAM) from short (~250bp) insert library preps
- Matching whole-genome-amplified (WGA) PacBio SMRT aligned reads (cmp.h5 / BAM)
- Reference genome FASTA
- SMALR inputs file listing native_cmph5, wga_cmph5, and ref paths
- Motif and modified position specification (e.g. --motif=CTGCAG --mod_pos=2)

## Outputs

- SMsn.out per-contig output: contig, strand, motif position, SMsn score, molecule ID, native score, WGA score, native and WGA data-point counts, mean subread length
- Per-molecule methylation classification of motif sites, stratified by covSM stratum
- Estimated methylated and non-methylated motif-site fractions

## How to apply

Build an inputs file with native_cmph5, wga_cmph5, and ref (reference FASTA) lines, fastq=NONE, then run the SMsn protocol of SMALR under Python 2.7, e.g. with --motif=CTGCAG --mod_pos=2. Apply the documented subread QC filters: --minSubreadLen=100, --minAcc=0.8, --nativeCovThresh=10 (per molecule/strand), and --wgaCovThresh=10 (aggregate WGA coverage below which a position is skipped). For each motif site on each molecule, compute the SMSN score as the site- and strand-specific mean subread-normalized ln(IPD) from the native molecule minus the aggregate mean from WGA molecules, stratifying sites by single-molecule coverage (covSM, passes per molecule) without imposing subjective score thresholds. Reliability increases with more passes, and at covSM>=15 the approach achieved 98.5% sensitivity and 99.5% specificity for 6mA at methylated CTGCAG sites in E. coli O104:H4 C227-11. Results are written to SMsn.out with contig/strand, motif position, SMsn score, molecule ID, native and WGA scores, data-point counts, and mean subread length.

## Related tools

- **SMALR** (Framework implementing the SMsn single-molecule, single-nucleotide scoring protocol (native minus WGA log-IPD difference) with motif-site identification and subread filtering) — https://github.com/fanglab/smalr
- **Python 2.7** (Runtime dependency for running the SMALR pipeline (installed via virtualenv/setup.py)) — https://github.com/fanglab/smalr
- **samtools** (Dependency (>= 0.1.18) for BAM/cmp.h5 handling in the SMALR pipeline)
- **R** (Dependency (>= 3.0) for downstream analysis of SMsn.out results)

## Examples

```
smalr -i --SMsn --motif=CTGCAG --mod_pos=2 --procs=4 -c 15 input.txt
```

## Evaluation signals

- Running the SMsn test script (./run_test_SMsn.sh) generates the contig-specific folder (e.g. J99_SMsn) containing SMsn.out, confirming the scoring module runs end-to-end.
- On E. coli O104:H4 C227-11 5'-CTGCAG data (SRA SRP057274), classification at covSM>=15 should recover ~98.5% sensitivity and ~99.5% specificity against the expected fully methylated CTGCAG landscape.
- Every motif site reported must have aggregate WGA coverage above wgaCovThresh=10 and native molecules above nativeCovThresh=10; positions below these thresholds must be skipped.
- SMsn.out schema check: each row carries contig, strand, motif position, SMsn score, molecule ID, native score, WGA score, native/WGA data-point counts, and mean subread length.
- SMsn score equals native mean subread-normalized ln(IPD) minus WGA aggregate mean, with no subjective score thresholds used to assign methylation classes.

## Limitations

- Reliability of SMsn scores increases with single-molecule coverage (passes); for a given read length there is a tradeoff between library size and single-molecule, single-nucleotide methylation accuracy.
- Detection depends on the number of repeated observations per single molecule, so low covSM strata are unreliable (reported performance requires covSM>=15).
- Modifications whose kinetic signatures extend into surrounding bases are not detected by the current algorithms.
- The study was designed for cultures of single bacterial strains; application to mixed populations (clinical isolates, microbiome samples) remained future work.

## Evidence

- [intro] Each motif site on each sequencing molecule is assessed for methylation status. This is designed for use with short (~250bp) sequencing library preps: "Each motif site on each sequencing molecule is assessed for methylation status. This is designed for use with short (~250bp) sequencing library preps"
- [readme] SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads.: "SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads."
- [discussion] The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds, even at low levels of genomic: "The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds, even at low levels of genomic"
- [methods] Per mol/strand coverage threshold below which to ignore molecules [10]: "Per mol/strand coverage threshold below which to ignore molecules [10]"
- [readme] The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage).: "The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage)."
