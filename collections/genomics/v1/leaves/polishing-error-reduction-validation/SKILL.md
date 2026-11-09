---
name: polishing-error-reduction-validation
description: Use when a draft long-read assembly and its ntEdit+Sealer-polished counterpart
  both exist as FASTA, a reference genome is available, and you must quantify base-accuracy
  improvement (mismatches/indels per 100 kbp).
when_to_use_negative:
- No reference genome is available for the organism, since the mismatch/indel per
  100 kbp metrics are reference-based
- Only a single (draft or polished) assembly is available, since the skill's core
  judgment is a draft-vs-polished decrease in error metrics
- The goal is polishing/finishing itself (running ntEdit+Sealer) rather than validating
  that error rates decreased afterward
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Reference-based quality assessment of draft vs polished assemblies; computes
    mismatches and indels per 100 kbp in a single comparison report
- name: ntEdit
  role: Polishing step whose effect on base accuracy and small indels this validation
    quantifies
  repo: https://github.com/BirolLab/ntEdit
- name: Sealer
  role: Gap-filling/finishing step whose effect this validation quantifies; part of
    the ABySS repository
  repo: https://github.com/BirolLab/abyss
- name: ntEdit+Sealer protocol
  role: Makefile pipeline that produces the polished assembly (.ntedit_edited.prepd.sealer_scaffold.fa)
    evaluated here
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:b7cc20dcb310d5a6d4b64f16c1472d1ddccc85e6d4a3276131c24a2812a2b8f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/ntEdit
  tools:
  - QUAST
  - Sealer
  - ntEdit
  - ntEdit+Sealer
  - ntEdit+Sealer protocol
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# polishing-error-reduction-validation

## Summary

Validates that ntEdit+Sealer polishing/finishing actually reduced assembly error rates by running reference-based QUAST on the draft and polished (suffixed .ntedit_edited.prepd.sealer_scaffold.fa) assemblies and comparing mismatches and indels per 100 kbp.

## When to use

Use when a draft long-read assembly and its ntEdit+Sealer-polished counterpart both exist as FASTA, a reference genome is available, and you must quantify base-accuracy improvement (mismatches/indels per 100 kbp).

## When NOT to use

- No reference genome is available for the organism, since the mismatch/indel per 100 kbp metrics are reference-based
- Only a single (draft or polished) assembly is available, since the skill's core judgment is a draft-vs-polished decrease in error metrics
- The goal is polishing/finishing itself (running ntEdit+Sealer) rather than validating that error rates decreased afterward

## Inputs

- Draft long-read assembly FASTA (e.g., Shasta draft assembly)
- ntEdit+Sealer polished/finished assembly FASTA (suffixed .ntedit_edited.prepd.sealer_scaffold.fa)
- Reference genome FASTA (e.g., E. coli K-12 MG1655, NCBI GCF_000005845.2)

## Outputs

- QUAST report.tsv comparison table with per-assembly metrics, including '# mismatches per 100 kbp' and '# indels per 100 kbp'

## How to apply

Collect the draft assembly FASTA (e.g., the Shasta assembly) and the ntEdit+Sealer-polished assembly (.ntedit_edited.prepd.sealer_scaffold.fa) into one working directory alongside a reference genome FASTA (e.g., E. coli K-12 MG1655 GCF_000005845.2). Run QUAST once on both assemblies together with the --fast option and the -r reference flag so both are scored in a single comparison table. Parse report.tsv and confirm the '# mismatches per 100 kbp' and '# indels per 100 kbp' columns are populated for both assemblies. The rationale is that polishing should correct base errors and small indels (ntEdit) and gap-fill (Sealer), so both error metrics are expected to decrease after ntEdit+Sealer relative to the draft, consistent with the paper's claim of highly accurate, error-corrected assemblies.

## Related tools

- **QUAST** (Reference-based quality assessment of draft vs polished assemblies; computes mismatches and indels per 100 kbp in a single comparison report)
- **ntEdit** (Polishing step whose effect on base accuracy and small indels this validation quantifies) — https://github.com/BirolLab/ntEdit
- **Sealer** (Gap-filling/finishing step whose effect this validation quantifies; part of the ABySS repository) — https://github.com/BirolLab/abyss
- **ntEdit+Sealer protocol** (Makefile pipeline that produces the polished assembly (.ntedit_edited.prepd.sealer_scaffold.fa) evaluated here) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
quast.py --fast -r GCF_000005845.2_genomic.fna shasta_draft.fa polished.ntedit_edited.prepd.sealer_scaffold.fa
```

## Evaluation signals

- report.tsv exists and contains '# mismatches per 100 kbp' and '# indels per 100 kbp' columns for both assemblies
- Both assemblies appear together in a single QUAST comparison table (one invocation, not separate runs)
- Mismatch and indel per 100 kbp values for the polished assembly are lower than for the draft assembly
- The polished input is the ntEdit+Sealer output file (.ntedit_edited.prepd.sealer_scaffold.fa), not an intermediate
- Reference-based metrics were computed (QUAST was given -r with GCF_000005845.2), not de novo-only statistics

## Limitations

- Validation depends on having a suitable, closely matched reference genome (E. coli K-12 MG1655 GCF_000005845.2 in the paper); reference choice affects mismatch/indel counts
- The --fast option trades detailed statistics for speed, limiting the depth of the QC report
- Improved per-100-kbp metrics demonstrate error reduction relative to the reference, not absolute completeness of the assembly

## Evidence

- [abstract] ntEdit+Sealer produces highly accurate, error-corrected genome assemblies: "ntEdit+Sealer produces highly accurate, error-corrected genome assemblies"
- [other] Parse the generated report.tsv and confirm the '# mismatches per 100 kbp' and '# indels per 100 kbp' columns are present for both assemblies.: [evidence span withheld — anchor status: not_found]
- [other] Run QUAST on both assemblies together in a single invocation with the --fast option and the reference genome specified, so all assemblies are reported in one comparison table.: [evidence span withheld — anchor status: not_found]
- [readme] ntEdit polishes the draft assembly and flags erroneous regions, then Sealer fills assembly gaps and erroneous sequence regions flagged by ntEdit.: [evidence span withheld — anchor status: not_found]
