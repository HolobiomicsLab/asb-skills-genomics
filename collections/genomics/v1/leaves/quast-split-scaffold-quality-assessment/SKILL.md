---
name: quast-split-scaffold-quality-assessment
description: Use when a scaffolded assembly contains N gaps introduced by long-read
  gap-filling (e.g. ntLink gap_fill output) and you need to quantify how much real
  sequence replaced the ambiguous Ns.
when_to_use_negative:
- The assembly contains no N gaps (fully gap-closed contigs) — --split-scaffold statistics
  add no information about gap-filling quality
- You want to assess raw scaffolding joins before gap-filling rather than the quality
  of filled gap sequence
- The input is read data or a k-mer/NT-cardinality file rather than an assembled scaffold
  FASTA
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0193
tools:
- name: QUAST
  role: 'Run with --split-scaffold on the gap-filled scaffolds to obtain # N''s per
    100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics'
  repo: https://github.com/ablab/quast
- name: abyss-fac (ABySS)
  role: Compute assembly contiguity and N-content statistics (ABySS v2.3.0+) on the
    gap-filled scaffolds alongside QUAST
  repo: https://github.com/BirolLab/abyss
- name: ntLink
  role: Produces the gap-filled scaffolds being assessed, via `ntLink scaffold gap_fill`
    with overlap=True
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:f2952eeb6430b00d66ff7dc77a95ac4dd242d02ca9451096d647a6fef91f3714
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - QUAST
  - abyss-fac (ABySS)
  - ntLink
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0193
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# quast-split-scaffold-quality-assessment

## Summary

Assess the quality of gap-filling after ntLink scaffolding by running QUAST with the --split-scaffold option, which reports # N's per 100 kbp and separates Contig-level vs Scaffold-level NG50/NGA50 metrics so that effective gap-filling is visible as Contig statistics approaching Scaffold statistics.

## When to use

Use when a scaffolded assembly contains N gaps introduced by long-read gap-filling (e.g. ntLink gap_fill output) and you need to quantify how much real sequence replaced the ambiguous Ns.

## When NOT to use

- The assembly contains no N gaps (fully gap-closed contigs) — --split-scaffold statistics add no information about gap-filling quality
- You want to assess raw scaffolding joins before gap-filling rather than the quality of filled gap sequence
- The input is read data or a k-mer/NT-cardinality file rather than an assembled scaffold FASTA

## Inputs

- Gap-filled scaffold FASTA produced by ntLink gap-filling (e.g. from `ntLink scaffold gap_fill` with overlap=True)
- C. elegans ONT reads (SRR10028109.fq.gz) as used in the reference protocol run
- Optional pre-gap-fill scaffold FASTA for before/after comparison

## Outputs

- QUAST report with --split-scaffold Contig-level vs Scaffold-level NG50/NGA50 statistics and # N's per 100 kbp
- abyss-fac summary of assembly contiguity and N-content statistics for the gap-filled scaffolds

## How to apply

After running ntLink with gap-filling (e.g. `ntLink scaffold gap_fill target=celegans_flye.fa reads=SRR10028109.fq.gz k=32 w=100 t=5` with overlap=True), first compute overall contiguity and N-content statistics on the gap-filled scaffolds using abyss-fac from ABySS (v2.3.0+). Then run QUAST with the --split-scaffold option on the gap-filled FASTA: QUAST splits scaffolds at N runs and reports # N's per 100 kbp plus separate Contig-level and Scaffold-level NG50/NGA50 statistics. Rationale: with efficient gap-filling, the Contig NG50/NGA50 statistics approach the Scaffold statistics and the number of Ns per 100 kbp decreases, confirming that gap-fill inserted real read sequence rather than leaving ambiguous bases between joined contigs. Validate by comparing the abyss-fac and QUAST --split-scaffold values against the gap-filled assembly values reported in Table 4 of the ntLink protocol.

## Related tools

- **QUAST** (Run with --split-scaffold on the gap-filled scaffolds to obtain # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics) — https://github.com/ablab/quast
- **abyss-fac (ABySS)** (Compute assembly contiguity and N-content statistics (ABySS v2.3.0+) on the gap-filled scaffolds alongside QUAST) — https://github.com/BirolLab/abyss
- **ntLink** (Produces the gap-filled scaffolds being assessed, via `ntLink scaffold gap_fill` with overlap=True) — https://github.com/bcgsc/ntLink

## Examples

```
quast.py --split-scaffolds celegans_flye_gapfilled.fa -o quast_gapfill_out; abyss-fac celegans_flye_gapfilled.fa
```

## Evaluation signals

- The gap-filled output FASTA exists and parses as valid FASTA
- QUAST --split-scaffold report shows Contig NG50/NGA50 statistics approaching the Scaffold NG50/NGA50 statistics
- # N's per 100 kbp decreases relative to the pre-gap-fill assembly while NG50 is maintained
- abyss-fac contiguity/N-content metrics reproduce the trend of the gap-filled assembly values reported in Table 4 of the ntLink protocol

## Limitations

- Gap-fill quality conclusions depend on the reference protocol's C. elegans ONT benchmark; other genomes/reads may shift absolute NG50/NGA50 and N-content values
- abyss-fac requires ABySS v2.3.0 or higher
- The gap_fill feature of ntLink requires overlap=True; omitting it changes the scaffold output being assessed

## Evidence

- [abstract] Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
- [methods] Run QUAST with the --split-scaffold option on the gap-filled scaffolds to obtain # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics.: [evidence span withheld — anchor status: not_found]
- [results] Gap quality after ntLink gap-filling can be assessed with QUAST --split-scaffold statistics: with efficient gap-filling, Contig NG50/NGA50 statistics approach the Scaffold statistics and the number of Ns per 100 kbp decreases.: [evidence span withheld — anchor status: not_found]
