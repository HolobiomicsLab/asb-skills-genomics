---
name: corrected-read-reference-alignment-evaluation
description: Use when corrected long-read FASTA output (e.g., HECIL's LongRead_Corrected.fasta
  from one or more iterations) must be quantitatively validated against a reference
  genome using BLASR-derived PI and % matched bases.
when_to_use_negative:
- No reference genome is available and validation must be reference-free (e.g., k-mer
  counting with Jellyfish or de novo assembly with Canu followed by QUAST and MUMmer
  dnadiff)
- Evaluating assembly-level accuracy of corrected reads rather than read-level alignment
  metrics
- Input reads are uncorrected and no hybrid short-read data exists to run HECIL first
  — there would be no corrected FASTA to align
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: BLASR
  role: Aligns uncorrected and per-iteration corrected long reads to the reference
    genome so PI and % matched bases can be computed for validation
- name: HECIL
  role: Produces the per-iteration corrected long-read FASTA files (LongRead_Corrected.fasta)
    that are evaluated by reference alignment
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:9ce1fdc83e5e2465cc237deee47015229e13a8edc4f364e4e7b6695be23b999c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BLASR
  - BWA-MEM
  - HECIL
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# corrected-read-reference-alignment-evaluation

## Summary

Scores hybrid-corrected long reads (e.g., HECIL output per iteration) by aligning both uncorrected and corrected FASTA to a reference genome with BLASR and computing percent identity (PI) and % matched bases, as done for E. coli K-12 MG1655 PacBio/Sequel reads. This provides the alignment-based validation of whether iterative correction actually improved read accuracy.

## When to use

Use when corrected long-read FASTA output (e.g., HECIL's LongRead_Corrected.fasta from one or more iterations) must be quantitatively validated against a reference genome using BLASR-derived PI and % matched bases.

## When NOT to use

- No reference genome is available and validation must be reference-free (e.g., k-mer counting with Jellyfish or de novo assembly with Canu followed by QUAST and MUMmer dnadiff)
- Evaluating assembly-level accuracy of corrected reads rather than read-level alignment metrics
- Input reads are uncorrected and no hybrid short-read data exists to run HECIL first — there would be no corrected FASTA to align

## Inputs

- Uncorrected long-read FASTA (e.g., 33,360 filtered E. coli PacBio/Sequel reads, reads < 100 bp excluded)
- Corrected long-read FASTA per iteration (LongRead_Corrected.fasta from HECIL iterations 1–5)
- Reference genome FASTA (e.g., E. coli K-12 substr. MG1655)

## Outputs

- BLASR alignments of each corrected FASTA to the reference genome
- Per-iteration percent identity (PI) and % matched bases values
- Progression table of six rows (uncorrected through iteration 5), e.g., pi_matched_bases_progression.csv

## How to apply

Run HECIL first (BWA-MEM aligns short reads to long reads; core algorithm uses quick correction at consensus threshold η = 0.9, then optimization-based correction over cost J_k = ½(ω̂_q^k + ω̂_s^k), with confidence assignment at α > 95 percentile) for iterations 1 through 5, saving each iteration's corrected FASTA. Align the uncorrected long reads and each iteration's corrected reads to the reference genome (e.g., E. coli K-12 substr. MG1655) using BLASR. From the alignments compute percent identity and % matched bases per iteration. Write the six-row progression table (uncorrected baseline through iteration 5) and check that both metrics increase monotonically relative to the uncorrected baseline; for Sequel E. coli the expected trajectory is PI 85.0 → 95.6 (iteration 1) → 97.2 (iteration 5) and % matched bases 85.1 → 95.6 → 97.1.

## Related tools

- **BLASR** (Aligns uncorrected and per-iteration corrected long reads to the reference genome so PI and % matched bases can be computed for validation)
- **HECIL** (Produces the per-iteration corrected long-read FASTA files (LongRead_Corrected.fasta) that are evaluated by reference alignment) — https://github.com/NDBL/HECIL

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out  # then BLASR-align LongRead_Corrected.fasta per iteration to the reference
```

## Evaluation signals

- PI and % matched bases increase monotonically from the uncorrected baseline across iterations 1–5
- For Sequel E. coli, PI reaches ~95.6 at iteration 1 (≈12.4% improvement) and ~97.2 at iteration 5 (≈14.3% improvement)
- For Sequel E. coli, % matched bases rises from 85.1 (original reads) to ~95.6 at iteration 1 and ~97.1 at iteration 5 (≈14.1% improvement)
- The progression table contains exactly six rows: uncorrected plus iterations 1 through 5 (the maximal number of iterations serving as the secondary termination criterion)
- Every corrected-iteration alignment yields strictly higher PI and % matched bases than the uncorrected original reads

## Limitations

- Iterative improvement is only a conjecture with no mathematical proof; the paper notes proving iterative improvement likely requires unrealistic stochastic assumptions
- Saturation behavior of the iterative procedure is unvalidated speculation — metrics are expected to improve 'until saturation' but this is not demonstrated
- Six iterations are the evaluated maximum; beyond iteration 5 no validated expectation exists

## Evidence

- [results] we align corrected long reads to its reference genome using BLASR: "we align corrected long reads to its reference genome using BLASR21"
- [results] We use the strain K-12 substr. MG1655 for our alignment-based validation of HECIL: "We use the strain K-12 substr. MG1655 for our alignment-based validation of HECIL"
- [readme] HECIL output FASTA is the evaluation input: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
- [methods] Termination criterion caps iterations: "we also recommend selecting a secondary termination criterion: the maximal number n of allowable iterations for iterative learning."
