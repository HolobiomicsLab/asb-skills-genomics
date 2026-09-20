---
name: abyss-assembly-fasta-statistics
description: Use when you need contiguity and N-content statistics on a draft assembly
  or gap-filled scaffolds (e.g., after ntLink gap_fill) to judge whether scaffold
  gaps were filled with real sequence rather than ambiguous Ns.
when_to_use_negative:
- The assembly contains no N-runs/scaffold gaps (e.g., a pure contig assembly), where
  --split-scaffold contig-vs-scaffold comparisons are uninformative
- You need read-level or variant-level QC rather than assembly-level contiguity and
  N-content statistics
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: abyss-fac (ABySS)
  role: Computes assembly contiguity and N-content statistics on the gap-filled scaffolds
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: 'Run with --split-scaffold to obtain # N''s per 100 kbp and separate Contig-level
    vs Scaffold-level NG50/NGA50 metrics'
  repo: https://github.com/ablab/quast
- name: ntLink
  role: Produces the gap-filled scaffolds whose quality these statistics assess (run
    with gap_fill and overlap=True)
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:fee6820daef2d9f758b302c05d99c06f48aba5df2d59dfa0928892f6bd35cacd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/abyss
  tools:
  - QUAST
  - abyss-fac (ABySS)
  - ntLink
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# abyss-assembly-fasta-statistics

## Summary

Compute assembly contiguity and N-content statistics (NG50/NGA50, # N's per 100 kbp) on scaffolds using abyss-fac from ABySS v2.3.0+ and QUAST with --split-scaffold, to quantify gap-filling quality after ntLink scaffolding.

## When to use

Use when you need contiguity and N-content statistics on a draft assembly or gap-filled scaffolds (e.g., after ntLink gap_fill) to judge whether scaffold gaps were filled with real sequence rather than ambiguous Ns.

## When NOT to use

- The assembly contains no N-runs/scaffold gaps (e.g., a pure contig assembly), where --split-scaffold contig-vs-scaffold comparisons are uninformative
- You need read-level or variant-level QC rather than assembly-level contiguity and N-content statistics

## Inputs

- Gap-filled scaffold assembly in FASTA format (output of ntLink scaffold gap_fill)
- Optionally, the pre-gap-filling assembly FASTA for before/after comparison

## Outputs

- abyss-fac assembly statistics (contiguity and N-content metrics)
- QUAST --split-scaffold report with Contig-level vs Scaffold-level NG50/NGA50 and # N's per 100 kbp

## How to apply

After running ntLink gap-filling (e.g., `ntLink scaffold gap_fill target=celegans_flye.fa reads=SRR10028109.fq.gz k=32 w=100 t=5` with overlap=True), run abyss-fac from ABySS v2.3.0+ on the gap-filled FASTA to obtain basic contiguity and N-content statistics. Then run QUAST with the --split-scaffold option on the gap-filled scaffolds; this splits scaffolds at N-runs so it reports separate Contig-level vs Scaffold-level NG50/NGA50 metrics and # N's per 100 kbp. The rationale is that efficient gap-filling replaces N-runs with actual read sequence, so Contig-level statistics approach Scaffold-level statistics and the Ns per 100 kbp decreases; compare against the pre-gap-filling assembly to confirm this trend.

## Related tools

- **abyss-fac (ABySS)** (Computes assembly contiguity and N-content statistics on the gap-filled scaffolds) — https://github.com/BirolLab/abyss
- **QUAST** (Run with --split-scaffold to obtain # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics) — https://github.com/ablab/quast
- **ntLink** (Produces the gap-filled scaffolds whose quality these statistics assess (run with gap_fill and overlap=True)) — https://github.com/bcgsc/ntLink

## Examples

```
abyss-fac celegans_gapfilled.fa; quast.py --split-scaffold -o quast_out celegans_gapfilled.fa
```

## Evaluation signals

- The gap-filled FASTA exists and parses as valid FASTA
- abyss-fac and QUAST --split-scaffold statistics reproduce the trend reported in Table 4: maintained NG50 with reduced # N's per 100 kbp
- Contig-level NG50/NGA50 approach the Scaffold-level NG50/NGA50 after gap-filling
- The number of Ns per 100 kbp decreases relative to the pre-gap-filling assembly

## Limitations

- Trend-based validation: acceptance is judged by reproducing reported metric trends (maintained NG50, reduced Ns per 100 kbp) rather than exact values, so differences in input data or parameters may shift absolute statistics
- Assessment is limited to assembly-level contiguity and N-content; base-level accuracy of filled sequence requires other evidence

## Evidence

- [results] Gap quality after ntLink gap-filling can be assessed with QUAST --split-scaffold statistics: [evidence span withheld — anchor status: not_found]
- [methods] abyss-fac on gap-filled scaffolds: [evidence span withheld — anchor status: not_found]
- [methods] overlap=True required for gap_fill: [evidence span withheld — anchor status: not_found]
- [readme] ABySS repo citation: [evidence span withheld — anchor status: not_found]
- [abstract] Recent ntLink features include gap-filling: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
