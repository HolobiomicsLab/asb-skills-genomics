---
name: quast-based-assembly-comparison
description: Use when you have a draft long-read assembly and one or more polished/finished
  assemblies plus a reference genome (e.g., E. coli K-12 MG1655 GCF_000005845.2 FASTA)
  and need to quantify base accuracy improvements after polishing.
when_to_use_negative:
- No reference genome is available — reference-based accuracy metrics (mismatches/indels
  per 100 kbp) cannot be computed
- You only have a single assembly and no polished counterpart, so a draft-vs-polished
  error-reduction comparison is not possible
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Reference-based quality assessment of draft and polished assemblies, reporting
    mismatches and indels per 100 kbp in report.tsv
  repo: https://github.com/ablab/quast
- name: ntEdit+Sealer
  role: Protocol that produced the polished/finished assembly being evaluated against
    the draft
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:e982d8d83f74ecf10e496608524fbb82ac2f1e6de7c743a0a3d254af57b34c5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - QUAST
  - ntEdit
  - ntEdit+Sealer
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# quast-based-assembly-comparison

## Summary

Assess draft versus polished genome assemblies with QUAST by running a single reference-based invocation on all assemblies together and comparing per-100-kbp error metrics (mismatches and indels) from report.tsv.

## When to use

Use when you have a draft long-read assembly and one or more polished/finished assemblies plus a reference genome (e.g., E. coli K-12 MG1655 GCF_000005845.2 FASTA) and need to quantify base accuracy improvements after polishing.

## When NOT to use

- No reference genome is available — reference-based accuracy metrics (mismatches/indels per 100 kbp) cannot be computed
- You only have a single assembly and no polished counterpart, so a draft-vs-polished error-reduction comparison is not possible

## Inputs

- Draft long-read genome assembly FASTA (e.g., Shasta assembly of E. coli NDM5)
- Polished/finished assembly FASTA (.ntedit_edited.prepd.sealer_scaffold.fa)
- Reference genome FASTA (E. coli K-12 MG1655, GCF_000005845.2)

## Outputs

- QUAST report.tsv summary table with all assemblies compared in one table
- Per-assembly '# mismatches per 100 kbp' and '# indels per 100 kbp' values for draft vs. polished comparison

## How to apply

Collect the draft Shasta assembly FASTA and the ntEdit+Sealer polished assembly (suffixed .ntedit_edited.prepd.sealer_scaffold.fa) into a working directory alongside the reference genome FASTA. Run QUAST once on all assemblies together with the --fast option and the reference specified so a single comparison table is produced. Parse the generated report.tsv and confirm the '# mismatches per 100 kbp' and '# indels per 100 kbp' columns are populated for both assemblies. Judge correctness by verifying both error metrics decrease in the polished assembly relative to the draft, consistent with ntEdit+Sealer's claim of producing highly accurate, error-corrected assemblies.

## Related tools

- **QUAST** (Reference-based quality assessment of draft and polished assemblies, reporting mismatches and indels per 100 kbp in report.tsv) — https://github.com/ablab/quast
- **ntEdit+Sealer** (Protocol that produced the polished/finished assembly being evaluated against the draft) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
quast.py --fast -r GCF_000005845.2.fasta draft_shasta.fa assembly.ntedit_edited.prepd.sealer_scaffold.fa
```

## Evaluation signals

- report.tsv contains '# mismatches per 100 kbp' and '# indels per 100 kbp' columns for both assemblies
- Both error metrics are lower for the ntEdit+Sealer polished assembly than for the draft Shasta assembly
- All assemblies appear in a single QUAST comparison table from one invocation
- Assembly and reference FASTA files were valid inputs (QUAST completes without reference-related errors)

## Limitations

- Accuracy metrics require a high-quality reference genome; without one QUAST cannot report mismatches/indels per 100 kbp
- The --fast option trades runtime for reduced detail, so some advanced QUAST statistics may be omitted
- Decreases in mismatches/indels are expected outcomes, not guaranteed; a failed polishing run could increase errors

## Evidence

- [methods] The QUAST accuracy metrics (mismatches and indels per 100 kbp) are expected to decrease after polishing and finishing a draft genome with ntEdit+Sealer.: [evidence span withheld — anchor status: not_found]
- [methods] Run QUAST on both assemblies together in a single invocation with the --fast option and the reference genome specified, so all assemblies are reported in one comparison table.: [evidence span withheld — anchor status: not_found]
- [abstract] ntEdit+Sealer produces highly accurate, error-corrected genome assemblies: "ntEdit+Sealer produces highly accurate, error-corrected genome assemblies"
- [readme] An automated protocol for finishing long-read genome assemblies using short reads.: "An automated protocol for finishing long-read genome assemblies using short reads."
