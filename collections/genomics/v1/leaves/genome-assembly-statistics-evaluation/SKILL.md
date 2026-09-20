---
name: genome-assembly-statistics-evaluation
description: Use when a scaffolded or gap-filled assembly must be validated against
  a baseline assembly — e.g. to check that gap-filling reduced Ns per 100 kbp while
  maintaining NG50 — and contig-level vs scaffold-level statistics must be separated.
when_to_use_negative:
- The input is raw reads or a read-to-assembly alignment rather than an assembled
  FASTA — statistics evaluation requires assembled contig/scaffold sequences.
- The goal is scaffolding or gap-filling itself rather than evaluating its output
  — use ntLink directly instead of this evaluation skill.
- Only scaffold-level statistics are needed and contig-vs-scaffold separation is irrelevant
  — plain abyss-fac alone may suffice without QUAST --split-scaffold.
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: abyss-fac (ABySS)
  role: Computes assembly contiguity and N-content statistics on the gap-filled scaffolds
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: 'Run with --split-scaffold to obtain # N''s per 100 kbp and separate Contig-level
    vs Scaffold-level NG50/NGA50 metrics'
  repo: https://github.com/ablab/quast
- name: ntLink
  role: Produces the gap-filled scaffolds that are evaluated by this skill
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:6d27f5a0c82241b35c197452ab61e6efa24a8dfba7cac7559e058ac50c0340f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/abyss
  tools:
  - QUAST
  - Racon
  - abyss-fac (ABySS)
  - minimap2
  - ntLink
  - sra-tools
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# genome-assembly-statistics-evaluation

## Summary

Evaluates genome assembly contiguity and correctness after scaffolding/gap-filling by computing N-content and NG50/NGA50 statistics with abyss-fac and QUAST --split-scaffold, as done in the ntLink Basic Protocol 2 (C. elegans ONT data).

## When to use

Use when a scaffolded or gap-filled assembly must be validated against a baseline assembly — e.g. to check that gap-filling reduced Ns per 100 kbp while maintaining NG50 — and contig-level vs scaffold-level statistics must be separated.

## When NOT to use

- The input is raw reads or a read-to-assembly alignment rather than an assembled FASTA — statistics evaluation requires assembled contig/scaffold sequences.
- The goal is scaffolding or gap-filling itself rather than evaluating its output — use ntLink directly instead of this evaluation skill.
- Only scaffold-level statistics are needed and contig-vs-scaffold separation is irrelevant — plain abyss-fac alone may suffice without QUAST --split-scaffold.

## Inputs

- Gap-filled or scaffolded genome assembly FASTA (e.g. celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
- Baseline/draft assembly FASTA for comparison
- Optionally, polished assembly FASTA (e.g. celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa)

## Outputs

- abyss-fac statistics table (contiguity and N-content metrics for the assembly)
- QUAST report with # N's per 100 kbp and split Contig-level vs Scaffold-level NG50/NGA50 statistics
- Comparison against reported Table 4 values (accept/reject judgment)

## How to apply

After scaffolding/gap-filling (e.g. ntLink scaffold gap_fill with k=32 w=100 t=5 and overlap=True), compute N-content and contiguity statistics on the output FASTA using abyss-fac from ABySS v2.3.0+. Then run QUAST with the --split-scaffold option on the gap-filled scaffolds to obtain # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics. Compare the computed statistics against reported values (Table 4 of the ntLink paper); accept the result if the gap-filled FASTA parses as valid FASTA and the metrics reproduce the trend of maintained NG50 with reduced # N's per 100 kbp. The rationale is that with efficient gap-filling, Contig NG50/NGA50 statistics approach the Scaffold statistics because ambiguous N runs are replaced by actual raw ONT read sequence.

## Related tools

- **abyss-fac (ABySS)** (Computes assembly contiguity and N-content statistics on the gap-filled scaffolds) — https://github.com/BirolLab/abyss
- **QUAST** (Run with --split-scaffold to obtain # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics) — https://github.com/ablab/quast
- **ntLink** (Produces the gap-filled scaffolds that are evaluated by this skill) — https://github.com/bcgsc/ntLink

## Examples

```
quast.py --split-scaffold celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa ; abyss-fac celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa
```

## Evaluation signals

- The gap-filled FASTA exists and parses as valid FASTA.
- abyss-fac statistics on the gap-filled scaffolds match the values reported in Table 4 of the ntLink paper.
- QUAST --split-scaffold shows Contig NG50/NGA50 approaching the Scaffold NG50/NGA50 after gap-filling.
- QUAST --split-scaffold reports a decreased # N's per 100 kbp relative to the pre-gap-fill assembly.
- For the Racon-polished variant, QUAST statistics on celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa are consistent with Table 4.

## Limitations

- Statistics alone report contiguity/N-content trends; correctness assessment depends on comparison to reference-based metrics (NGA50) and reported values.
- The reported trend (maintained NG50, reduced Ns per 100 kbp) was established on C. elegans Flye ONT data (SRR10028109); other datasets may not reproduce exact Table 4 values.
- ABySS v2.3.0+ is required for abyss-fac; older versions may not be compatible.

## Evidence

- [results] Gap quality after ntLink gap-filling can be assessed with QUAST --split-scaffold statistics: with efficient gap-filling, Contig NG50/NGA50 statistics approach the Scaffold statistics and the number of Ns per 100 kbp decreases.: [evidence span withheld — anchor status: not_found]
- [methods] Compute assembly contiguity and N-content statistics on the gap-filled scaffolds using abyss-fac from ABySS (v2.3.0+).: [evidence span withheld — anchor status: not_found]
- [methods] accept the output if the gap-filled FASTA exists, parses as valid FASTA, and the contiguity/N-content metrics reproduce the reported trend of maintained NG50 with reduced # N's per 100 kbp: [evidence span withheld — anchor status: not_found]
- [methods] Run QUAST with the --split-scaffold option on the gap-filled scaffolds to obtain # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics.: [evidence span withheld — anchor status: not_found]
