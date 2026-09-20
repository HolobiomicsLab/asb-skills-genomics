---
name: motif-site-position-scoring
description: Use when short-insert (~250bp) PacBio SMRT library preps provide multiple
  passes over each motif site per molecule, and single-molecule, single-nucleotide
  methylation calls (rather than population-level consensus) are the analysis goal.
when_to_use_negative:
- Library preps are long (10Kb+) so each subread spans many distinct motif sites —
  use the SMp motif-pooled protocol instead, which assesses molecule-wide methylation
  status
- Only population-level consensus methylation calls across the whole sample are needed
  and single-cell/molecule epigenetic heterogeneity is not of interest
- No matched WGA control reads exist for kinetics comparison
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: SMALR
  role: Implements the SMsn protocol that scores each motif site on each sequencing
    molecule by comparing native vs WGA kinetics
  repo: https://github.com/fanglab/smalr
- name: Python 2.7
  role: Runtime environment/dependency required to install and run SMALR
- name: samtools
  role: Alignment manipulation dependency (>= 0.1.18) in the SMALR pipeline
- name: R
  role: Downstream analysis/plotting dependency (>= 3.0) for SMSN score distributions
    and stratification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:4dc1b7c377cc4698e2ccc30b1cfcd95338dde479a130cf8c0d2c9d0e5ffd5275
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
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# motif-site-position-scoring

## Summary

Scores each motif site on each individual sequencing molecule for methylation status by comparing native PacBio SMRT kinetics against matched WGA controls using the SMALR SMsn protocol, enabling single-molecule resolution methylome analysis without subjective thresholds.

## When to use

Use when short-insert (~250bp) PacBio SMRT library preps provide multiple passes over each motif site per molecule, and single-molecule, single-nucleotide methylation calls (rather than population-level consensus) are the analysis goal.

## When NOT to use

- Library preps are long (10Kb+) so each subread spans many distinct motif sites — use the SMp motif-pooled protocol instead, which assesses molecule-wide methylation status
- Only population-level consensus methylation calls across the whole sample are needed and single-cell/molecule epigenetic heterogeneity is not of interest
- No matched WGA control reads exist for kinetics comparison

## Inputs

- Native PacBio SMRT aligned_reads.cmph5 (short-insert ~250bp library)
- Matching WGA aligned_reads.cmp.h5 control alignment
- Reference FASTA for the strain
- inputs_file listing native_cmph5, wga_cmph5, ref, fastq=NONE
- Known methylation motif sequence and modified base position (e.g. CTGCAG, mod_pos=2)

## Outputs

- Per-molecule, per-motif-site SMSN scores (native minus aggregate WGA mean of subread-normalized ln(IPD))
- Molecule IDs, data-point counts, and mean subread lengths per site
- Methylated vs non-methylated classification per motif site per molecule
- Sensitivity/specificity estimates stratified by covSM strata

## How to apply

Retrieve paired short-insert native and WGA PacBio SMRT read sets and prepare an inputs_file with native_cmph5, wga_cmph5, and ref lines (fastq=NONE), referencing an alignment (BAM or cmp.h5) and the reference FASTA. Run the SMsn protocol of SMALR under Python 2.7 on a known motif (e.g. 5'-CTGCAG with modified position 2), specifying per-molecule and aggregate coverage thresholds (--nativeCovThresh, --wgaCovThresh, default 10), subread filters (--minSubreadLen 100, --minAcc 0.8), and a single-molecule coverage cutoff (-c). For each motif site on each molecule, compute the SMSN score as the difference between the native score and the aggregate WGA score, each being the mean of subread-normalized ln(IPD) values, recording molecule ID, data-point counts, and mean subread length. Stratify motif sites by covSM (number of passes per molecule) and classify methylated vs non-methylated status from the SMSN score distributions without subjective thresholds, since reliability increases with more passes. Validate against expected methylated motif landscapes, e.g. fully methylated CTGCAG sites in E. coli O104:H4 C227-11 where covSM>=15 yields 98.5% sensitivity and 99.5% specificity.

## Related tools

- **SMALR** (Implements the SMsn protocol that scores each motif site on each sequencing molecule by comparing native vs WGA kinetics) — https://github.com/fanglab/smalr
- **Python 2.7** (Runtime environment/dependency required to install and run SMALR)
- **samtools** (Alignment manipulation dependency (>= 0.1.18) in the SMALR pipeline)
- **R** (Downstream analysis/plotting dependency (>= 3.0) for SMSN score distributions and stratification)

## Examples

```
python 2.7 smalr -i --SMsn --motif=CTGCAG --mod_pos=2 --procs=4 -c 15 input.txt
```

## Evaluation signals

- At covSM>=15 on C227 5'-CTGCAG sites, expect sensitivity ≈ 98.5% and specificity ≈ 99.5% against the expected fully methylated CTGCAG landscape
- SMSN score distributions should separate methylated and non-methylated sites without imposing subjective thresholds
- Subread filters are honored: minSubreadLen 100, minAcc 0.8, and wgaCovThresh 10 positions below aggregate WGA coverage are skipped
- Per-site records include molecule ID, data-point counts, and mean subread length for traceability
- Native molecules below nativeCovThresh (default 10) should be excluded from scoring

## Limitations

- Detection depends on the number of repeated observations per single molecule; for a given read length there is a tradeoff between library size and accuracy of single-molecule, single-nucleotide methylation detection
- Reliability of SMsn scores increases with more passes, so low single-molecule coverage (low covSM) degrades call accuracy
- Some DNA modifications create kinetic signatures extending into surrounding bases, which are beyond the scope of the current algorithms and would require more sophisticated methods
- Some modifications beyond 6mA-type kinetic signatures at the modified position may not be detectable

## Evidence

- [intro] SMsn assesses each motif site on each molecule: "Each motif site on each sequencing molecule is assessed for methylation status. This is designed for use with short (~250bp) sequencing library preps"
- [readme] Coverage dependency of SMsn reliability: "The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage)."
- [results] Validation performance at covSM>=15: [evidence span withheld — anchor status: not_found]
- [methods] Filter thresholds: "Per mol/strand coverage threshold below which to ignore molecules [10]"
- [discussion] No subjective thresholds: "The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds, even at low levels of genomic"
