---
name: reference-based-assembly-alignment-metrics
description: Use when you have draft and polished/finished FASTA assemblies and a
  closely related reference genome, and you need to quantify base errors (mismatches,
  indels per 100 kbp) to confirm polishing improved accuracy.
when_to_use_negative:
- No reference genome is available or the reference is too divergent from the sample,
  making reference-based accuracy metrics unreliable
- You only need contiguity metrics (N50, total length) and have no interest in base-level
  error rates; --fast QUAST without a reference would suffice
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: QUAST
  role: Reference-based quality assessment of draft vs polished assemblies; reports
    mismatches and indels per 100 kbp in report.tsv
  repo: https://github.com/ablab/quast
- name: ntEdit
  role: Produces the polished assembly (corrects base errors and small indels) whose
    accuracy is evaluated against the draft
  repo: https://github.com/BirolLab/ntEdit
- name: ntEdit+Sealer
  role: Makefile finishing pipeline producing the gap-filled, polished assembly compared
    against the draft with QUAST
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:da67eb459817971ac9938e6b9a161d9643bb2ff643777b7f0a1ef3d751f90331
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
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# reference-based-assembly-alignment-metrics

## Summary

Assess draft versus polished genome assembly quality by running QUAST against a reference genome and comparing accuracy metrics (mismatches and indels per 100 kbp) before and after ntEdit+Sealer polishing/finishing.

## When to use

Use when you have draft and polished/finished FASTA assemblies and a closely related reference genome, and you need to quantify base errors (mismatches, indels per 100 kbp) to confirm polishing improved accuracy.

## When NOT to use

- No reference genome is available or the reference is too divergent from the sample, making reference-based accuracy metrics unreliable
- You only need contiguity metrics (N50, total length) and have no interest in base-level error rates; --fast QUAST without a reference would suffice

## Inputs

- Draft long-read assembly FASTA (e.g. Shasta draft assembly)
- Polished/finished assembly FASTA (ntEdit+Sealer output, e.g. .ntedit_edited.prepd.sealer_scaffold.fa)
- Reference genome FASTA (e.g. E. coli K-12 MG1655, GCF_000005845.2)

## Outputs

- QUAST report.tsv comparison table with per-assembly quality metrics
- Confirmation that '# mismatches per 100 kbp' and '# indels per 100 kbp' columns are present for both assemblies

## How to apply

Collect the draft Shasta assembly FASTA and the ntEdit+Sealer polished assembly (e.g. suffixed .ntedit_edited.prepd.sealer_scaffold.fa) into one working directory, together with the reference genome FASTA (e.g. E. coli K-12 MG1655, GCF_000005845.2). Run QUAST on all assemblies in a single invocation with the --fast option and the reference genome specified, so all assemblies are reported in one comparison table. Parse the generated report.tsv and confirm the '# mismatches per 100 kbp' and '# indels per 100 kbp' columns are populated for each assembly. Compare draft versus polished values of these two error metrics: both should decrease after ntEdit+Sealer polishing, consistent with the paper's claim of highly accurate, error-corrected assemblies.

## Related tools

- **QUAST** (Reference-based quality assessment of draft vs polished assemblies; reports mismatches and indels per 100 kbp in report.tsv) — https://github.com/ablab/quast
- **ntEdit** (Produces the polished assembly (corrects base errors and small indels) whose accuracy is evaluated against the draft) — https://github.com/BirolLab/ntEdit
- **ntEdit+Sealer** (Makefile finishing pipeline producing the gap-filled, polished assembly compared against the draft with QUAST) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
quast.py --fast -r GCF_000005845.2.fa draft.fa polished.ntedit_edited.prepd.sealer_scaffold.fa
```

## Evaluation signals

- report.tsv is generated and contains '# mismatches per 100 kbp' and '# indels per 100 kbp' rows for both assemblies
- Mismatch count per 100 kbp is lower for the polished assembly than the draft
- Indel count per 100 kbp is lower for the polished assembly than the draft
- Both assemblies were evaluated in one QUAST run against the same reference, so values appear in a single comparison table

## Limitations

- Metrics depend on the choice of reference genome; a distant reference inflates apparent errors
- --fast mode trades runtime for reduced analysis depth, so some QUAST metrics may be skipped
- Alignment-based quality assessment is memory-intensive compared to the alignment-free ntEdit+Sealer protocol itself

## Evidence

- [abstract] QUAST accuracy metrics: "ntEdit+Sealer produces highly accurate, error-corrected genome assemblies"
- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [intro] intro: "While read alignments are information-rich, they are also highly memory intensive and therefore not practical for organisms with large genomes."
- [readme] readme: "An automated protocol for finishing long-read genome assemblies using short reads."
