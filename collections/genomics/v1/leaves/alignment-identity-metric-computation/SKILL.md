---
name: alignment-identity-metric-computation
description: Use when corrected or uncorrected PacBio long reads (e.g., HECIL iterations
  1–5) must be scored against a reference genome such as E. coli K-12 MG1655 using
  BLASR-derived PI and % matched bases metrics.
when_to_use_negative:
- There is no trusted reference genome for the organism — HECIL's hybrid correction
  itself is short-read-based, but this PI/% matched-bases validation specifically
  requires a reference (e.g., K-12 MG1655) to align against with BLASR
- Evaluation target is a de novo assembly rather than reads — use QUAST and MUMmer's
  dnadiff on Canu assemblies instead
- Evaluation target is k-mer complexity of corrected reads — compute unique k-mer
  counts with Jellyfish instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: BLASR
  role: Aligns uncorrected and corrected long reads to the reference genome to derive
    PI and % matched bases per iteration
- name: HECIL
  role: Produces the corrected long-read FASTA files (iteration 1 through 5) whose
    alignment identity metrics are computed by this skill
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: 'Upstream: aligns Illumina short reads to long reads to mark questionable
    bases before correction, feeding the reads evaluated by this skill'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:08871556570881ddb5469da4b53816fa1996a84f282d6ed8e025586ac30316e9
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# alignment-identity-metric-computation

## Summary

Quantifies long-read correction quality by aligning uncorrected and HECIL-corrected PacBio long reads to a reference genome with BLASR and computing percent identity (PI) and % matched bases per iteration, tracking gains across iterative-learning rounds.

## When to use

Use when corrected or uncorrected PacBio long reads (e.g., HECIL iterations 1–5) must be scored against a reference genome such as E. coli K-12 MG1655 using BLASR-derived PI and % matched bases metrics.

## When NOT to use

- There is no trusted reference genome for the organism — HECIL's hybrid correction itself is short-read-based, but this PI/% matched-bases validation specifically requires a reference (e.g., K-12 MG1655) to align against with BLASR
- Evaluation target is a de novo assembly rather than reads — use QUAST and MUMmer's dnadiff on Canu assemblies instead
- Evaluation target is k-mer complexity of corrected reads — compute unique k-mer counts with Jellyfish instead

## Inputs

- FASTA of uncorrected filtered long reads (e.g., 33,360 E. coli Sequel reads, reads < 100 bp excluded)
- Per-iteration corrected long-read FASTA (LongRead_Corrected.fasta from each HECIL run)
- Reference genome FASTA (E. coli K-12 substr. MG1655)
- BLASR alignment tool

## Outputs

- Per-iteration percent identity (PI) values
- Per-iteration % matched bases values
- Six-row progression table (uncorrected through iteration 5), e.g., pi_matched_bases_progression.csv

## How to apply

Take the uncorrected long-read FASTA (e.g., the 33,360 filtered E. coli Sequel reads left after excluding reads shorter than 100 bp) plus each iteration's corrected output (LongRead_Corrected.fasta per HECIL run) and align each set to the E. coli K-12 substr. MG1655 reference genome using BLASR. From each alignment compute percent identity (PI) and % matched bases, producing one metric pair per correction stage (uncorrected baseline through iteration 5). Judge success by writing the six-row progression table (pi_matched_bases_progression.csv) and confirming PI and % matched bases increase monotonically relative to the uncorrected baseline; for Sequel E. coli the expected trajectory is PI 85.0 → 95.6 (iteration 1) → 97.2 (iteration 5) and % matched bases 85.1 → 95.6 → 97.1. The same alignment-based validation is what the paper uses to demonstrate that HECIL in conjunction with iterative learning consistently outperforms on these metrics.

## Related tools

- **BLASR** (Aligns uncorrected and corrected long reads to the reference genome to derive PI and % matched bases per iteration)
- **HECIL** (Produces the corrected long-read FASTA files (iteration 1 through 5) whose alignment identity metrics are computed by this skill) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Upstream: aligns Illumina short reads to long reads to mark questionable bases before correction, feeding the reads evaluated by this skill)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out  # then align LongRead_Corrected.fasta per iteration to the K-12 MG1655 reference with BLASR to compute PI and % matched bases
```

## Evaluation signals

- PI and % matched bases increase monotonically relative to the uncorrected baseline across iterations 1–5
- For Sequel-sequenced E. coli, expected values: PI 85.0 → 95.6 (iter 1, +12.4%) → 97.2 (iter 5, +14.3%); % matched bases 85.1 → 95.6 (+12.3%) → 97.1 (+14.1%)
- Progression table contains exactly six rows (uncorrected plus iterations 1–5), matching the maximal-number-of-iterations termination criterion
- Alignment is run on the same filtered read set (33,360 E. coli long reads) for every iteration so metric differences reflect correction, not input changes
- Corrected and uncorrected reads are both present in each HECIL output file (LongRead_Corrected.fasta), so the baseline can be recomputed from the same artifact

## Limitations

- Iterative improvement of the correction policy is a conjecture, not mathematically proven; the paper notes proving iterative improvement remains an open problem
- Saturation behavior of the iterative procedure (whether metrics keep improving or plateau) is unvalidated speculation in the paper
- Metric validity depends on an accurate reference genome; misassemblies or strain differences in the reference will depress PI independent of correction quality

## Evidence

- [results] we align corrected long reads to its reference genome using BLASR21: "we align corrected long reads to its reference genome using BLASR21"
- [results] We use the strain K-12 substr. MG1655 for our alignment-based validation of HECIL: "We use the strain K-12 substr. MG1655 for our alignment-based validation of HECIL"
- [results] We filter long reads of E. coli to exclude reads shorter than 100 bp, creating a final set of 33,360: "We filter long reads of E. coli to exclude reads shorter than 100 bp, creating a final set of 33,360"
- [readme] The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
- [results] We then use QUAST23 to evaluate assembly quality: "We then use QUAST23 to evaluate assembly quality"
